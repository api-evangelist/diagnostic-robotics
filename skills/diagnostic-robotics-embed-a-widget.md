---
name: Embed a Diagnostic Robotics population-health widget
description: Exchange a bearer token for a short-lived signed widget URL and embed the risk list, risk profile, patient profile or risk-adjustment view in a host application.
api: openapi/diagnostic-robotics-precision-population-health-openapi.yml
operations:
  - get_risk_list_widget_api_v2_RiskList__id___widget_get
  - get_risk_profile_widget_api_v2_RiskProfile__patientId___widget_get
  - get_patient_profile_widget_api_v2_PatientProfile__patientId___widget_get
  - get_patient_risk_adjustment_profile_widget_api_v2_RiskAdjustmentProfile__patientId___widget_get
generated: '2026-08-12'
method: generated
source: https://docs.diagnosticrobotics.com/docs/proactive-patient-risk-feed-api/aroq4qukz0mhm-widget-integration
---

# Embed a widget

Prerequisite: a bearer token from `skills/diagnostic-robotics-authenticate.md`.

## 1. Ask for the widget URL

```bash
curl -i -X GET ".../api/v2/RiskList/{id}/_widget" -H "Authorization: Bearer {token}"
```

Pick the operation that matches the view:

| View | Operation |
|---|---|
| Ranked population risk list | `get_risk_list_widget_api_v2_RiskList__id___widget_get` |
| One patient's risk profile | `get_risk_profile_widget_api_v2_RiskProfile__patientId___widget_get` |
| One patient's profile | `get_patient_profile_widget_api_v2_PatientProfile__patientId___widget_get` |
| One patient's HCC/RAF gaps | `get_patient_risk_adjustment_profile_widget_api_v2_RiskAdjustmentProfile__patientId___widget_get` |

The response is **`204 No Content`** with the signed URL in the `Location` header. There is no body — a client
that only reads response bodies will see nothing and conclude the call failed.

## 2. Embed it

```html
<iframe src="{url}" sandbox="allow-scripts" referrerpolicy="origin"/>
```

Native apps use the equivalent WebFrame.

## Rules

- The URL **embeds a single-use authentication token that expires in minutes**. Fetch it at render time. Never
  persist it, never put it in a shareable link, never log it, never email it.
- Keep `sandbox="allow-scripts"` and `referrerpolicy="origin"` — that is the provider's documented minimum.
- Declared failures: `401` unauthorized, `404` unknown id, `422` validation error.
- The older Widget Integration guide shows `/api/v1/RiskProfile/[ID]/_widget`; the published spec is `/api/v2`.
  Follow the spec.
