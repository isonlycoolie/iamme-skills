---
name: iamme
description: Integrate iamme, Tanzania's verified identity layer, into an app. Use when the user wants to verify customers or businesses with iamme, add "Connect with iamme", call the iamme API, read an attestation, or check an iamme receipt.
---

# iamme

iamme verifies a person or business once (staff review documents by hand) and gives each one a single number.
A verifier (your app's company) receives only the fields its approved grant allows, after the person consents,
with a signed receipt for every disclosure.

## Before you write code

1. The user needs an approved verifier in the iamme Dashboard with a product, an API key and a registered return URL.
   Keys and return URLs are under **Developers** in the Dashboard. If they have none, stop and tell them to create them.
2. Put the key in the server's environment as `IAMME_API_KEY`. Never ship it to a browser or mobile app.
3. Find or create the workflow (the list of fields to ask for) in the Dashboard and copy its ID.

## The integration in five steps

All calls go from the app's **server** to `https://api.iamme.tz/api/v1`.

1. **Start**: `POST /connect/requests` with the workflow ID, a purpose, the registered `return_url`, a random `state`
   and a PKCE S256 `code_challenge`. Keep `state` and the `code_verifier` in the user's session.
2. **Send** the person's browser to the `url` in the response. iamme asks them to consent to exactly that workflow's fields.
3. **Return**: iamme redirects to `return_url?code=...&state=...` (or `?error=...&state=...`). Check `state` first.
4. **Exchange**: `POST /connect/token` with `{ code, code_verifier }` within 60 seconds. You get `person` and/or `business`,
   each with a `consent_request` and a `subject` (your app's pseudonym for them).
5. **Read**: `GET /subjects/{subject}/attestation?consent_request={id}` returns the verified fields and a signed receipt.
   Store the receipt with the record.

Details: [references/connect.md](references/connect.md); a working Node example: [references/connect-example.md](references/connect-example.md).
Reading fields and the non-hosted flow: [references/data.md](references/data.md).
Verifying receipts: [references/receipts.md](references/receipts.md).
Errors: [references/errors.md](references/errors.md).

## Rules you must follow

- The API key stays server-side, read from `IAMME_API_KEY`. Never log it or commit it.
- Never log or print personal data: NIN, names, dates of birth, phone numbers, codes or tokens. Log IDs only.
- Always use PKCE (S256) and always check `state` on return.
- Send a fresh `Idempotency-Key` (a UUID) on every POST, and reuse it only to retry the same request.
- Ask only for fields the grant covers; others are refused (`grant-field-not-allowed`).
- Store the attestation's `fields` exactly as received, so the receipt's hash still matches.
- Do not invent endpoints. Webhooks are not available yet: poll the request's status instead (see connect.md).
- There is no separate test key or sandbox today; every key is live and tied to one product and one grant.
