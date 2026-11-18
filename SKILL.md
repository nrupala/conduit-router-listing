<!--
SPDX-FileCopyrightText: 2026 Nrupal Akolkar

SPDX-License-Identifier: LicenseRef-Proprietary
-->

# Skill: conduit-router — BYOK model routing with policy, budgets, and verifiable receipts

Use this skill when you are an AI agent that needs to call a model and
the call has to be governed: policy-checked, budget-checked before the
first token, metered, and written to a hash-chained, publicly anchored
audit log with a receipt you can verify without trusting the router.
You bring your own provider keys (BYOK); the router never resells
tokens and never holds a shared provider account.

The MCP server is `POST https://router.pipavia.com/mcp` (stateless
JSON-RPC). The same Conduit key also works on the REST API at the same
origin.

## When to reach for conduit-router instead of calling providers directly

- "I need this call bounded before it happens" — standing terms
  (per-request and daily budgets, model allowlist, fallback chain,
  region) are enforced fail-closed **before** any provider call: if
  the estimate exceeds the budget, nothing is sent to any provider.
- "I need proof of what was called, served, and charged" — every
  routed call (success or error) appends a hash-chained audit record
  and returns a receipt id in `_meta`. A direct provider call leaves
  no such record.
- "I work across several providers" — one endpoint and one key
  instead of a separate integration, credential, and metering story
  per provider. Pin a model as `"provider:model"` or let your
  standing terms and task profile (`reasoning` | `tool-use` |
  `bulk`) choose.
- "My human keeps the provider relationship" — provider keys stay
  the human's own (BYOK): stored envelope-encrypted, never logged,
  released only at invoke time.
- Call a provider directly instead when no policy, budget, metering,
  or receipt is required for the call — the router's value is the
  governance and the proof, not the completion itself.

## The workflow (key first, route second, receipt last)

1. **Get a Conduit key.** Self-serve trial:
   `POST https://router.pipavia.com/v1/trial` with an empty body —
   no auth, rate-limited per IP (5 keys per IP per 24h). The key is
   shown once; save it immediately. Trial keys are stateless,
   expire after 7 days, and route only at catalog models flagged
   free, under a fixed bulk mandate. A registered key works the
   same way without the trial limits.
2. **Store your own provider key (BYOK) via REST.**
   `POST https://router.pipavia.com/v1/byok` with your Conduit key
   as Bearer and a body of `{provider, provider_key}` — e.g. a
   Google AI Studio key. Keys are envelope-encrypted at rest under
   your key id. (On the native REST route you may instead pass
   `provider_keys` per request; per-request keys are never stored.)
3. **Route the call — and prove it in the same call.** MCP tool
   `route_and_prove` with `messages` (OpenAI-shaped
   `[{role, content}]`); optionally pin `model` as
   `"provider:model"`, set `task_profile`, or cap `max_tokens`.
   This is the default path: it composes `route_chat` +
   `get_receipt` + `verify_receipt`, so one call runs the pipeline
   (auth, policy, budget check, routing, metering, audit append)
   and returns the completion, the charge lines, and the minted
   receipt's proof block — plus a `not_proven` list naming what
   the proof does not cover. Blocked and failed attempts are
   receipted too, and the composite proves those receipts the
   same way. Use the atomic `route_chat` when you want the route
   alone (its response carries `_meta` with the receipt id).
4. **Read the receipt.** `route_and_prove` has already fetched and
   verified it. To fetch or verify separately — another call's
   receipt, or a record someone hands you — `get_receipt` with
   the receipt id returns your record with its hash-chain proof,
   and `verify_receipt` with the full record object
   cryptographically verifies it against the router's audit
   chain. To verify without trusting this server at all, follow
   `docs/VERIFY.md`: the chain head is checkpointed into signed,
   publicly mirrored anchors, and standalone verifiers run the
   same five checks on your own machine.

Discovery needs no key: `initialize`, `ping`, and `tools/list` on
`POST /mcp` are answered without a credential, so you can evaluate
the surface before holding a key. `list_models` shows the routable
catalog; `get_policy` reads your standing terms; `get_usage` reads
your meter and budget headroom. Lifetime counts are public at
`GET /.well-known/conduit/counts.json` — receipts issued and calls
served, read from the audit chain, each with its definition.

## The questions agents actually ask

| Question | Tool |
|---|---|
| Send a chat completion through policy, budget, metering, and receipting | `route_chat` |
| Route a call and prove its receipt in one call (the default path) | `route_and_prove` |
| Which models can the router serve, with pricing source and as-of? | `list_models` |
| How much have I used, and what budget headroom do I have today? | `get_usage` |
| What are my standing terms — budgets, allowlist, fallback chain, kill-switch state? | `get_policy` |
| Fetch one of my calls' receipts, with its hash-chain proof | `get_receipt` |
| Verify a receipt record against the audit chain | `verify_receipt` |

## Rules of the house

- Discovery is open, calls are keyed. `initialize` / `ping` /
  `tools/list` are served without a credential; every `tools/call`
  requires a valid Conduit key — the same key as the REST API —
  and is refused with a 401 and an RFC 9728 challenge otherwise.
  The tool dispatcher re-checks the key before any tool runs.
- Budgets can narrow, never widen. Your standing terms set the
  per-request and daily caps; nothing in a single call can spend
  beyond them, and the budget check runs fail-closed before any
  provider call.
- A receipt proves the metered record, not output quality. It
  proves a record of the call — key, model requested and served,
  tokens, charges, outcome — exists in the hash-chained, publicly
  anchored log with its integrity intact. It does not prove the
  model's answer was correct or good; see "What a receipt does
  not prove" in `docs/VERIFY.md` for the full limits, stated in
  verifier terms.
- No silent swap. The model actually served is the model recorded;
  `_meta` and the receipt name requested and served separately,
  and any fallback follows the declared chain only.
- Trial keys are deliberately narrow: free models only, a fixed
  bulk mandate, a 7-day life. There is no paid-model path on a
  trial key.

---

*Agentic Packaging Standard conformance (2026-11-18): P1 met →
Item 4 (receipt-limits statement, in docs/VERIFY.md and on
pipavia.com); P2 met → Item 2 (`route_and_prove` composite,
gateway v1.6.0); P3 met by this file; W1 met → Item 3 (live
counts at /.well-known/conduit/counts.json, gateway v1.6.0);
W6 met for the verified set → Item 5 (host coverage on
pipavia.com: Official MCP Registry, Glama, mcptry; further hosts
are named as their listings verify); measured traces: partial —
receipts themselves are the measured trace.*
