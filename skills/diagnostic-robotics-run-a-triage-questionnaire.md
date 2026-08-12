---
name: Run a Diagnostic Robotics patient triage questionnaire
description: Drive the adaptive symptom questionnaire end to end — create a patient, open a visit, attach symptoms, answer questions until exhausted, finish the visit and read the triage outcome.
api: openapi/diagnostic-robotics-patient-questionnaire-openapi.yml
operations:
  - post_patient_list_resource
  - post_create_visit_to_patient_resource
  - get_symptom_search_autocomplete_resource
  - post_symptoms_resource
  - get_questions_first_unanswered_resource
  - post_question_response_resource
  - post_finished_resource
  - get_visit_outcomes_resource
  - get_medical_summary_resource
generated: '2026-08-12'
method: generated
source: https://docs.diagnosticrobotics.com/docs/patient-questionnaire-api/6530c434c2f93-diagnostic-robotics-api
---

# Run a triage questionnaire

Auth for this API is **not** OAuth. Send the `x-client` API key header issued by Diagnostic Robotics on every
request. The published host is the placeholder `env-name.diagnosticrobotics.com` — use the environment host you
were given. Questionnaire paths sit under `/api/pq`, symptom search under `/api/search`.

## 1. Create the patient — `post_patient_list_resource`

`POST /v2/patients/` with a `CreatePatientRequest`. Returns `201` and a `CreatePatientResponse`.

## 2. Open a visit — `post_create_visit_to_patient_resource`

`POST /v2/patients/{patient_id}/visits`. Returns `201` and a `CreateVisitResponse` carrying the `visit_id` that
every remaining step needs. (`get_create_visit_to_patient_resource` lists a patient's visits; `404` means the
patient id is wrong.)

## 3. Resolve and attach symptoms

- `get_symptom_search_autocomplete_resource` — `GET /symptoms/autocomplete` on the search service turns free
  text into `Symptom` records. Never invent a symptom id.
- `post_symptoms_resource` — `POST /v2/visits/{visit_id}/symptoms/{symptom_id}` attaches one. Returns `201`.
- `get_symptom_search_resource` — `GET /visits/{visit_id}/symptoms` lists what is attached.

## 4. Loop the question tree

```
while true:
  q = get_questions_first_unanswered_resource(visit_id)   # GET /v2/visits/{visit_id}/questions/first_unanswered
  if no question remains: break
  post_question_response_resource(visit_id, q.seed, answer) # POST /v2/visits/{visit_id}/questions/{seed}/respond
```

`post_question_response_resource` returns `201` on success and **`400 Invalid seed`** when the seed is not the
currently open question — that is the signal to re-read `first_unanswered` rather than to retry the same seed.
`get_question_by_seed_resource` re-reads a specific question.

## 5. Finish and read the result

- `post_finished_resource` — `POST /v2/visits/{visit_id}/finish`.
- `get_visit_outcomes_resource` — `GET /v2/visits/{visit_id}/outcomes` returns the `OutcomeModel` triage result.
- `get_medical_summary_resource` — `GET /v2/visits/{visit_id}/medical_summary` returns the clinical summary.

Both return `404` if the visit id is unknown.

## Rules

- **This is a triage aid, not a diagnosis.** Present the outcome as a recommendation for a clinician, never as
  a clinical decision, and never suppress a higher-acuity outcome.
- No idempotency key exists. `post_symptoms_resource` and `post_question_response_resource` on a retry may
  duplicate or collide — re-read state instead of blind retrying.
- Most operations in this spec declare **no error response schema at all**, so failure bodies are unspecified.
  Branch on status code, not on body shape.
