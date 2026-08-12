---
name: Work a Diagnostic Robotics actionable risk list
description: Find the right risk list for a population and product, page through the ranked members, open a member's risk profile, and record or dismiss a recommended next step.
api: openapi/diagnostic-robotics-precision-population-health-openapi.yml
operations:
  - search_risk_lists_api_v2_RiskList_get
  - get_risk_list_api_v2_RiskList__id__get
  - export_risk_list_api_v2_RiskList__id___export_get
  - get_risk_profile_api_v2_RiskProfile__patientId__get
  - update_next_step_api_v2_RiskProfile__patientId__NextStep__nextStepId___update_post
  - ignore_next_step_api_v2_RiskProfile__patientId__NextStep__nextStepId___ignore_post
generated: '2026-08-12'
method: generated
source: https://docs.diagnosticrobotics.com/docs/proactive-patient-risk-feed-api/ldp3dalnzn9c1-risk-list
---

# Work a risk list

Prerequisite: a bearer token from `skills/diagnostic-robotics-authenticate.md`.

## 1. Find the list — `search_risk_lists_api_v2_RiskList_get`

```bash
curl -X GET "https://{CLIENT}.precision-population-health.diagnosticrobotics.com/api/v2/RiskList?population=ACO&product=CHF&product=ALL_INPATIENT" \
  -H "Authorization: Bearer {token}"
```

`product` and `population` are repeatable query parameters. The response is an array of `RiskListMetadata`
(`id`, `url`, `product`, `population`, `serveDate`, `updateDate`). Pick by `product` + `population`, then by the
most recent `serveDate` — a stale `serveDate` means the underlying claims feed has not refreshed.

## 2. Read the ranked members — `get_risk_list_api_v2_RiskList__id__get`

```bash
curl -X GET ".../api/v2/RiskList/{id}?offset=0&limit=30" -H "Authorization: Bearer {token}"
```

Returns `{metadata, total, offset, entities[]}`. Each entry carries `patientId`, a `patient` object
(`firstName`, `lastName`, `dob`, `state` — PHI), a `riskScore`, and link fields `riskProfileUrl` and
`patientProfileUrl`. Follow those links rather than composing paths.

`offset` and `limit` work but are **not declared in the OpenAPI**, so a generated client will not expose them —
add them by hand. Page until `offset + len(entities) >= total`.

For a bulk pull use `export_risk_list_api_v2_RiskList__id___export_get`
(`GET /api/v2/RiskList/{id}/$export`), which returns a file rather than JSON.

## 3. Open one member — `get_risk_profile_api_v2_RiskProfile__patientId__get`

Returns `{patientId, updateDate, patient, riskScore, drivers[], nextSteps[]}`. `drivers[]` explains *why* the
member scored; `nextSteps[]` is the recommended intervention set, each with `id`, `displayText`,
`contributingDrivers`, `reference`, `owner` and `status`.

## 4. Close the loop

- Advance a step: `POST /api/v2/RiskProfile/{patientId}/NextStep/{nextStepId}/$update` with a
  `TaskUpdatePayload` (`status`). Returns `204`.
- Dismiss a step: `POST /api/v2/RiskProfile/{patientId}/NextStep/{nextStepId}/$ignore` with a
  `TaskIgnorePayload`. Returns `204`.

## Rules

- **These two writes are not idempotent and are not documented as retry-safe.** There is no `Idempotency-Key`.
  On a network timeout, re-read the risk profile and check the step's `status` before retrying.
- Errors: `400` bad request, `401` expired token, `404` unknown list/patient/step, `422` validation failure with
  a `detail[].loc` pointing at the offending field. No `problem+json`, no `429`, no `Retry-After`.
- Everything returned here is PHI. Do not log payloads, and do not echo patient names or dates of birth into a
  transcript that leaves the customer's control.
