# Hosted connect: Node example

Both calls run on your server only. Replace `<workflow id>` and the return URL with your own.

```js
import { createHash, randomBytes, randomUUID } from "node:crypto";

// PKCE (S256) and state: keep both in the user's session until they come back.
const verifier = randomBytes(32).toString("base64url");
const challenge = createHash("sha256").update(verifier).digest("base64url");
const state = randomBytes(16).toString("base64url");

const res = await fetch("https://api.iamme.tz/api/v1/connect/requests", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.IAMME_API_KEY}`,
    "Idempotency-Key": randomUUID(),
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    workflow: "<workflow id>",
    purpose: "Customer onboarding",
    return_url: "https://your-app.example/iamme/return",
    state,
    code_challenge: challenge,
    code_challenge_method: "S256",
  }),
});

const { url } = await res.json();
// Send the person's browser to url; iamme asks for exactly this workflow's fields.
```

```js
import { randomUUID } from "node:crypto";

// iamme sends the person back to return_url?code=...&state=...
if (query.state !== session.state) throw new Error("state does not match");

const res = await fetch("https://api.iamme.tz/api/v1/connect/token", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.IAMME_API_KEY}`,
    "Idempotency-Key": randomUUID(),
    "Content-Type": "application/json",
  },
  // The code lives 60 seconds and works once.
  body: JSON.stringify({ code: query.code, code_verifier: session.verifier }),
});

const { person, business } = await res.json();
// Each holds the consent request and pseudonym to fetch its attestation with.
```
