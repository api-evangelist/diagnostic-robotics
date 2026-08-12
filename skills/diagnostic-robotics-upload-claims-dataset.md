---
name: Upload a claims or EHR dataset to Diagnostic Robotics
description: Push FHIR R4 US Core ndjson, CMS CCLF, CSV or Parquet claims and EHR data into the clinical engine so risk recommendations stay fresh.
api: openapi/diagnostic-robotics-precision-population-health-openapi.yml
operations:
  - login_for_access_token_api_oauth_token_post
  - upload_api_v1_dataset__dataset_type__upload_post
generated: '2026-08-12'
method: generated
source: https://docs.diagnosticrobotics.com/docs/proactive-patient-risk-feed-api/z7azwtx4q4s33-fhir-cclf-file-data-upload-api
---

# Upload a dataset

Prerequisite: a bearer token from `skills/diagnostic-robotics-authenticate.md`.

## 1. Choose the format

| Data | Accepted formats |
|---|---|
| Claims | FHIR R4 US Core STU3 ndjson (`Patient`, `Coverage`, `ExplanationOfBenefit`, `Claim`, `ClaimResponse`), CMS CCLF |
| EHR | FHIR R4 US Core STU3 ndjson |
| Custom / care events | CSV, Parquet, JSON per the published data dictionary |

## 2. Upload — `upload_api_v1_dataset__dataset_type__upload_post`

Note this endpoint is still on `/api/v1` while every resource path is `/api/v2`.

```bash
curl -X POST "https://{CLIENT}.precision-population-health.diagnosticrobotics.com/api/v1/dataset/claims/upload?updated_for=2020-01-01&format=ndjson" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/x-gzip" \
  --data-binary "@/path/yourdataset.ndjson.gz"
```

For CCLF, set `format=cclf` and post the gzipped CCLF bundle.

- `updated_for` is the snapshot date the file represents — required, and wrong values silently misdate the feed.
- `format` must match the body.

## 3. Rules

- **Maximum 200MB per request.** Larger datasets must be split into chunks; gzip before upload; contact support
  for a bulk path.
- Claims should be refreshed **at least monthly** — a stale feed shows up downstream as a stale `serveDate` on
  every risk list.
- The only declared failure response is `422 HTTPValidationError`. There is no upload status resource, no job id
  and no webhook: after upload, confirm ingestion by checking that `serveDate`/`updateDate` on the relevant risk
  list has advanced.
- Not idempotent. Re-posting the same file for the same `updated_for` has undefined semantics — ask the account
  team before a retry storm.
- This is bulk PHI in transit. Use only the customer's own subdomain and never a shared or logged path.
