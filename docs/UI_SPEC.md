# HyunDPCR — Streamlit UI/UX 설계 v1.0

**Target:** 합성 연구 데이터 전용 Streamlit 다중 페이지 앱. **Status:** 계획, 아직 실행 화면 아님.

## 1. 제품 전체 내비게이션

```text
HYUND PCR / Dynamic Pan-Cancer Risk
Research-only • Synthetic demo • Not for clinical use

Overview
  > 현재 연구모드, 가상 코호트, 연구기간, 모든 가정의 상태
Patient Timeline
  > 실제 발생 추정 / 기록 / 분석 가능 시점 별도
Risk Evolution
  > 같은 예측 대상/수평선의 날짜별 위험 추정 및 누출 방지
Pathway Simulator (P1)
  > 합성 대기열 및 시나리오 비교
Options Lab (P1)
  > 가정 기반 가치정보/기대 손실/민감도 분석
Validation & Audit
  > 모델상태·데이터버전·결과검증·보고서 다운로드
```

## 2. 메인 화면 와이어프레임

```text
┌────────────────────────────────────────────────────────────────┐
│ HyunDPCR   Research Workbench                  [SYNTHETIC DEMO]│
├──────────────┬─────────────────────────────────────────────────┤
│ NAV          │ Cohort: synthetic-v1        Model: toy-v1      │
│ Overview     │ Endpoint: recorded cancer diagnosis 90 days   │
│ Timeline     │                                                 │
│ Risk         │ Example scenario / as-of time / horizon         │
│ Pathway      │ ┌──────────┐ ┌─────────────┐ ┌──────────────┐   │
│ Options      │ │Risk toy  │ │Uncertainty  │ │Data quality  │   │
│ Audit        │ │demo only │ │demo only    │ │Pass / Fail   │   │
│              │ └──────────┘ └─────────────┘ └──────────────┘   │
│              │                                                 │
│              │ Risk-by-as-of chart + textual trend + table     │
│              │ Time-aware events + missingness / quality flags │
│              │                                                 │
│              │ Assumptions | Comparison | Export               │
├──────────────┴─────────────────────────────────────────────────┤
│ Not patient advice. No clinical model or performance validation │
└────────────────────────────────────────────────────────────────┘
```

## 3. 화면별 구체 상호작용

| Page | Inputs | Visuals | Disabled states / warnings |
|---|---|---|---|
| Overview | 가상 코호트, cohort seed | 샘플 현황, 분석모드, 정의, 연구목적 | 실제 환자 정보 업로드 없음 |
| Patient Timeline | 가상 사례 ID, 표시 기간 | 기록시점/사용가능시점 타임라인, 표 | 증상 발현일 모르면 'unknown' |
| Risk Evolution | 시점(as-of), 90/180/365일 수평선, toy model | 조건부 가상 위험 추세, 구간, 원천 이벤트 | 미래 관측 제외, 학습 안 된 모델 표시 |
| Pathway Simulator | 재진 주기·검사 역량·대기열 가정·seed | 샘플 경로 분포, 대기기간 비교 | 실제 행동 효과로 표시 금지 |
| Options Lab | 연구용 검사 성능·비용·위해·허용 행동 | VOI 범위·민감도·경계 조건 | 가정/의학적 허용 행동 정의 없으면 계산 금지 |
| Validation & Audit | run ID / model ID | 입력 스냅샷, 실행 버전, 모형 검증 상태, export | 검증되지 않은 모델에 정확도 라벨 금지 |

## 4. 상호작용 동작 계약

- `st.navigation` 및 `st.Page` 사용(단일 entrypoint) 또는 파일기반 페이지 구조 중 **하나**만 선택해 일관성 유지.
- `st.session_state`에는 **합성 사례 ID, 선택된 as-of, horizon, seed** 등 가벼운 상태만 저장.
- 계산 버튼을 누르기 전 필수 입력을 검증하고, 결과 화면에는 사용한 입력·가정·모델을 다시 표시.
- 슬라이더나 selectbox를 조정하더라도 이전 분석의 `run_id`/결과와 섞이지 않도록 명시적으로 재계산.
- `st.cache_data`는 공개 합성 fixture에서만 허용. 실제 환자 데이터·권한 부여 상태에 대한 캐싱 금지.
- CSV/JSON 내보내기에는 분석 기준시점/수평선/데이터 및 모델 버전, 경고/한계를 포함.
- 시각화는 화면 확대와 텍스트 전용 설명, 테이블을 병행하여 고대비·저시력 사용성 유지.

## 5. 메시지 카피

- Primary warning: **"SYNTHETIC RESEARCH DEMO — NOT FOR CLINICAL USE"**
- Toy estimate: **"Illustrative, uncalibrated model output. Not a patient's cancer probability."**
- No evidence: **"Insufficient supported assumptions to calculate value of information. No recommendation is generated."**
- Timestamp: **"Only events known on or before this as-of date are included."**
- Outcome: **"Recorded cancer diagnosis during the selected horizon; not cancer biological onset or tumour progression."**

## 6. 다국어·접근성 기준

- MVP 언어: 한국어 기본 + 필수 의학/수학 용어는 영어 병기.
- 폰트 200% 확대에서 주요 조작/결과를 확인할 수 있어야 한다.
- 데이터시각화는 색만으로 상태를 구분하지 않음; 적어도 표·수치·텍스트 설명을 제공.
- 키보드로 주요 컨트롤에 접근, 정보 입력 오류는 텍스트와 연결해 표시.
- 로그/식별자/진단 결과는 확대 화면에도 라벨이 잘리지 않도록 반응형 레이아웃.
- 실제 접근성 적합성 주장은 사용성 테스트 이후에만 한다.

## 7. 테스트 시나리오

1. 새 페이지 이동에도 합성 사례 선택 유지.
2. 동일 화면에서 기간 변경 시 제목/그래프/내보내기 결과가 일치.
3. 미래 결과를 가상 데이터에 넣어도 현재 시점 결과에 영향 없음.
4. 설명 없이 숫자만 표시되는 차트 없음.
5. 의료적 가정 누락 시 Options Lab의 점수·추천값이 비활성화.
6. 모든 다운로드 파일에 `SYNTHETIC`, `NOT FOR CLINICAL USE` 포함.
7. Streamlit `AppTest` 자동 테스트 + 확대/키보드/스크린리더 수동 점검.

References: [Streamlit multipage docs](https://docs.streamlit.io/develop/concepts/multipage-apps/page-and-navigation), [Streamlit AppTest](https://docs.streamlit.io/develop/concepts/app-testing).
