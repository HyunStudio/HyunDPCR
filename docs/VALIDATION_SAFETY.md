# 임상 연구 검증·안전성 계획 v1.0

**작성:** 2026-11-13 · **대상 단계:** 연구 기획 → 승인된 후향적 검증 → silent-mode → 규제 검토

> 본 문서의 검증 단계는 계획이며 완료된 임상 성능, 환자 대상의 사용 승인, 의료기기 허가를 의미하지 않습니다.

## 1. 임상연구 질문

**RQ1:** 비특이적 증상으로 성인 1차의료를 방문한 환자에게, 현재까지의 병력을 이용한 종단 모형이 고정 시점 모형 대비 **정해진 기간 내 암 진단**의 예측 성능/보정을 개선하는가?

**RQ2:** 예측시점에 실제 이용 가능한 정보만 사용하고, 재진·검사 관측과정을 고려할 때 예측이 얼마나 달라지는가?

**RQ3:** 명시적으로 허용된 진료/검사 전략들의 정보가치·진단 경로 시간·자원 사용 분석이 기존 연구자 workflow 대비 재현성과 의사결정 이해를 개선하는가?

**RQ4 (후속):** 실제 행동의 효과를 추정할 식별 조건이 충족되는가? 안 되면 결정분석은 **가상 시뮬레이션**으로만 표기한다.

## 2. 임상 코호트 프로토콜(사전등록 대상)

| 구성 | 권고 정의 / 반드시 확정할 사항 |
|---|---|
| Target population | 성인(예: 18세 이상) 1차의료 비특이적 증상 첫 진료 코호트. 암 관련 긴급 경로는 현지 지침에 따라 별도 분리 |
| Time zero | 사전 정의한 최초 적격 증상 진료일. 재진 방문만 선택하지 않도록 분모 유지 |
| Exclusion | 이미 암이 진단된 사람 등 연구 목적에 따라 사전명시; 재발/전이는 별도 문제 |
| Event | 등록/진료코드/병리 등으로 확인된 암 **진단 날짜**(정의·코드셋 고정), 암종별 보조 결과 |
| Competing events | 사망, 다른 암종의 우선 진단 등; 결과 정의에 맞게 처리 |
| Follow-up | 90/180/365일 등 사전 지정. 검열·등록 종료·상실·이탈·데이터 완결성 처리 |
| Time-varying inputs | 증상 코드, 진료, 검사 주문/결과, 의뢰, 기존 진단 등; 해당 시점의 `available_at`만 허용 |
| Evidence gaps | 발현일 vs 최초 기록일, 진단일 vs 병기/치료일, 재진 누락과 암 비발견 구분 |
| Geography | 최초 실증 시장과 기관의 지침/코딩/진료방식으로 한정; 국가 간 일반화 금지 |

**표본수:** 사건수·희귀 암종·변수 수·외부 검증 요구를 고려해 설계 단계에 임상의·의료통계학자가 산정. 임의의 N으로 '충분한 데이터'라고 주장하지 않는다.

## 3. 데이터 품질·이용권

- 데이터 소유자와 DUA/IRB 또는 적절한 윤리 승인을 확인하기 전 실제 환자 데이터를 가져오지 않는다.
- 최소 필요 변수만 접근·사용. 익명/가명화 방식·재식별 가능성 검토. 법적 요구사항은 관할권별(예: 한국 개인정보보호법, UK GDPR, HIPAA 등 해당 시) 법률 검토.
- 최초 증상일은 'reported onset', 진료 기록일은 'recorded', 검사 확정일은 'result available'로 별도 관리.
- 데이터 사전 및 phenotype 알고리즘(암종·증상·의뢰·진단)을 버전 관리.
- 기관 외부 유출 차단, 암호화, RBAC, 보관/파기, 연구별 접근·내보내기 감사.

## 4. 모델 검증 프로토콜

1. **분할:** 환자 단위 중복 방지 + 시간 기반 테스트 + 기관/지역별 외부 검증(가능한 경우). 튜닝과 최종 평가는 분리.
2. **누수:** `available_at <= as_of` 강제, 진단 후 코딩/의뢰 결과를 예측 이전 특징에 포함 금지.
3. **기준선:** 단순 모델, survival/discrete-time competing-risk, 종단 모델을 같은 데이터·코호트·예측기간에서 비교.
4. **보정:** calibration plot, calibration-in-the-large, slope, Brier / integrated Brier (적절한 검열 보정), 불확실성 구간.
5. **판별:** AUROC, AUPRC(발생 빈도 보고), C-index/time-dependent AUC(해당 시), 민감도·특이도·PPV/NPV 및 신뢰구간.
6. **임상 순편익:** 예측 기반 전략과 단순 기준선을 decision curve(net benefit)로 비교. 적절한 임상 임계값은 의료진과 사전 합의; 전체 암종에 보편적인 '3%' 기준을 임의 적용하지 않음.
7. **유해성:** 누락 암 사례, 낮은 위험으로 잘못 안심시키는 결과, 추가 검사·불필요 의뢰 증가, 검사 위해, 지연, 모델 오버라이드 사례.
8. **공정성:** 연령·성별·사회경제·지역·기관·희귀 암종 등 법적/윤리적으로 허용되는 집단별 성능 차이. 표본 부족 시 부정확한 집단별 결론 금지.
9. **외부유효성:** 기관/지역/시간 간 distribution shift, calibration drift 및 모델 중단/재보정 규칙.
10. **재현성:** 보고서에 cohort SQL/코드 버전, data snapshot, model version, seed, outcome definition, 평가시점, confidence interval 부착.

