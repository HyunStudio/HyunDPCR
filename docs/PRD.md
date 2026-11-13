# HyunDPCR — Product Requirements Document (PRD) v1.1

**일자:** 2026-11-13 | **상태:** Draft / research-only | **책임:** HyunStudio | **첫 제품:** 연구자용 합성데이터 시뮬레이터

## 0. Product contract

- **목표:** 시간 갱신 암 진단 위험과 평가 경로의 수학적 가정을 투명하게 비교하는 B2B 연구 워크벤치.
- **초기 사용자:** 임상역학/의료정보학 연구자, 의료 AI 검증 연구팀.
- **초기 제외:** 환자·보호자 자가진단, 의사에게 특정 검사/의뢰/기다리기를 권고, 자동 진료 실행, 소아 뇌종양 실사용, 생존율·병기·종양 성장 예측.
- **핵심 결과:** 특정 시점에서 **향후 90/180/365일 이내 암 진단 기록이 나타날 위험** (모델·데이터 검증 전에는 합성 위험만).
- **처리 상태:** `SYNTHETIC_DEMO` → `RETROSPECTIVE_RESEARCH` → `SHADOW_EVALUATION` → `CLINICAL` (각 단계 독립 승인·규제 게이트 필요).

## 1. 페르소나와 핵심 Job

1. **연구책임자:** 코호트/기간/모델을 명시해 재현 가능한 분석 및 보고서를 생성한다.
2. **의료통계 연구자:** 같은 데이터에서 기준선 모델과 시계열 모델의 보정/판별/순편익을 비교한다.
3. **병원 운영 연구자:** 가상 검사의 대기열 및 의뢰 시나리오의 차이를 검토한다.
4. **임상자문 의사:** 결과와 대안 행동 가정의 의료적 타당성·위해·중단 조건을 감사한다.

## 2. 핵심 사용자 여정

**UC-01 합성 사례 보기:** 데모 연구자가 가상 환자 1명의 시계열(증상 기록, 검사, 재진, 의뢰)을 열고 각 시점에 이용 가능했던 증거만 확인한다.

**UC-02 순차 위험 비교:** 첫 방문과 마지막 방문의 예측기간을 통일해 '현재 진단될 가능성', 'n일 내 진단기록 위험' 혼동 없이 표기한다. 점수 변화, 불확실성 및 입력 변경 추적이 가능해야 한다.

**UC-03 경로 시뮬레이션:** 임상적으로 허용되는 사전 지정된 전략에 대해서만 검사시간/추적주기/자원 용량을 바꾼 가상 경로를 몬테카를로로 비교한다. **임상 권고가 아닌 가정 기반 시나리오**라고 명시한다.

**UC-04 검증보고:** 시간 분할·사이트 분할 결과, 누수 검사, 캘리브레이션, DCA, 신뢰구간, 부분집단 비교, 버전 및 데이터 계보를 포함한 보고서를 다운로드한다.

## 3. 기능 요구사항 및 수용 기준

| ID | P | 요구 | 수용 기준 (연구용 MVP) |
|---|---|---|---|
| F-01 | P0 | 합성 코호트/환자 타임라인 | 고정 seed에서 재생성 가능하고 화면에 'synthetic' 표시 |
| F-02 | P0 | 세 가지 시점/기간의 정확한 정의 | 출력마다 index date, as-of date, horizon, endpoint 명시 |
| F-03 | P0 | 사건 기반 위험 갱신 | t 이후 정보 제거 시 t의 결과가 바뀌지 않음(누수 방지 테스트) |
| F-04 | P0 | 위험과 불확실성 | 점 추정치 + bootstrap/모델 가정에 근거한 구간 및 결측 표시; 가상 예시에서는 구간도 가상 명시 |
| F-05 | P0 | 가정/모델 감사 로그 | 모델 버전, feature cutoff, 데이터 버전, 시나리오 설정, 생성 시각 추적 |
| F-06 | P0 | 연구용 배지 및 면책 | 모든 위험 화면/CSV/보고서에 'research simulation; not for clinical use' 포함 |
| F-07 | P1 | 기준선 대비 비교 | 동일 코호트·기간·endpoint로 단순 기준선과 개선모델 비교 |
| F-08 | P1 | 상태전이·대기열 시뮬레이터 | 고정 seed, 환자 흐름 보존, 검사 역량 제약, 가정 변경 차이 표시 |
| F-09 | P1 | 테스트 정보가치 시나리오 | 검사 민감도/특이도·비용·위해가 필수 입력; 결측/무근거 시 계산 비활성 |
| F-10 | P1 | 보고서 내보내기 | 방법, 가정, 모델·데이터 버전, 제한점, 예제에 대한 재현 키 포함 |
| F-11 | P1 | 접근성 | 키보드 조작, 적절한 대비, 200% 확대, 도표 대체 텍스트, 색상 외 상태 구분 |
| F-12 | P2 | FHIR 연동 어댑터 | 모의 FHIR 번들 입력→정규화 이벤트 변환, 타입·시간·출처 검증 |
| F-13 | P2 | 다기관 연구 워크스페이스 | 데이터 접근 권한, 프로젝트 경계, 감사/내보내기 권한 분리 |
| F-14 | P2 | 임상용 API | 별도의 규제·검증 게이트가 충족되기 전까지 제품에서 비활성 |

