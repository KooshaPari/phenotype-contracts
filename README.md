# phenotype-contracts

Canonical, language-agnostic **contracts** (JSON Schemas + policy specs) shared across the Phenotype provider/LLM-proxy ecosystem. This repo is the neutral source of truth that consumer repos pin to and align with — no single consumer owns it.

## Why this exists

forgecode (Rust), OmniRoute (TypeScript), and cliproxyapi-plusplus (Go) each independently implement provider/model handling, OAuth refresh, retry/resilience, and SSE stream termination. The bulk of that code is per-language runtime/HTTP/storage glue and is **not** liftable into a single shared library across three languages. What *is* genuinely shared is the **contract**: the data shapes, the policy parameters, and the classification rules. Those live here, and each repo conforms to them in its own language.

## Contracts

`provider-models/`
- **`provider-model.schema.json`** — provider/model registry + normalization: model id, provider, capabilities, context window, etc.
- **`oauth-refresh-policy.schema.json`** — parameterized OAuth refresh-lead + expiry policy. Refresh-lead is a **parameter** (forgecode 5 min, cliproxy per-provider, some providers 24 h) — never a hardcoded constant. Predicate: `now + lead >= token.expires_at`.
- **`resilience-policy.schema.json`** — retry params (max_attempts, backoff_factor, min/max delay, jitter), retryable-error classification (HTTP status set + transport error kinds), and the SSE terminal-marker rule set.

## Consumers (conformance)

| Repo | Lang | Conforms via |
|------|------|--------------|
| forgecode | Rust | reference implementation; vendored pin + pointer here |
| OmniRoute | TypeScript | spec-aligned (values validated against these schemas) |
| cliproxyapi-plusplus | Go | spec-aligned (values validated against these schemas) |

Consumers SHOULD pin a version (git tag/sha) of this repo and add a conformance test asserting their constants (refresh-lead default, retryable status set, SSE terminal markers) match the schema.

## Versioning

Schemas are versioned via `$id`/`version` fields. Breaking changes bump the major; consumers pin and migrate deliberately.
