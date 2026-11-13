# HyunDPCR 아키텍처·데이터 계약 v1.0

**상태:** 설계안. 실제 구축된 시스템이 아님. 초기에는 단일 레포/로컬 합성 데이터로 구현하고 검증된 요구가 생기면 분리한다.

## 1. 개념 아키텍처

```text
Research UI (timeline / risk / scenario / audit)
            |
       API boundary
            |
   Input normalization + provenance gate
            |
   ------------------------------
   |      |       |       |     |
 Signal  Options  Pathway  Guard  Reports
   |      |       |       |     |
   -------- Model / rules layer --
            |
  Cohort metadata + secure research store
            |
  Synthetic generator (MVP)
  Approved de-identified EHR / FHIR adapters (future)
```

**분리 원칙:** 위험 예측 엔진이 검사 권고를 직접 결정하지 않는다. 임상 지침/허용된 행동 제약은 결정분석보다 먼저 적용한다.

## 2. 기술 선택 (변경 가능한 초안)

- **실험/백엔드:** Python 3.12+, Pydantic 검증, FastAPI (선택), pandas/polars, numpy, scipy, scikit-learn + 통계적 생존/경쟁위험 패키지(선정 시 검증).
- **웹 UI:** 빠른 PoC는 Streamlit 또는 React/TypeScript + 시각화. 단일 기능 구현 우선. 브랜드와 제품은 프레임워크에 종속되지 않음.
- **저장:** 합성 데이터는 fixture 파일/SQLite. 실제 승인된 데이터는 기관 관리 환경, 암호화 및 기관 DUA에 따름.
- **보고:** Markdown/HTML/CSV + 기계 판독 JSON, 모든 결과에 model card / cohort card 동봉.
- **표준 연동 (후속):** FHIR R4의 `Patient`, `Encounter`, `Condition`, `Observation`, `ServiceRequest`, `DiagnosticReport` 매핑. 기관별 프로파일과 실제 데이터 계약 필요.

## 3. 분석 레코드

```json
{
  "event_id": "synthetic-event-001",
  "patient_id": "synthetic-person-001",
  "event_type": "observation",
  "clinical_time": "2026-01-03T12:00:00Z",
  "recorded_at": "2026-01-03T13:00:00Z",
  "available_at": "2026-01-04T09:00:00Z",
  "code_system": "DEMO",
  "code": "SYMPTOM_X",
  "value": "DEMO_ONLY",
  "source": "synthetic-generator",
  "quality_flags": []
}
```

- **이벤트 처리:** `available_at`이 예측시점보다 이후인 사건은 특징값에서 제외.
- **증상 시점:** 발현일이 불확실하면 별도 `reported_onset_at`과 `onset_confidence`를 둠. 기록일과 혼동 금지.
- **검사:** 주문/수행/결과 발행/환자·의료진에게 가용해진 시점을 구분.
- **암 결과:** 결과 데이터의 기록 여부·진단출처·암종 코딩·재발/전이 제외 규칙을 버전 관리.
- **데모 가상 수치:** 실제 임상 진단 성능으로 표시 금지.

## 4. API 초안 (제안; 구현 전)

| Endpoint | Action | Guard |
|---|---|---|
| `GET /v1/demo/cases` | 가상 사례 조회 | synthetic only |
| `POST /v1/risk/evaluate` | 특정 as-of 시점/기간의 위험 추정 | allowed model + dataset, trace ID |
| `POST /v1/pathway/simulate` | 가상 경로 시뮬레이션 | guideline restrictions + provenance |
| `POST /v1/options/compare` | 가정 기반 VOI 시나리오 비교 | validated costs, action set, disable when missing |
| `GET /v1/audits/{run_id}` | 모델·가정·입력 이력 | RBAC + research access boundary |
| `GET /v1/reports/{run_id}` | 비임상 보고서 | status watermark + version |

**응답 필수 필드:** `analysis_mode`, `as_of`, `horizon_days`, `outcome_definition`, `population_definition`, `dataset_version`, `model_version`, `assumptions`, `uncertainty`, `warnings`, `run_id`.

## 5. 개발 레포 구조 제안 (미구현)

```text
README.md
docs/
  PRODUCT_STRATEGY.md
  COMPETITIVE_LANDSCAPE.md
  PRD.md
  QUANT_METHODS.md
  VALIDATION_SAFETY.md
  ROADMAP_BUSINESS.md
  ARCHITECTURE.md
app/                  # future UI
src/hyundpcr/
  data/               # normalization, time provenance
  models/             # risk/competing risk
  decisions/          # VOI and permitted action sets
  simulation/         # event-time and queues
  governance/         # logging, model and data cards
tests/
  unit/
  integration/
  leakage/
  safety/
examples/synthetic/
```

## 6. 필수 테스트

- T-01: 시점 `t` 뒤에만 존재하는 임상사건으로 예측 결과가 변하지 않는다.
- T-02: 동일 seed + 모델/데이터 버전/가정에서 결과가 재현된다.
- T-03: 음수 기간, 역전된 진료 날짜, 잘못된 코드·단위가 오류로 처리된다.
- T-04: 결과에 'diagnosis risk'와 기간 및 가상/후향적/임상 모드가 표시된다.
- T-05: 미기재 검사 성능/비용이 있을 때 의학적 VOI 숫자를 생성하지 않는다.
- T-06: 긴급 의뢰·지침상 필수 평가 시 '기다리기' 옵션이 차단된다(실제 지침 구현 시 전문 검토 필요).
- T-07: 동의/계약 없이 PHI 파일·로그·저장소 커밋을 수집하지 않는다.
- T-08: 내보내기 파일에서도 분석 상태 및 모든 한계가 남는다.
- T-09: 키보드·화면 확대·스크린리더 텍스트와 대비가 확인된다.
- T-10: model card·data card에 버전·대상 코호트·제한점이 기록된다.

## 7. 보안·운영 원칙

- 공개 GitHub 저장소에 환자식별자료/계약상 제한 데이터/토큰/접근키 업로드 금지.
- 후속 B2B 도입은 프로젝트·기관별 테넌트 격리, 인증·인가, 감사/접근로그, 키관리, 백업·삭제·사고 대응 설계가 선행.
- 민감정보는 최소 접근과 최소 보관. 데이터 이용 및 국제 이전 허용 여부를 관할 기관과 확인.
- 모델 업데이트 시 결과 재현성과 버전 추적을 유지. 기준선 성능 악화 시 rollback.
- 외부 LLM 호출은 환자 원자료의 제3자 전송을 수반할 수 있으므로 기본 비활성; 승인·계약 후 별도 검토.

## 8. 참고

- [HL7 FHIR R4 Observation](https://hl7.org/fhir/R4/observation.html)
- [HL7 FHIR R4 DiagnosticReport](https://hl7.org/fhir/R4/diagnosticreport.html)
- [FDA GMLP](https://www.fda.gov/medical-devices/artificial-intelligence-enabled-medical-devices/good-machine-learning-practice-medical-device-development-guiding-principles)
