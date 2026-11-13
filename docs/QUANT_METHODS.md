# HyunDPCR 정량 모델링 설계 v1.0

**목적:** 금융공학의 확률적 위험·의사결정 개념을 의료 데이터에 맞게 **검증 가능한 연구 가설**로 변환한다. 임상 치료·검사 권고 알고리즘의 명세가 아니다.

## 1. 먼저 분리할 추정 대상(Estimands)

A. `P(recorded cancer diagnosis by t+H | observed history up to t, currently without recorded cancer)`
: t 시점까지의 기록을 알고 있을 때 H일 이내의 **기록된 암 진단** 위험.

B. `P(site j diagnosed by t+H | history)`
: 암종별 위험(다른 암종·사망을 경쟁 사건으로 고려).

C. `E[diagnostic pathway time | observed design]`
: 진료→검사→의뢰→암 진단/비진단/추적종료 경로 시간.

D. `E[loss(action) | assumptions]`
: **의학적으로 허용된 행동들 사이의** 기대 비용·위해·정보 가치를 비교하는 가상 시나리오. 실제 인과적 효과는 별도 식별이 필요.

**다름에 주의:** A/B는 암 생물학적 발병 시점, 종양 병기 변화, 생존 기간, '안전한 대기 시간'이 아니다.

## 2. 금융공학 → 의료 모델 매핑

| 원 개념 | 의료적 번역 | 설계/주의 |
|---|---|---|
| Filtering / Bayesian updates | 순차 임상증거 이후 위험 분포 갱신 | 임상자료의 관측과정(검사 대상 선택·재진 편향)을 명시해야 함 |
| Survival / hazard dynamics | 방문/진단/사망 시간 사건모형 | 암종별 경쟁사건, 좌측절단·우측검열·관측 기간 고려 |
| Scenario Monte Carlo | 검사/의뢰/대기열의 가능한 경로 분포 | 임상적으로 근거 없는 시뮬레이션 결과는 효과 추정이 아님 |
| Value of Information | 추가 검사가 선택을 얼마나 바꾸는가 | 검사 위해·자원·선택편향·실행 가능성을 비용과 함께 분석 |
| Tail-risk / CVaR | 긴 진단 지연 또는 심각한 불리한 결과의 하방위험 | 환자별 위해를 금융수익처럼 환산하지 말고 임상 공동설계 |
| Portfolio/capacity optimisation | 제한된 검사 자원·예약용량의 대기열/경로 비교 | 개인에게 불이익을 주는 효율 최적화 금지; 공정성 제약 |

**독창성 메모:** 경쟁위험 분석과 의료 value-of-information은 기존 기술이다. '새로운 금융공학 알고리즘'이라고 주장하지 않는다.

## 3. 모형 후보와 기본식

### 3.1 Sequential conditional risk

관측가능 병력 `H_t`와 알려진 결과 상태를 이용해 예측한다.

```text
p_t(H) = P(T_diagnosis <= t+H, cancer type in set J | H_t, cancer not yet recorded by t)
```

- 매 예측마다 시간 `t`와 수평선 `H`를 저장한다.
- 단순 베이지안 업데이트는 개념 데모에 유용하나, 실제 EHR의 증상·검사 이벤트는 조건부 독립이 아니다. 임의의 likelihood ratio를 계속 곱해서 임상 확률이라고 표시하면 안 된다.
- 검사 주문/결과가 기록되는 과정 자체가 임상 의사결정에 의해 달라진다. informative visit/testing process를 보정하거나 민감도 분석한다.

### 3.2 Competing risks

암종 j의 원인별 hazard `lambda_j(t|X)`, 사망 등 경쟁사건을 포함하는 생존함수 `S(t)`에서:

```text
F_j(t) = integral_0^t S(u-) * lambda_j(u) du
```

`F_j(t)`는 j 암종의 누적 진단 위험. 복수 암종·사망을 단순 독립 이항으로 합산하지 않는다. Fine–Gray, cause-specific hazards, multi-state 또는 discrete-time multi-class hazard를 적절한 평가 대상에 맞춰 비교한다.

### 3.3 Landmark / dynamic prediction

`t = 0, 30, 60, ...` 또는 방문시점에서 누적정보만 사용해 동적 업데이트. 학습·검증 예제는 환자 수준으로 분리한다. `available_at` 기준 미래 정보 누출을 방지한다. t 이후 재진 여부를 특징으로 쓰지 않는다.

### 3.4 Value of information (VOI)

```text
Current optimum loss = min_a E[L(a, theta) | H_t]

Expected loss after test k =
  E_y[ min_a E[L(a, theta) | H_t, Y_k=y] ] + C_k + Harm_k

Expected value of sample information =
  Current optimum loss - E_y[ min_a E[L(a, theta) | H_t, Y_k=y] ]
```

