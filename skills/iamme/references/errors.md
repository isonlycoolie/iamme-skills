# Errors

Errors are RFC 9457 `application/problem+json`. Branch on the slug at the end of `type`
(`https://docs.iamme.tz/errors/<slug>`), never on `title` or `detail`.

| Status | Slug | What to do |
| --- | --- | --- |
| 401 | `invalid-api-key` | Key wrong, revoked or expired, or its product or grant is not active. Check the Dashboard |
| 400 | `idempotency-key-required` | Send an `Idempotency-Key` on the POST |
| 422 | `idempotency-key-reused` | Same key with a different body: use a new key for a new request |
| 409 | `idempotency-in-progress` | The first call with this key is still running: wait and retry |
| 400 | `invalid-json` | Body is not valid JSON |
| 415 | `json-required` | Send `Content-Type: application/json` |
| 422 | `validation` | A field breaks the rules (length, format, unknown field) |
| 400 | `return-url-not-registered` | Register the exact `return_url` on the product in the Dashboard |
| 400 | `invalid-grant` | Connect code wrong, expired (60 s) or used: start a new request |
| 422 | `grant-field-not-allowed` | The workflow or request asks for a field outside your grant |
| 422 | `grant-not-kyb` | Business fields need a KYB grant |
| 404 | `connect-request-not-found`, `consent-request-not-found`, `workflow-not-found`, `receipt-not-found` | Wrong ID, or not yours |

Retry only on network errors, 409 and 5xx, with the same `Idempotency-Key`.