**P0 완료 정의:** 외부 데이터/계정 없이 합성 사례 동작 + 누수·seed·표시·회귀 테스트 모두 통과.

## 4. 화면 IA (Information Architecture)

- **Overview:** 연구목표, 데이터 상태, 모집단, 연구기간, 전체 경고.
- **Patient Timeline (synthetic only):** 날짜별 symptom/encounter/observation/diagnostic report; 기록일과 발현일 분리.
- **Risk Evolution:** 조건부 예측 위험 시간변화, 예측 대상기간, 신뢰도, 기준선 모델.
- **Pathway Simulator:** 허용된 정책 집합, 대기시간/가용 검사 용량/비용 시나리오, 정해진 seed.
- **Decision Analysis:** EVSI/EVPI 또는 예상 손실, 민감도 분석, 지침 제한, 결과 설명.
- **Validation & Audit:** DCA, calibration, subgroup, data leakage, model card, export history.

## 5. 입력 계약(최소)

`patient_id` (가명·데모), `event_id`, `event_type`, `event_time` (임상 발생/검사 관찰 시점), `recorded_at`, `available_at` (모델 이용가능 시점), `code_system`, `code`, `value`, `unit`, `source`, `quality_flag`.

- symptoms: `reported_onset`은 알려진 경우에만; 확인되지 않으면 `unknown`.
- encounters: 진료 기록·재진·의뢰.
- tests: 주문 날짜, 채취 날짜, 결과 제공 날짜를 구분.
- outcome: 암 진단 코드/등록일, 암종, 확인 수준; 사망·탈락·추적 종료 기록.
- 임의의 입력 텍스트에서 LLM이 생성한 요약은 임상 확인 없이 '사실'로 등록하지 않음.

**모델 입력은 `available_at <= prediction_as_of`로 강제한다.** 진단 후 기록이나 검사 결과가 미래 시점에 들어오는 오류를 차단한다.

## 6. 비기능 / 보안

- **데모:** 사용자 계정 없이 동작하며 원격 추적·실제 PHI/PII 수집 금지.
- **연구 배포:** 암호화(전송/저장), RBAC, 최소권한, 감사로그, 비밀키 관리, 데이터 보존기간, 연구 별 프로젝트 분리, 유출 대응 계획.
- **품질:** 동일 seed/환경에서 재현성; 무효 입력/누락 날짜/역전된 시간 순서 검사; CI 단위·통합 테스트.
- **성능:** 데모 목표치(가설): 1,000개 합성 환자 위험 타임라인 비교가 일반 노트북에서 10초 내 처리되는지 측정(모델별 추후 재설정).
- **UX:** 결과 산출 근거/연구 상태/가정/관측된 것과 시뮬레이션을 분명하게 구분.

## 6.1. 구현 기술 및 기능 수용 기준 (Architecture v2.0과 연동)

**확정 MVP 구성:** Python 3.12+ + Streamlit multi-page UI + Pydantic + NumPy/pandas/SciPy + scikit-learn baseline + pytest. 합성 데이터 fixture를 먼저 사용하며 초기 실행경로는 향후 **app/Home.py**로 설계한다. API/React/운영 DB/실환자 데이터는 필수가 아니다.