- 행위 `a`는 임상적으로 허용된 것만.
- `C_k`는 검사·시간·자원 비용; `Harm_k`는 검사 위해를 의미. 같은 측정 단위/효용 함수가 명시되어야 계산 가능.
- **EVSI > 검사 비용**이 곧바로 임상 권고를 뜻하지 않는다. 의료진·환자 선호·지침의 상위 제약을 준수한다.
- 관찰 데이터의 실제 검사 여부는 무작위가 아니므로, 인과효과처럼 제시하려면 정책평가의 식별 가정·전문가 검증 필요.

### 3.5 Monte Carlo and tail risk

- 경로 상태: `first_contact → reassessment/test → referral → diagnosis/non-diagnosis/censoring`.
- 대기열/검사 용량/기간을 가상 설정하고 반복 시뮬레이션; 결과는 분포·구간·민감도 분석으로 제공.
- 예: `CVaR_alpha(D)` = 상위 지연 구간에서의 조건부 평균 지연(정의·유한 표본 편향 설명). '진단 지연의 질병 위해'는 시뮬레이터가 자동으로 입증하지 못한다.
- worst-case 대안을 숨기지 않고 모델 불확실성과 구조적 시나리오(검사 부족/기록 누락)도 함께 제공.

## 4. 실증연구의 기준선

1. Prevalence/연령·성별·증상 기반 단순 모형.
2. Logistic regression at fixed horizon (검열 처리 전략 명시).
3. Cause-specific Cox, Fine–Gray 또는 discrete-time competing risk.
4. Gradient boosting/다른 ML (충분한 데이터·튜닝·누수 통제 조건).
5. **Ablation**: (a) 정적, (b) 종단, (c) +관측과정, (d) +uncertainty, (e) +decision analysis.
6. QCancer 같은 외부 모델은 적법한 접근 및 같은 모집단/endpoint/horizon을 맞출 수 있을 때만 별도 비교.

## 5. 편향·식별 위협

| 위협 | 해석 문제 | 대응 |
|---|---|---|
| Selection/spectrum bias | 검사를 받은 사람만 학습하면 일반 환자에 오적용 | 출발 코호트·검사 의뢰 여부·분모 고정 |
| Informative observation | 많이 방문하는 사람이 더 많이 발견됨 | 방문/검사 관측과정 모델링·민감도 분석 |
| Immortal-time/lead-time bias | '조기 진단' 효과 과대평가 | 정확한 time zero, landmark, 정책평가 별도 |
| Outcome misclassification | 암 '미진단'을 암 '부재'로 간주 | 등록자료 확인·충분한 추적·검열 |
| Data leakage | 결과·진단 후 코드가 모델 입력에 혼입 | feature cutoff + 테스트 + 데이터 provenance |
| Confounding by indication | 검사가 위험해 보이는 환자에게 더 수행됨 | 인과 모델/target trial emulation 필요 |
| Rare subtype events | 불안정한 암종별 확률 | 수축/계층모형·정밀도 보고·최소 표본 제한 |
| Transportability | UK 개발모델을 한국에서 그대로 사용 | 현지 재보정·외부 평가·지침 현지화 |

## 6. 최소 실험 순서

**E1 (합성):** 역학/시간 일관성 테스트. 베이지안 필터와 survival landmark 기준선 비교. 어떠한 숫자도 의학적 위험 수치로 언급하지 않음.

**E2 (후향적):** 데이터 이용권/윤리 승인, 미리 정한 분석 계획과 결과 정의. discrimination + calibration + decision curves + subgroup 평가. 시간/기관별 외부 검증.

**E3 (정책 평가):** 데이터에서 행동 정책을 식별할 수 있는지 확인. 식별 불가능하면 *illustrative simulation only*로 제한. 임상 자문과 함께 비용·위해·효용함수 정의.

**E4 (전향적 silent-mode):** 실제 의료진 판단과 병행하여 임상 행동을 바꾸지 않고 성능/사용성 측정. 정당한 안전성·규제 근거 없이는 실제 의뢰 정책 배포 금지.

## 7. 핵심 참고문헌

- [QCancer 공식](https://www.qcancer.org/) — 이미 경쟁위험 모델을 사용하는 범암종 도구.
- [TrajOnco 2026](https://doi.org/10.1038/s41746-026-03377-9) — 종단 EHR 기반 다중암 조기 예측.
- [BMJ clinical prediction model development guide](https://www.bmj.com/content/386/bmj-2023-078276) — 단순모형, 결정곡선, 유효성.
- [Vickers et al. 2016 net benefit](https://doi.org/10.1136/bmj.i6).
- [TRIPOD+AI 2024](https://doi.org/10.1136/bmj-2023-078378).
- [PROBAST+AI 2025](https://doi.org/10.1136/bmj-2024-082505).
