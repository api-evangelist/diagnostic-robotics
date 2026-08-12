---
name: Authenticate against the Diagnostic Robotics Precision Population Health API
description: Exchange the customer client_id and client_secret for an OAuth 2.0 bearer token on the customer's own subdomain, and reuse it correctly.
api: openapi/diagnostic-robotics-precision-population-health-openapi.yml
operations:
  - login_for_access_token_api_oauth_token_post
generated: '2026-08-12'
method: generated
source: https://docs.diagnosticrobotics.com/docs/proactive-patient-risk-feed-api/3y8qknbsqo42r-authentication
---

# Authenticate

Every other Diagnostic Robotics skill starts here. There is no self-serve signup — the `client_id` and
`client_secret` are issued by Diagnostic Robotics during onboarding, together with the customer's own subdomain.

## 1. Resolve the host

The customer identifier is part of the hostname, not a header or a path:

```
https://{CLIENT}.precision-population-health.diagnosticrobotics.com
```

The published OpenAPI ships only `sandbox` as the `{CLIENT}` value. Never guess a customer subdomain — if you
were not told one, stop and ask.

## 2. Get a token — `login_for_access_token_api_oauth_token_post`

```bash
curl -X POST "https://{CLIENT}.precision-population-health.diagnosticrobotics.com/api/oauth/token" \
  -H "accept: application/json" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id={client_id}&client_secret={client_secret}"
```

The response is a `TokenResponse`. Present it on every subsequent call as `Authorization: Bearer {token}`.

## 3. Rules

- The spec declares two OAuth schemes. Use `OAuth2PasswordBearer` (`/api/oauth/token`). The
  `OAuth2AuthorizationCodeBearer` scheme points at the Auth0 tenant `digital-outreach.us.auth0.com`, which no
  longer resolves — that flow is dead.
- No scopes exist. The token carries the whole tenant grant, including read access to patient demographics and
  write access to diagnoses. Treat it as a high-consequence credential.
- A missing or expired token returns `401` with body `{"detail":"Not authenticated"}`. Re-run this skill and retry.
- Token lifetime is not published. Refresh on `401` rather than on a timer.
