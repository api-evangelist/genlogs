---
name: Configure and test alert webhooks
description: Authenticate, register an HTTPS webhook endpoint, test it, and list active webhooks that receive alert.matches_found notifications.
api: openapi/genlogs-openapi.yml
operations: [createAccessToken, create_webhook_alerts_webhook_post, validate_webhook_endpoint_alerts_webhook_test_post, list_webhooks_alerts_webhook_get, deleteAlertWebhook]
---

# Configure and test alert webhooks

GenLogs pushes `alert.matches_found` notifications to customer webhooks when detection alerts match roadside sightings.

## Auth
1. `POST /auth/token` (`createAccessToken`) → `access_token_data.token`.
2. Send `x-api-key` + `Access-Token` on every call.

## Steps
1. **Register** — `POST /alerts/webhook` (`create_webhook_alerts_webhook_post`) with `{webhook_url, secret, description}`. The URL must be HTTPS. Requires `create-alert-webhook-endpoint` or `admin`.
2. **Test before enabling** — `POST /alerts/webhook/test` (`validate_webhook_endpoint_alerts_webhook_test_post`) to send a synthetic payload and confirm reachability. Requires `test-alert-webhook-endpoint` or `admin`.
3. **List** — `GET /alerts/webhook` (`list_webhooks_alerts_webhook_get`) to view configured endpoints. Requires `external-api-get-webhook-list`.
4. **Remove** — `DELETE /alerts/webhook/{webhook_id}` (`deleteAlertWebhook`). Requires `external-api-delete-alert-webhook-endpoint`.

## Verifying payloads
- Each payload is signed **HMAC-SHA512** with your per-webhook `secret`; recompute over the raw body and compare before trusting it.
- Payloads are company-wide and consolidated (all matches across all alerts per cycle). Event schema: `json-schema/genlogs-alert-matches-found.schema.json`; channel: `asyncapi/genlogs-alerts-asyncapi.yml`.
- Truck image URLs (front/side/rear) expire one month after delivery — fetch them promptly.