| ID | 구현 요구 | 완료 조건 |
|---|---|---|
| ENG-01 | UI와 도메인 계층 분리 | Streamlit page는 src/hyundpcr/services의 typed function만 호출. 분석 함수 내 Streamlit import 없음 |
| ENG-02 | 합성 데이터·분석의 재현성 | 동일 seed + data/model version + parameters에서 동일 결과; 테스트 통과 |
| ENG-03 | 시간적 데이터 누출 차단 | available_at > as_of 사건의 추가·삭제가 현재시점 결과를 바꾸지 않음 |
| ENG-04 | 모델·데이터 출처 표시 | UI/CSV/JSON 결과에 as_of, horizon, endpoint, assumptions, run_id, research mode/경고 포함 |
| ENG-05 | 세션·캐시 데이터 안전 | Session State에는 데모 상태만; 전역 cache에 환자별 데이터/비밀정보를 저장하지 않음 |
| ENG-06 | 실행·테스트가 가능한 배포 | lockfile을 포함하고 clean install, pytest, Streamlit smoke test 확인 |
| ENG-07 | 멀티페이지 UX | Overview, Patient Timeline, Risk Evolution, Validation & Audit가 MVP 우선; Pathway/Options는 P1 |
| ENG-08 | 추가 API 전환 경계 | 향후 FastAPI route는 같은 Pydantic schema와 서비스를 호출, 현재는 HTTP 호출 불필요 |

**인터페이스 계약:** 결과에는 적어도 **analysis_mode, as_of, horizon_days, outcome_definition, model_version, dataset_version, uncertainty, warnings, run_id**가 포함된다. 종단 위험 분석에 미래 정보/임상 데이터가 혼입되면 계산을 차단한다.

**코드 현황:** 이 요구사항은 설계/수용기준이며 구현·임상 검증 완료를 뜻하지 않는다. 상세 도식 및 테스트는 [ARCHITECTURE v2.0](ARCHITECTURE.md)에서 관리한다.

## 6.2. v3.0 실행명세 및 이슈 추적

- [Implementation Blueprint v3.0](IMPLEMENTATION_BLUEPRINT.md): 개발 가능한 서비스·데이터 계약과 연구 모드별 모듈 분리.
- [UI_SPEC](UI_SPEC.md): Streamlit 화면·접근성·수치/의학적 경고 기준.
- [DATA_MODEL_AND_TESTS](DATA_MODEL_AND_TESTS.md): PRD ENG-01~ENG-08 및 누수/재현성/임상 안전 검증 조건.
- [SECURITY_RELEASE_GATES](SECURITY_RELEASE_GATES.md): 법적·윤리적 승인 단계, PHI 금지, 릴리스 차단 기준.
- **개발 순서:** [M0 issue #1](https://github.com/HyunStudio/HyunDPCR/issues/1) → [M1 issue #2](https://github.com/HyunStudio/HyunDPCR/issues/2) → [M2 issue #3](https://github.com/HyunStudio/HyunDPCR/issues/3).

**현황:** 모든 구현 항목은 미완료(설계 상태)이며, 테스트·임상 성능·상용 배포 완료를 의미하지 않는다.

## 7. 계산·기능 안전 규칙

- `SYNTHETIC_DEMO`는 모두 비임상적 가상 확률.
- 분석에 필요한 임상 데이터나 검사 성능 근거가 없으면 **모르는 값을 0으로 간주하지 않는다**.
- 지침상 즉각 의뢰가 필요한 상황에는 지연 선택지/검사 스킵 비교를 활성화하지 않음.
- 계산된 최저 비용을 최선의 의료행동으로 자동 추천하지 않음.
- 임상 기준 미충족 또는 외부 검증 실패 시 `CLINICAL` 기능 출시는 차단.

## 8. 성공지표(실적 아닌 목표 후보)

- 연구 프로토콜/코드/결과를 제3자가 재현할 수 있는 비율.
- 유출·가짜 숫자·미표시 가정 관련 치명적 오류 0건.
- 핵심 작업 완료시간 및 보고서 오류율의 기존 연구 workflow 대비 차이.
- 실제 데이터 단계에서 별도 정의할 캘리브레이션, DCA, 민감도/PPV, 공정성, 임상 안전 지표.
- 구매 의사 검증(유료 파일럿/명시적 구매자/조달 경로).

## 9. 오픈 질문

1. 첫 진료 코호트 정의: 체중감소 vs 비특이적 다증상? (임상의 협력 하 선택)
2. 영국 1차 의료·한국 병원·국제 연구 중 어느 단일 시장과 데이터 접근을 먼저 확보할 것인가?
3. 첫 연구의 암 진단 정의(등록자료 연계 가능성)와 희귀 암종 처리 방식은?
4. 실제 검사별 비용·위해·정확도 추정에 사용할 현지 자료는?
5. 제품·회사 법률상 소유권 및 라이선스는 어떻게 정할 것인가?

> PRD는 '개발할 기능'을 명시하는 계획이며 실제 구현·성능 검증을 의미하지 않습니다.
