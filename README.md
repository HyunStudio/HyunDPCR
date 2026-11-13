# HyunDPCR

**Dynamic Pan-Cancer Risk & Diagnostic Decision Intelligence**

> **Status: Concept / research prototype planning (2026-11-13).** No clinically validated predictive model, real-patient deployment, regulatory clearance, or demonstrated performance is claimed. The repository currently contains planning documents only.

HyunDPCR is a proposed **research-first, B2B diagnostic decision-intelligence platform** that combines longitudinal cancer-risk estimation with uncertainty-aware diagnostic-pathway simulation. Its quantitative-finance-inspired toolkit explores **sequential Bayesian updating, event-time/stochastic models, value of information, Monte Carlo scenario analysis, and downside-risk measures**—subject to clinical validity, human oversight and explicit assumptions.

**Product question:** Not simply “How high is the cancer risk?” but **“How do new evidence, diagnostic choices, and time affect uncertainty and expected outcomes?”**

## 한국어 소개

HyunDPCR은 비특이적 증상으로 진료받는 **성인 환자의 범암종 진단 경로**를 연구하기 위한 의사결정 인텔리전스 플랫폼 기획입니다. 금융공학에서 사용하는 확률·위험·의사결정 분석을 의료 시계열 데이터에 맞게 적용하되, 의료적 타당성과 데이터 검증을 우선합니다.

**첫 번째 고객 가설:** 대학병원 임상연구팀 및 연구기관(분석·시뮬레이션·감사 가능한 결과 보고). 환자 대상 자가진단 앱이나 자동 진료 권고로 시작하지 않습니다.

## Proposed product modules

| Module | Purpose | MVP status |
|---|---|---|
| **Signal** | Time-updated, horizon-specific cancer-diagnosis risk and uncertainty | Planned |
| **Options** | Compare clinically permissible investigation strategies using value-of-information scenarios | Planned / simulation-only |
| **Pathway** | Model diagnostic encounters, test queues and referral delay | Planned / simulation-only |
| **Guard** | Audit trail, calibration, subgroup evaluation, monitoring and safety constraints | Planned |

## Documentation

- [Product strategy & positioning (KO)](docs/PRODUCT_STRATEGY.md)
- [Product requirements document / PRD (KO)](docs/PRD.md)
- [Quantitative methods & scientific hypotheses (KO)](docs/QUANT_METHODS.md)
- [Evidence, competitors & references (KO)](docs/COMPETITIVE_LANDSCAPE.md)
- [Clinical research validation & safety (KO)](docs/VALIDATION_SAFETY.md)
- [Commercialisation plan & milestones (KO)](docs/ROADMAP_BUSINESS.md)
- [Architecture, data contract & security (KO)](docs/ARCHITECTURE.md)

## Technical architecture decision (2026-11-13)

**MVP: Python 3.12+ + Streamlit.** The initial research app will directly call a tested, UI-independent Python package; **FastAPI, React/Next.js, PostgreSQL, and hospital integrations are not MVP requirements**. Optional scientific libraries (NumPy, SciPy, pandas, scikit-learn, and survival-analysis tools when justified) support reproducible synthetic-data experiments.

~~~mermaid
flowchart LR
  ST["Streamlit research UI"] --> SV["Python domain services"]
  SV --> SIG["Signal: risk timeline"]
  SV --> PTH["Pathway: simulation"]
  SV --> OPT["Options: value of information"]
  SV --> GD["Guard: audit and safety"]
  SIG --> SYN["Synthetic research data"]
  PTH --> SYN
  OPT --> SYN
  GD --> REP["Watermarked research exports"]
~~~

**Development status:** Architecture only; executable application files and dependencies have **not** been committed. MVP entrypoint is planned as app/Home.py; implementation details, planned repository structure, time-provenance gate, deployment boundaries and future FastAPI migration are in [Architecture v2.0](docs/ARCHITECTURE.md).

## Scope and boundaries

- **Initial use:** Synthetic-data research demonstration, then appropriately approved retrospective research and silent-mode evaluation.
- **Primary estimand:** Cancer **diagnosis recorded within a prespecified horizon** after an index encounter; this is **not** the biological onset of cancer, nor a patient's true stage progression.
- **Not in scope at launch:** Clinical recommendations, individual treatment decisions, automated test/referral scheduling, consumer reassurance, cancer incidence prediction for the entire public, or pediatric oncology deployment.
- **Separate study:** Pediatric headache-to-brain-tumour diagnostic research is methodologically related but is **not** a validation cohort or a claimed application of this product.
- **Safety:** Never use generated probabilities to defer an indicated test or referral; guideline-mandated pathways override optimization.

## Evidence and responsibility

Technical novelty is a **hypothesis, not yet a verified competitive advantage**. QCancer already offers cross-cancer risk scores (including competing-risk methodology); C the Signs provides cancer-pathway decision support; other products and research address longitudinal prediction. Our candidate differentiation is the **transparent joint evaluation of time-updated risk, test information value, capacity and downside risk**, with prospective clinical research required before utility claims.

References: [QCancer](https://www.qcancer.org/), [TrajOnco (2026)](https://doi.org/10.1038/s41746-026-03377-9), [TRIPOD+AI](https://doi.org/10.1136/bmj-2023-078378), [NICE NG12](https://www.nice.org.uk/guidance/ng12), [FDA CDS (2026)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software). See the full evidence register in the competitor document.

## Development principles

1. **Evidence before claims.** Transparent baselines, confidence intervals, external validation, decision-curve analysis.
2. **Human-in-the-loop.** Simulation does not replace qualified clinical judgement.
3. **No unsupported numbers.** Synthetic examples are prominently labelled and are not clinical estimates.
4. **Privacy by design.** No identifiable records committed to Git; data provenance, access controls and retention controls are required.
5. **Accessible UX.** Keyboard support, zoom, high contrast, text alternatives, distinguishable status text and non-colour-only signals.

## Contributing / legal

This repository is at the product-planning stage. No open-source license has been selected; **public visibility does not automatically grant reuse rights**. Validate the brand's potential conflict with the existing medical abbreviation `dPCR` (digital PCR) before commercial launch.

© 2026 HyunStudio. All rights reserved until a license is explicitly published.
