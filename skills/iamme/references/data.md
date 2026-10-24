# Reading verified data

## Attestation

`GET /subjects/{subject}/attestation?consent_request={id}`. `consent_request` is required, and the consent must be
approved.

```json
{
  "subject": "test-subject-0001",
  "level": 1,
  "method": "agent_reviewed",
  "verified_at": "2026-01-01T10:00:00Z",
  "fields": { "full_name": "Test Person", "date_of_birth": "1990-01-01" },
  "missing": [],
  "receipt": "<compact JWS>",
  "receipt_id": "<uuid>",
  "authorised_by": { "name": "...", "role": "authorised_person" }
}
```

- What comes back is the grant, intersected with the person's consent, intersected with what iamme has verified.
  A field outside the grant is absent, not null. A field the profile lacks is listed in `missing`.
- `level` is the person's verification level; `method` is how the fields were checked (`agent_reviewed`: iamme staff).
- Asking again for the same consent returns the same disclosure and receipt.
- Keep `fields` exactly as received (the raw JSON bytes) next to `receipt`: the receipt signs their hash.

## Asking a known person directly (no hosted page)

If you already have the person's iamme number:

- `POST /consent-requests` with `{ number, fields: [...], purpose, channel?: "remote" | "in_person" }`. The fields must be
  within the grant. It returns `201 { id, status: "pending", expires_at }`, even for an unknown number, so it never
  reveals who is registered.
- The person approves in iamme. Poll `GET /consent-requests/{id}`: `status` is
  `pending|approved|denied|expired|used`, plus `subject` once approved.
- Then read the attestation with that `subject` and consent request ID.

## Your grant

`GET /grant` describes what your key may ask for: `{ id, org_id, org_name, tier, compliance_type, purpose, legal_basis,
fields, consent_mode: per_use|standing, retention_months, expires_at }`. Use `fields` to know what you can request and
`retention_months` for how long you may keep the data.
