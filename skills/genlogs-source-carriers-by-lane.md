---
name: Source and verify carriers on a lane
description: Authenticate, get carrier recommendations for an origin/destination lane, verify a carrier was recently sighted near the lane, and pull its profile.
api: openapi/genlogs-openapi.yml
operations: [createAccessToken, getCarrierRecommendations, verifyCarrierSighting, getCarrierProfileDetail]
---

# Source and verify carriers on a lane

Use the GenLogs Truck Intelligence API to find carriers operating a lane and confirm real-world capacity from roadside sensor sightings.

## Auth (required on every call)
1. Mint an access token: `POST /auth/token` (`createAccessToken`) with `x-api-key` header and a JSON body `{"email","password"}`. Read `access_token_data.token`.
2. Send BOTH headers on every subsequent request: `x-api-key: <key>` and `Access-Token: <token>`. Tokens expire — re-mint on `401`.

## Steps
1. **Recommend carriers** — `GET /carrier/recommendations` (`getCarrierRecommendations`) with `origin_city`/`origin_state` and optional destination. Requires the `external-api-carrier-recommendation` permission. Each result carries a composite `carrier_score` (lane + equipment + company factors).
2. **Verify a sighting** — `GET /visual_sightings/verify` (`verifyCarrierSighting`) with `usdot`, `origin_city`, `origin_state`. Returns `{"verified": true|false}` — true means the carrier was seen within 150 miles of origin/destination/lane in the last 90 days. Requires `verifier-carrier`. Reward a `true`; do not penalize a `false`.
3. **Pull the profile** — `GET /carrier/profile` (`getCarrierProfileDetail`) for compliance/safety detail on the chosen USDOT. Requires `external-api-carrier-profile`.

## Rules
- `403` means your key lacks the endpoint's named permission — request it from GenLogs.
- Pagination (where present) is cursor-based via the `Link` header; follow `rel="next"`, never build cursors. See `conventions/genlogs-conventions.yml`.
- Errors return `{message,code}` / `{detail}` envelopes (not RFC 9457). See `errors/genlogs-problem-types.yml`.
