---
name: Close HCC / RAF risk-adjustment gaps with Diagnostic Robotics
description: Read a patient's coded and uncoded HCC conditions with suggested ICD codes and supporting reasons, then accept or reject each suggestion back into the platform.
api: openapi/diagnostic-robotics-precision-population-health-openapi.yml
operations:
  - get_patient_raf_gaps_data_by_patient_id_api_v2_RiskAdjustmentProfile__patientId__get
  - update_diagnoses_api_v2_RiskAdjustmentProfile__patientId__diagnoses_patch
generated: '2026-08-12'
method: generated
source: openapi/diagnostic-robotics-precision-population-health-openapi.yml
---

# Close risk-adjustment gaps

Prerequisite: a bearer token from `skills/diagnostic-robotics-authenticate.md`.

## 1. Read the gap profile — `get_patient_raf_gaps_data_by_patient_id_api_v2_RiskAdjustmentProfile__patientId__get`

```bash
curl -X GET ".../api/v2/RiskAdjustmentProfile/{patientId}" -H "Authorization: Bearer {token}"
```

Returns `PatientRiskAdjustmentProfileResponse`:

- `codedRiskScore` / `uncodedRiskScore` — RAF today versus RAF if every suggestion were confirmed.
- `codedConditions[]` (`PatientHccResponse`: `code`, `value`, `description`, `version`, `diagnoses[]`) — HCCs
  already supported by coded claims.
- `uncodedConditions[]` (`PatientHccRecommendationResponse`: `diagnoses[]` of
  `PatientIcdSuggestionResponse`, each with a `status` and a `reason[]`) — suspected HCCs with the evidence
  behind the suspicion.
- `meta` — provenance for the profile.

Always surface `reason[]` to the reviewing clinician or coder. A suggestion without its evidence is not
reviewable, and confirming an unsupported diagnosis is a compliance problem, not just a data problem.

## 2. Record the review — `update_diagnoses_api_v2_RiskAdjustmentProfile__patientId__diagnoses_patch`

```bash
curl -X PATCH ".../api/v2/RiskAdjustmentProfile/{patientId}/diagnoses" \
  -H "Authorization: Bearer {token}" -H "Content-Type: application/json" \
  -d '{"diagnoses":[{"...":"UpdateIcdPayload with a status"}]}'
```

Body is `UpdateDiagnosesPayload` — an array of `UpdateIcdPayload`, each carrying a `Status`.

## Rules

- **A human decides.** This operation writes a coding decision that flows into risk-adjustment submissions. Do
  not auto-accept suggestions; an agent may prepare the payload, a person confirms it.
- `PATCH` here is **not idempotent and carries no idempotency key**. On an ambiguous failure, re-read the
  profile and diff before resending.
- Only `200` and `422` are declared on this operation — the error surface is thinner than the read side.
- `codedRiskScore`, ICD codes and HCC descriptions are PHI plus regulated coding data.
