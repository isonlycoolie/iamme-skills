# iamme agent skills

Teach your coding agent to integrate iamme, Tanzania's verified identity layer: hosted connect,
attestations and signed receipts. The agent learns the API and the safety rules; it never gets access to your iamme
account.

## Install

Any agent (Claude Code, Cursor, Codex, Copilot, Windsurf and others):

```sh
npx skills add isonlycoolie/iamme-skills
```

Add `-g` to install for every project.

Claude Code plugin:

```text
/plugin marketplace add isonlycoolie/iamme-skills
/plugin install iamme@iamme
```

## Then

Ask your agent, for example: "Add Connect with iamme to sign-up". It will need, from the iamme Dashboard:

- an API key (Developers > API keys), kept in `IAMME_API_KEY` on your server;
- a registered return URL (Developers > Return URLs);
- a workflow ID (Workflows).

## What is inside

- `skills/iamme/SKILL.md`: when to use it, the five-step integration and the rules.
- `skills/iamme/references/`: connect (with a Node example), data, receipts and errors.
