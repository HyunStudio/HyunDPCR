# HyunDPCR — Security, Privacy, Product & Regulatory Gates v1.0

**작성일:** 2026-11-13 · **계획 문서이며 보안심사, 임상 승인, 규제허가를 의미하지 않음**

## 1. Intended use and deployment restriction

- **MVP intended use:** Synthetic-data educational/research workflow demonstration of probabilistic methods, not health services.
- **MVP prohibited:** collection/storage of real medical files, live patient assessment, generated 'safe to wait' advice, automated test orders, physician workflow interventions, user-uploaded personal clinical data.
- **Research next phase:** institutionally approved data and DUA/IRB, specific cohort, minimization, site controls, method review.
- **Clinical next phase:** intended-use review, validation, human factors and patient-safety evaluation, jurisdiction-specific medical device/CDS legal assessment.

## 2. Threat model (STRIDE-inspired, high-level)

| Asset / boundary | Threat | Mitigation in design | Gate |
|---|---|---|---|
| Public GitHub | secrets/PHI accidentally committed | .gitignore, secret scanning, code review, synthetic fixture checks | block any release |
| Streamlit process | leaking state across users | synthetic-only UI, no PHI, no patient data in shared cache; test session behavior | block clinical use |
| Input layer | fabricated/invalid timestamps, wrong units | Pydantic schema, timezone consistency, fail-closed input | block estimate |
| Risk model | look-ahead leakage, missing-data bias | timestamp cutoff, masked outcome labels, independent validation | block clinical claim |
| Options engine | dangerous 'wait' ranking or unjustified costs | permitted-action gate, mandatory evidence parameters, no recommendation | block scenario |
| Report/export | synthetic label or limitations removed | centralized watermark/report schema, export tests | block release |
| Institution link (future) | cross-tenant exposure | SSO, RBAC, encrypted store, tenant audit, penetration test | not in MVP |
| Model updates (future) | silent drift or version substitution | version registry, data/model cards, audit, rollback | not in MVP |

## 3. Model card minimum

- Model name/version, training data status (synthetic vs authorized real data), clinical endpoint, target population, index/as-of, horizon.
- Intended and prohibited use, inputs, preprocessing, missingness/censoring, limitations.
- Training/evaluation data split and characteristics, subgroup metrics if available, calibration/uncertainty.
- Status: `TOY_ONLY` until appropriate real data, external validation and governance.
- Responsible owner, change log, rollback policy.

## 4. Release gates (no implied certification)

| Release | Required evidence | Can product be used for clinical care? |
|---|---|---|
| R0 Documentation | technical/clinical assumptions and safety boundaries identified | **No** |
| R1 Synthetic MVP | deterministic tests, leakage guards, watermarks, accessible UI checks | **No** |
| R2 Retrospective research | IRB/DUA/data governance + cohort + internal model evaluation | **No** |
| R3 Independent external & silent-mode pilot | institution approvals, external validation, usage and safety monitoring | **Not by default** |
| R4 Clinical deployment (optional) | applicable regulator, medical quality management, benefit/risk, legal/privacy, human oversight and monitoring | only within authorized intended use |

## 5. Country-specific diligence (not legal conclusions)

- **Korea:** MFDS medical device/SaMD applicable guidance and PIPA; institutional clinical approvals.
- **UK:** MHRA software/AI medical device framework, UK GDPR and applicable NHS guidance.
- **US:** FDA CDS function-level distinctions and, where applicable, HIPAA.
- Verify requirements based on exact clinical claims, user role, workflow impact, data transfers and jurisdiction. No generic assertion that 'research-only' branding exempts a clinically used application.

## 6. Security backlog sequence

1. Initial `.gitignore`, permissions and secret scanning, no file upload in public demo.
2. Model/data/artifact manifests; input validation and explicit logging without PHI.
3. Unit/integration/safety tests before public synthetic demo.
4. Before institutional research: security architecture and data retention schedules, access control, agreements, encryption, operational alerts, backup/restore.
5. Before clinical deployment: human factors assessment, quality system, incident response, post-market model monitoring as applicable.

## 7. Key regulatory and research references

- [FDA Clinical Decision Support Software (2026)](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software)
- [FDA GMLP](https://www.fda.gov/medical-devices/artificial-intelligence-enabled-medical-devices/good-machine-learning-practice-medical-device-development-guiding-principles)
- [NICE suspected cancer NG12](https://www.nice.org.uk/guidance/ng12)
- [TRIPOD+AI](https://doi.org/10.1136/bmj-2023-078378)
- [ISO 14971](https://www.iso.org/standard/72704.html)
