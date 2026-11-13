# HyunDPCR — 기술 아키텍처 설계 v2.0

**Updated:** 2026-11-13 | **Decision:** Python + Streamlit-first research MVP | **Status:** 설계 문서(구현·배포 전)

> 이 문서의 모든 컴포넌트·CLI·폴더 경로는 **계획**이며 현재 실행 가능한 코드나 임상 검증된 시스템을 뜻하지 않는다. 합성 데이터 연구 데모에서 실제 환자 데이터를 처리하거나 임상 판단·검사 지연을 권고하지 않는다.

## v3.0 설계 확장 문서 (2026-11-13)

본 문서는 기술 아키텍처의 상위 설계(v2.0)이다. **개발 실행 명세(v3.0)**는 아래에서 관리하며, 실제 소스코드/앱은 아직 구축되지 않았다.

- [IMPLEMENTATION_BLUEPRINT.md](IMPLEMENTATION_BLUEPRINT.md): Python 모듈 책임·Pydantic I/O 계약·Streamlit 실행 흐름·수학 엔진/상태전이·Go/No-Go
- [UI_SPEC.md](UI_SPEC.md): 6개 화면 와이어프레임, 기능·경고·접근성
- [DATA_MODEL_AND_TESTS.md](DATA_MODEL_AND_TESTS.md): ER 다이어그램, 시점 불변식, 테스트/CI 매트릭스
- [SECURITY_RELEASE_GATES.md](SECURITY_RELEASE_GATES.md): PHI 및 캐시 경계, 위협/규제/임상 전환 게이트
- 개발 과제: [M0 #1](https://github.com/HyunStudio/HyunDPCR/issues/1) · [M1 #2](https://github.com/HyunStudio/HyunDPCR/issues/2) · [M2 #3](https://github.com/HyunStudio/HyunDPCR/issues/3)

## 1. 핵심 결정 (ADR-001)

**Phase 1:** Python 3.12+ 분석 라이브러리 + Streamlit 다중 페이지 UI + 로컬 합성 데이터. **FastAPI, React, PostgreSQL, 별도 인증 서버, 클라우드 PHI 저장소는 초기 필수 구성에서 제외**한다.

**Phase 2:** 실데이터 연구·기관별 격리, 재현 가능한 잡 실행, 데이터 접근과 배포 요건이 확인되면 DB/작업큐/권한 계층을 단계적으로 도입한다.

**Phase 3:** 인증된 고객용 API·다기관 통합 요건이 생길 때 FastAPI, 필요시 React/Next.js. 단, 임상 CDS 기능은 **외부 검증·안전·관할 규제 검토 이후** 별도 게이트로 관리한다.

**설계 원칙:** UI는 얇게, 분석은 Python 패키지의 순수 함수/명시적 서비스로, 데이터 접근과 결과 근거는 분리한다. Streamlit 화면을 FastAPI의 클라이언트로 처음부터 만들지 않는다.

## 2. 전체 시스템 설계도

### Phase 1 — Streamlit 단일 프로세스 / 합성 연구용

~~~mermaid
flowchart TB
  U["Research user (synthetic demo)"] --> ST["Streamlit UI: Overview / Timeline / Risk / Pathway / Guard"]
  ST --> CTRL["UI adapter + Pydantic input validation"]
  CTRL --> SERVICE["Python application services (no HTTP required)"]
  SERVICE --> SIG["Signal: time-updated diagnosis risk"]
  SERVICE --> PTH["Pathway: stochastic event simulation"]
  SERVICE --> OPT["Options: VOI scenario (guarded, optional)"]
  SERVICE --> GD["Guard: metadata / safety / audit"]
  SIG --> CORE["Quant core: NumPy / SciPy / scikit-learn"]
  PTH --> CORE
  OPT --> CORE
  SIG --> VAL["Provenance gate: available_at <= as_of"]
  PTH --> VAL
  OPT --> SAFE["Guideline / permitted-action constraints (simulation)"]
  VAL --> DATA["Synthetic fixtures + versioned scenario manifests"]
  SAFE --> DATA
  GD --> REPORT["Research report: CSV / JSON / HTML"]
  SERVICE --> REPORT
  ST --> REPORT
  style DATA fill:#e5f3ff,color:#0b355b
~~~

- **Signal:** 고정된 진단 수평선(예: 90/180/365일) 내의 **암 진단 기록** 위험. 암 생물학적 발병/병기/성장 예측 아님.
- **Pathway:** 가상 재진·검사·의뢰 및 검사 역량/대기열의 이벤트 시뮬레이션.
- **Options:** 검사 정확도, 비용, 위해 및 허용된 행동집합이 가정으로 명시된 연구용 가치정보(VOI) 비교. 불충분하면 수치 계산 중단.
- **Guard:** 데이터·모델 버전/seed/인과 해석 경고/누수 검출/재현성/보고서 워터마크.

**위 Mermaid는 GitHub에서 렌더되는 논리 아키텍처 도식**이며, 실제 의료용 서비스 계통도나 동작하는 시스템을 의미하지 않는다.

### Phase 2/3 — 확장 가능한 배포 경계 (구현·보안 심사 전)

~~~mermaid
flowchart LR
  UI["Streamlit research UI"] --> GATE["Institutional SSO / gateway"]
  CLIENT["Future enterprise client / EMR"] --> GATE
  GATE --> API["Optional FastAPI service"]
  API --> JOB["Research job queue / workers"]
  API --> CORE["Reusable Python domain package"]
  JOB --> CORE
  CORE --> STORE["Approved encrypted research data store"]
  CORE --> AUDIT["Access / model / data audit"]
  API --> AUDIT
  GATE --> POLICY["RBAC + tenant boundary + policy"]
  POLICY --> API
~~~

이 도식은 향후 **기관 승인 데이터 처리 환경의 목표 구조**이다. Streamlit Community Cloud 등 공개 데모 호스팅에 실제 환자 자료 업로드를 허용하지 않는다. 인증/접근권한은 별도의 기관 승인을 거쳐 구현한다.

## 3. 기술 스택 결론

| Layer | Phase 1 채택 | Future / 조건부 | 이유 |
|---|---|---|---|
| Runtime | **Python 3.12+** | 지원 버전 변경 시 검증 | 분석 생태계와 재현성 |
| Research UI | **Streamlit**, multi-page | React / Next.js (필요할 때만) | 빠른 대화형 데모, Python 분석 직결 |
| Charts | Streamlit + Altair/Plotly 중 선택 | 고급 시각화 추가 | 시계열 및 불확실성 표시 |
| Input schema | **Pydantic** | FHIR R4 adapter | 데이터 계약·오류 메시지 |
| Data / numerics | **NumPy, pandas, SciPy** | Polars (대량 처리 때) | 합성 시계열/확률·최적화 |
| Prediction baseline | **scikit-learn** | lifelines, scikit-survival, XGBoost/PyTorch | 복잡한 AI보다 baseline 우선 |
| Simulation | **NumPy random.Generator**, SciPy | 이벤트 시뮬레이션 전용 패키지 | 고정 seed·배포 재현성 |
| Storage | **Versioned synthetic files**, 필요 시 SQLite(로컬 결과만) | 기관 승인 PostgreSQL/객체 저장소 | 초기 복잡성 최소화 |
| Backend | **직접 Python 서비스 호출** | **FastAPI** (외부 연동·권한·작업큐 필요 시) | 이중 앱 운영 회피 |
| Quality | **pytest, Ruff**, type check(선택) | CI 품질/보안 자동화 강화 | 테스트 우선 |
| Environments | **pyproject.toml + locked dependencies**, Docker(선택) | 기관 관리 Docker/Kubernetes | 재현성·이식성 |

**버전 관리:** 정확한 종속성 버전은 첫 실행 가능한 앱을 작성할 때 lockfile로 고정하고 CI 테스트해야 한다. 지금은 라이브러리 설치·호환성이나 배포가 완료된 것이 아니다.

## 4. 설계된 저장소 구조 (아직 생성되지 않은 소스)

~~~text
HyunDPCR/
├── README.md
├── pyproject.toml                 # future: package + deps + tooling
├── uv.lock / pinned lockfile      # future: reproducible environment
├── .gitignore                     # future: secrets/PHI/artifacts blocking
├── app/
│   ├── Home.py                    # Streamlit entrypoint
│   ├── pages/
│   │   ├── 01_Patient_Timeline.py
│   │   ├── 02_Risk_Evolution.py
│   │   ├── 03_Pathway_Simulator.py
│   │   ├── 04_Decision_Analysis.py
│   │   └── 05_Validation_Audit.py
│   └── components/                # accessible plots/forms/warnings
├── src/hyundpcr/
│   ├── schemas/                   # Pydantic: events, queries, outputs
│   ├── data/                      # synthetic generator + time filtering
│   ├── models/                    # baseline, landmark, competing risk
│   ├── quant/                     # uncertainty, Monte Carlo, VOI
│   ├── simulation/                # event-time and queue models
│   ├── governance/                # safe actions, provenance, audit, cards
│   ├── services/                  # UI-independent application functions
│   └── reporting/                 # versioned research exports
├── examples/synthetic/            # strictly fake data and scenario configs
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── leakage/
│   ├── safety/
│   └── accessibility/
├── docs/
│   ├── ARCHITECTURE.md
│   └── ...                        # business, quant and validation docs
└── .github/workflows/             # future: test + static checks
~~~

**초기 실행 계획(코드 구현 이후에만):**

~~~bash
python -m pip install -e .
python -m streamlit run app/Home.py
python -m pytest
~~~

첫 구현의 Definition of Done: 위 3개 명령이 문서화된 개발 환경에서 성공하고, 가상 데이터 warning / 누수 방지 / 고정 seed / CSV warning을 자동 검증해야 한다.

## 5. 컴포넌트 의존성 및 실행 계약

~~~mermaid
sequenceDiagram
  actor R as Researcher
  participant UI as Streamlit
  participant V as Schemas and provenance
  participant S as Python services
  participant Q as Signal / Quant / Simulation
  participant G as Guard / Reporting
  R->>UI: Choose synthetic scenario and as-of date
  UI->>V: Validate inputs and time horizon
  V-->>UI: Validated request or error
  UI->>S: evaluate(request)
  S->>V: Filter events available by as-of
  V-->>S: Time-consistent synthetic history
  S->>Q: Compute reproducible research outputs
  Q-->>S: Estimate + assumptions + uncertainty
  S->>G: Register run metadata / safety warnings
  G-->>S: Versioned research result
  S-->>UI: Structured result object
  UI-->>R: Chart + uncertainty + clear synthetic warning
~~~

**API-less 서비스 계약:** Streamlit 페이지는 다음 구조의 입력/출력 타입으로 내부 Python 서비스를 호출한다.

- RiskEvaluationRequest: synthetic_case_id, prediction_as_of, horizon_days, model_id, dataset_version, random_seed.
- SimulationRequest: synthetic_case_id, permitted_scenario_ids, capacity_assumptions, horizon_days, random_seed.
- AnalysisResult: analysis_mode, run_id, model_version, dataset_version, as_of, horizon_days, outcome_definition, population_definition, risk_estimate, uncertainty, assumptions, warnings.
- Clinical-use flags: 현재 **hard-coded disabled**, UI/백엔드 양쪽에서 실제 임상 모드 불허.

**중요:** 사용자가 폼에서 '지금 검사/기다리기'를 선택하는 연구 데모는 임상 의사결정 시스템이 아니다. 임상 지침상 평가 의무가 있거나 증거가 부족하면 비교 기능은 중단/제외한다.

## 6. 데이터 계약: 시간 가용성 우선

~~~json
{
  "event_id": "synthetic-event-001",
  "patient_id": "synthetic-person-001",
  "event_type": "observation",
  "clinical_time": "2026-01-03T12:00:00Z",
  "recorded_at": "2026-01-03T13:00:00Z",
  "available_at": "2026-01-04T09:00:00Z",
  "reported_onset_at": null,
  "code_system": "DEMO",
  "code": "SYMPTOM_X",
  "value": "DEMO_ONLY",
  "source": "synthetic-generator",
  "quality_flags": []
}
~~~

- 예측 입력 이벤트는 항상 **available_at <= prediction_as_of**를 만족해야 한다.
- clinical_time(사건 발생 추정), recorded_at(기록), available_at(분석 가능)을 구분한다. 진단 당시에 **알 수 없던 정보**를 포함하면 미래정보 유출이다.
- 검사 주문·채취·결과 보고·의료진 확인 시간은 구분한다.
- 합성 위험과 실제 검증된 위험을 혼합하지 않는다.
- 암 진단의 event_date와 recorded_date, death/censoring, 결과 확인수준을 따로 보존한다.
- 최초 데이터셋은 **개인식별정보가 존재하지 않는 synthetic-only fixtures**로 제한한다.

## 7. Streamlit 구체 구현 규칙

- **상태:** 연구 케이스/입력/seed는 st.session_state로 세션 단위 보관하며, UI 재실행 시 계산 이벤트가 불필요하게 누적되지 않도록 제어한다.
- **캐싱:** st.cache_data 또는 st.cache_resource의 기본 공유 범위에 환자별 식별정보, 세션별 승인 데이터, 비밀정보를 넣지 않는다. Phase 1은 불변의 공개 합성 fixture나 thread-safe 모델에 대해서만 캐싱 검토한다.
- **정책/의료 규칙:** Streamlit 페이지에 숫자 임계값을 하드코딩하지 않는다. governance의 검증된 규칙만 호출한다.
- **비동기/대용량:** 초기에는 작은 합성 샘플로 동기 처리. 과도한 시간·메모리가 필요한 연구 실행은 향후 작업큐로 분리.
- **UI/보고 모두:** 'SYNTHETIC RESEARCH DEMO — NOT FOR CLINICAL USE'와 예측 시점·기간·결과 정의 표시.
- **접근성:** 키보드 내비게이션, 200% 확대, 고대비, 단순 시각화 외 텍스트 요약과 표 제공, 색상에 의존하지 않는 상태 표시.
- **예외 처리:** 미래 데이터, 임상 위해 입력, 수치 범위 오류, 불충분 정보, 실패한 모델의 문제를 사용자에게 설명하고 조용히 기본값으로 대체하지 않는다.

Streamlit는 재실행·전역 캐싱 모델을 가지므로 의료정보로 넘어갈 때는 멀티유저/세션/테넌트 경계를 별도 보안 설계해야 한다. 현 단계에서는 **실제 환자 기록 사용 금지**.

## 8. 품질보증: 필수 자동화 테스트

| ID | Test | 통과 기준 |
|---|---|---|
| T01 | 미래 정보 누출 | t 이후 available_at 데이터 추가에도 t 예측 불변 |
| T02 | 합성 데이터 및 seed 재현 | 같은 seed+모델/데이터/설정에서 결과 동일 |
| T03 | 수치 유효성 | 모든 확률 0~1, 잘못된 입력은 명시 오류 |
| T04 | 사건 시간 | 역전된 날짜, 누락된 availability, 음수 horizon 거부 |
| T05 | 생존/경쟁위험 | 누적발생률 합 <= 1, 시간 단조성(해당 모형) |
| T06 | VOI 게이트 | 민감도/특이도/비용/위해/허용행동 불명 시 계산 중단 |
| T07 | 지침 제약 | 즉시 평가가 필요한 시나리오의 지연 옵션 미제공 |
| T08 | 위험 결과 표시 | UI/CSV/HTML에 synthetic·endpoint·horizon 명기 |
| T09 | 감사 | 실행마다 데이터/모델/seed/파라미터/상태 기록 |
| T10 | 접근성 | 확대·키보드·대체 표/텍스트 사용성 수동+자동 점검 |
| T11 | 보안 | 비밀키/PHI 커밋 차단, 경로/입력 검증 |
| T12 | 재현 배포 | clean install → pytest → Streamlit smoke test |

정확한 검증 방법/임상 평가지표는 [VALIDATION_SAFETY.md](VALIDATION_SAFETY.md) 및 [QUANT_METHODS.md](QUANT_METHODS.md) 참조.

## 9. 배포와 데이터 안전 경계

**Phase 1 (권장):** 로컬 개발 또는 합성 데이터만 포함하는 격리된 연구 데모. 공개 데모는 어떠한 실제 환자 업로드 기능도 제공하지 않는다.

**Phase 2 (기관용):** 인가된 데이터 접근 환경을 준비한 후 SSO, RBAC, 감사로그, 데이터 암호화, 보관·파기, 승인된 네트워크/저장소를 구현. 기관에서 보안/법률 검토한 경우에만 실제 기록 분석.

**Phase 3 (기업/API):** 독립적인 FastAPI 서비스, 인증 게이트웨이, DB, 백그라운드 작업 처리, 테넌트 분리, 롤백/버전 거버넌스. 표준 FHIR 어댑터는 구체적인 현지 프로파일과 이용권 확인 후 추가.

- Git에 데이터 원본·토큰·비밀 설정·PHI를 올리지 않는다. 후속 구현에서 .gitignore와 secret scanning을 설정한다.
- Streamlit Community Cloud는 **합성 자료 데모 전용**으로만 검토한다. 거기에 실제 의료데이터를 업로드하거나 저장하지 않는다.
- LLM을 핵심 예측기나 원자료 처리자로 처음부터 채택하지 않는다.
- 상업용 시스템에서는 규제/인증과 데이터 계약을 충족하기 전 환자별 위험 반환을 막는다.

## 10. FastAPI 도입 기준 및 마이그레이션 계획

**도입 트리거:** 여러 고객/앱이 공통 예측을 호출해야 함, 기관 보안·권한과 API 감사가 필요함, 장시간 작업/비동기 큐 도입, FHIR 기반 외부 연동이 확정됨.

**전환 방법:**

1. Streamlit이 직접 부르던 **src/hyundpcr/services/** 함수를 유지.
2. FastAPI router는 같은 Pydantic request/result schema로 thin adapter 구성.
3. Python 서비스에 대한 유닛 테스트는 변경 없이 재사용.
4. API 수준 인증/RBAC/Rate limit/감사·에러 표준을 추가.
5. 필요할 때만 React 등 분리 프런트엔드를 도입.

**외부 API 초안(현재 없음):**

~~~text
GET  /v1/demo/cases             # synthetic example only
POST /v1/risk/evaluate          # gated / research
POST /v1/pathway/simulate       # synthetic scenarios
POST /v1/options/compare        # conditional VOI analysis
GET  /v1/audits/{run_id}        # authorized audit
GET  /v1/reports/{run_id}       # watermarked report
~~~

## 11. 참고 및 결정 사유

- [Streamlit official caching](https://docs.streamlit.io/develop/concepts/architecture/caching): 재실행, data/resource 캐시, 사용자 간 공유/세션 범위.
- [Streamlit official secrets](https://docs.streamlit.io/deploy/concepts/secrets): 설정/자격증명 보호.
- [Streamlit official deployment](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app/deploy): 공개 합성 데모 배포 흐름.
- [FastAPI official docs](https://fastapi.tiangolo.com/tutorial/bigger-applications/): 차후 API layer.
- [FHIR R4 Observation](https://hl7.org/fhir/R4/observation.html), [DiagnosticReport](https://hl7.org/fhir/R4/diagnosticreport.html).
- [FDA GMLP](https://www.fda.gov/medical-devices/artificial-intelligence-enabled-medical-devices/good-machine-learning-practice-medical-device-development-guiding-principles).

**현재의 결론:** Python + Streamlit으로 즉시 합성 연구용 MVP를 시작할 수 있도록 설계한다. FastAPI/React/DB는 '있으면 좋아 보이는 스택'이 아니라 **측정된 요구에 의해 순차 도입**한다.
