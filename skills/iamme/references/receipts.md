# Receipts

Every disclosure comes with a receipt: a compact JWS that proves who received which fields, when, and on what basis.

- Algorithm `EdDSA` (Ed25519), header `typ: JWT` and a `kid`.
- Public keys: `GET https://api.iamme.tz/.well-known/jwks.json` (no auth, cache for 5 minutes). Pick the key whose
  `kid` matches the receipt header.
- Claims: `iss, jti, iat, org, sub, grant, tier, purpose, basis, fields, payload_sha256, level, method, verified_at, consent`.
- `GET /receipts/{id}` returns `{ id, receipt }` again later (`404 receipt-not-found` otherwise).

## How to verify

1. Verify the JWS signature with the JWKS key for its `kid` (any JOSE library, for example `jose` in Node).
2. Check `payload_sha256` is the hex SHA-256 of the attestation's `fields` JSON exactly as you received it.
3. Check `sub` is the subject and `fields` lists the fields you hold.

```js
import { createRemoteJWKSet, jwtVerify } from "jose";
import { createHash } from "node:crypto";

const jwks = createRemoteJWKSet(new URL("https://api.iamme.tz/.well-known/jwks.json"));
const { payload } = await jwtVerify(receipt, jwks, { algorithms: ["EdDSA"] });
const hash = createHash("sha256").update(rawFieldsJson).digest("hex");
if (payload.payload_sha256 !== hash) throw new Error("receipt does not match the stored fields");
```

`rawFieldsJson` is the `fields` object's bytes as they came in the response. Re-serialising may change key order or
spacing and break the hash, so store the original text.
