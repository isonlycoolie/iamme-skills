# Hosted connect

The person consents on iamme's own page; your server makes two calls.

## Auth and headers

- `Authorization: Bearer <key_id>.<secret>` where `key_id` starts with `ak_`. One key belongs to one product and one grant.
  A revoked or expired key, or an inactive product or grant, returns `401 invalid-api-key`.
- `Idempotency-Key`: 8 to 64 characters, required on every POST. Same key and same body replays the first response
  (header `Idempotent-Replayed: true`); same key with a different body is `422 idempotency-key-reused`.
- `Content-Type: application/json`. Unknown body fields are rejected.

## 1. Pick the workflow (optional)

`GET /workflows` returns `{ workflows: [...] }`, templates first. `GET /workflows/{id}` returns one. Each workflow is a
named list of fields. Every field must be in the key's grant; business fields need a KYB grant.

## 2. Create the request

`POST /connect/requests`

| Field | Rule |
| --- | --- |
| `workflow` | Workflow ID (uuid) |
| `purpose` | 3 to 200 characters, shown to the person |
| `return_url` | Must exactly match a return URL registered on the key's product, else `400 return-url-not-registered` |
| `state` | 1 to 200 characters, random; keep it in the session |
| `code_challenge` | base64url SHA-256 of the verifier, 43 characters |
| `code_challenge_method` | `S256` only |

Response `201 { id, url, expires_at }`. The request lives 15 minutes. Send the person's browser to `url`.

## 3. The person comes back

iamme redirects to `return_url` and always includes `state`:

- approved: `?code=<one-time code>&state=...`
- otherwise: `?error=access_denied|needs_upgrade|expired&state=...`

Reject the return if `state` does not match the session. `needs_upgrade` means the person must raise their
verification level in iamme first; show a friendly message and let them retry later.

## 4. Exchange the code

`POST /connect/token` with `{ code, code_verifier }` (verifier 43 to 128 characters). The code lives 60 seconds and works
once; a second use withdraws the consent it produced. A bad or used code is `400 invalid-grant`.

Response `200 { person?, business? }`, each `{ consent_request, subject, authorised_by? }`. `subject` is your app's
pseudonym for them: store it as their iamme ID. `authorised_by` (`{ name, role: "authorised_person" }`) appears on a
business when a person confirmed it on the business's behalf.

Then read each one's attestation: see [data.md](data.md).

## Status instead of webhooks

Webhooks are not available yet. If you need to know where a request stands, poll
`GET /connect/requests/{id}`: `{ id, status: open|approved|denied|needs_upgrade|expired, expires_at }`.

A working Node example of both calls: [connect-example.md](connect-example.md).