**보고:** [TRIPOD+AI](https://doi.org/10.1136/bmj-2023-078378) 적용. [PROBAST+AI](https://doi.org/10.1136/bmj-2024-082505)로 편향·적용성 점검. TRIPOD+AI는 보고 지침이며 임상 인증·성능 기준이 아니다.

## 5. 반사실적 정책분석 주의사항

- 검사를 많이 받는 이유는 원래 고위험이었기 때문일 수 있다(**confounding by indication**).
- 단순 관찰자료에서 '검사를 더 빨리 하면 암이 더 빨리 발견된다'는 인과적 개선을 직접 추정하지 않는다.
- 정책 효과를 주장하려면 target trial emulation, 시간가변 교란, 선택 및 검열 조정, 자료 지원 범위(positivity), 정책의 실행가능성 등의 별도 설계 필요.
- 식별 불확실성·의료진 검토 미충족 시 임상 진료 정책을 자동 추천하지 않는다.

## 6. 안전 위험 레지스터

| ID | 잠재 위해 | 탐지 / 통제 | 출시 차단 여부 |
|---|---|---|---|
| R1 | 잘못 낮은 위험도로 임상 안심 제공 | 모든 데모에 비임상 경고, 환자용 배포 금지, 임상 게이트 | Yes |
| R2 | 지침상 즉시 평가 대상의 진료 지연 | 지침 우선·최적화 선택집합 제한, 의료진 승인 | Yes |
| R3 | 미래 기록 유출로 모델 성능 부풀림 | timestamp availability invariant, 테스트/감사 | Yes |
| R4 | '진단'을 '발병/진전'으로 오해 | 모든 결과에서 endpoint·기간 명시 | Yes |
| R5 | 재진·검사를 많이 받은 집단의 선택편향 | 출발 코호트·관측과정 분석, subgroup 검증 | Yes for clinical |
| R6 | 저빈도 암종 확률이 불안정 | 최소 사건수·계층화·신뢰구간·출력 보류 | Yes for affected subtype |
| R7 | 실환자 기록의 개인정보 유출 | 개인정보 비수집 데모, 비식별화·보안·계약 | Yes |
| R8 | 과도한 검사/의뢰 증가 | DCA·검사 위해·자원 비용·실제현장 평가 | Yes for clinical |
| R9 | 모델/지침 drift | 현장 모니터링 및 rollback 계획 | Yes for clinical |
| R10 | 분석 시뮬레이션을 인과 효과로 오해 | 시나리오 vs 추정 효과 구분 및 한계 주석 | Yes |

## 7. 규제·윤리 게이트

**연구 데모:** 가상 데이터로만 진행, 개인정보 불수집, 의료적 조언 제공 금지.

**실제 데이터 연구:** 기관별 승인·동의/면제 요건·DUA·연구 보안 환경·책임자 명시.

**임상 워크플로우 도입:** intended use, target users, 진료 의사결정 영향, 법적 의료기기 범위, 품질관리, 사건보고, 보험, 책임 주체를 관할권별 전문 검토. 미국은 [FDA Clinical Decision Support Software guidance, 2026](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software) 참고. 영국은 [NICE NG12](https://www.nice.org.uk/guidance/ng12) 등 현지 지침 적용. [ISO 14971](https://www.iso.org/standard/72704.html)의 위험관리 원칙 검토.

**현장 배포 판단:** 성능이 좋아 보이는 연구 논문 한 편만으로 임상 허가/유효성을 인정하지 않는다. 모델뿐 아니라 사용자와 업무흐름에서의 위해·순편익을 별도로 평가한다.

## 8. Go / No-Go

| 단계 | 진입·진행 조건 | No-Go 조건 |
|---|---|---|
| Synthetic MVP | 수학적·시간적 일관성 테스트, 연구 경고/표시 확인 | 무근거 확률을 임상 수치처럼 표현 |
| Retrospective | 정식 데이터 접근·사전계획·임상 자문 | 데이터 권한/endpoint 검증 불가능 |
| External validation | 독립 장소/시간에서 보정·편향·순편익 평가 | 핵심 집단에서 미허용할 성능/불확실성 |
| Silent mode | 기관·윤리·법적 승인 및 운영 모니터링 | 실제 치료/검사 판단을 조용히 변경하는 구현 |
| Clinical commercialization | 관할권별 요구사항 및 안전·성능 근거 충족 | 규제 미준수 또는 위해 통제 부족 |

## 참고

- [TRIPOD+AI](https://doi.org/10.1136/bmj-2023-078378)
- [PROBAST+AI](https://doi.org/10.1136/bmj-2024-082505)
- [BMJ clinical prediction model guide](https://www.bmj.com/content/386/bmj-2023-078276)
- [FDA Good Machine Learning Practice](https://www.fda.gov/medical-devices/artificial-intelligence-enabled-medical-devices/good-machine-learning-practice-medical-device-development-guiding-principles)
