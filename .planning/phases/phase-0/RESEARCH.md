# Phase 0: Planning / Protocol Spike - Research

**Researched:** 2026-05-29
**Domain:** Anthropic-compatible gateway protocol spike, SSE orchestration, provider adapter boundary
**Confidence:** MEDIUM

## Summary

Phase 0 should stay a design-and-fixtures spike, not a production implementation. The project docs already lock the core scope: v1 is an advisor-orchestrating local gateway with an Anthropic Messages-compatible `POST /v1/messages` ingress, mandatory support for both non-streaming and SSE streaming, and a gateway-owned advisor orchestration path that must preserve consumable event ordering. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md] [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md] [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]

The main planning value of Phase 0 is to reduce protocol ambiguity before code exists. The spike should produce fixtures and interface drafts that lock the minimum contract boundary, the canonical transcript shape, the provider adapter seam, and the event-ordering rules around pause/hold/resume. FastAPI and LiteLLM are credible candidates for the spike because FastAPI officially exposes `StreamingResponse`, Starlette officially documents `EventSourceResponse` as a third-party response class for Server-Sent Events, and LiteLLM officially documents streaming support plus usage/cost-oriented proxy features; however, the stack is not yet locked and Anthropic contract details still have an official-documentation verification gap in this environment. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/] [CITED: https://www.starlette.io/responses/] [CITED: https://docs.litellm.ai/docs/completion/stream] [CITED: https://docs.litellm.ai/docs/proxy/virtual_keys] [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]

**Primary recommendation:** Use Phase 0 to define golden protocol fixtures and a gateway-owned streaming state machine contract first; treat FastAPI and LiteLLM as spike candidates only, with explicit verification commands left in the plan before any production lock-in. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md] [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]

<phase_requirements>

## Phase Requirements

| ID  | Description                           | Research Support                                                                                                                                                                            |
| --- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| R1  | Anthropic Messages-compatible ingress | Defines minimum request/response fixture boundary and docs verification gap strategy. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]         |
| R3  | SSE streaming path                    | Defines SSE event model, ordering invariants, and spike verification commands. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]                     |
| R9  | SSE event order correctness           | Recommends explicit pre/pause/advisor/resume/post fixture families and anti-reordering rules. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md] |
| R10 | Canonical transcript model            | Defines minimum transcript fields and normalization responsibilities. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]                              |
| R13 | Provider adapter boundary             | Recommends a transport-agnostic adapter interface owned below ingress/orchestrator. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]           |
| R14 | Adapter capability set                | Maps the required adapter methods and normalized outputs. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]                                          |
| R22 | Golden contract tests                 | Recommends acceptance fixtures as the primary P0 artifact and planning baseline. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]                   |

</phase_requirements>

## Architectural Responsibility Map

| Capability                                       | Primary Tier  | Secondary Tier          | Rationale                                                                                                                                                                                                  |
| ------------------------------------------------ | ------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Anthropic-compatible `POST /v1/messages` ingress | API / Backend | Frontend Server (SSR) — | The gateway owns the HTTP contract and normalization boundary, not a browser client. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]                              |
| SSE stream emission                              | API / Backend | CDN / Static —          | Event ordering and backpressure behavior must be controlled by the gateway response layer. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]                        |
| Advisor tool injection/interception              | API / Backend | Database / Storage —    | This is orchestration logic on the request path; storage is secondary only if future ledger persistence is added. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md] |
| Provider transport calls                         | API / Backend | Database / Storage —    | Provider-specific communication belongs behind adapters owned by the backend. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]                                |
| Canonical transcript normalization               | API / Backend | Database / Storage      | Transcript semantics are computed in-process first, with optional future persistence. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]                             |
| Golden fixtures / contract tests                 | API / Backend | —                       | The spike validates protocol behavior around backend ingress and streaming semantics. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]                             |

## Project Constraints (from CLAUDE.md)

- Follow project docs and retrieval-led reasoning before making implementation recommendations. [CITED: /home/seven/.claude/CLAUDE.md]
- Break non-trivial work into atomic tasks with explicit acceptance criteria and verification. [CITED: /home/seven/.claude/CLAUDE.md]
- Treat ambiguity affecting architecture or scope as a decision point instead of silently locking assumptions. [CITED: /home/seven/.claude/CLAUDE.md]
- Use TDD for implementation work; Phase 0 itself is a design spike, so the planner should prepare fixture-first validation rather than production coding. [CITED: /home/seven/.claude/CLAUDE.md]
- Prefer minimal change and avoid broad adjacent refactors. [CITED: /home/seven/.claude/CLAUDE.md]
- The stack is not locked; do not present FastAPI/LiteLLM details as final product commitments. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]
- Phase 0 is a spike/design phase and should not turn into production implementation. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]
- Third-party doc details with unresolved verification gaps must stay provisional. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]

## Standard Stack

### Core

| Library                    | Version   | Purpose                                                                                                                                                      | Why Standard                                                                                                                                                                                                                                                                       |
| -------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fastapi` [VERIFIED: PyPI] | 0.136.3   | Candidate HTTP ingress layer for `POST /v1/messages` and streaming response plumbing. [CITED: https://pypi.org/project/fastapi/]                             | FastAPI officially documents `StreamingResponse`, making it a practical spike candidate for streaming ingress evaluation. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/]                                                                                          |
| `starlette` [ASSUMED]      | [ASSUMED] | Underlying response primitives and ASGI behavior for FastAPI.                                                                                                | FastAPI’s streaming behavior is built on Starlette response primitives; official Starlette docs explicitly document `StreamingResponse` and `EventSourceResponse`. [CITED: https://www.starlette.io/responses/]                                                                    |
| `litellm` [VERIFIED: PyPI] | 1.86.2    | Candidate provider abstraction layer for streaming, multi-provider access, and usage/cost-related proxy features. [CITED: https://pypi.org/project/litellm/] | LiteLLM officially documents streaming support and proxy features related to usage, routing, and budgets, which makes it suitable for a controlled adapter spike. [CITED: https://docs.litellm.ai/docs/completion/stream] [CITED: https://docs.litellm.ai/docs/proxy/virtual_keys] |

### Supporting

| Library                          | Version   | Purpose                                                                                                                                             | When to Use                                                                                                                                                                |
| -------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sse-starlette` [VERIFIED: PyPI] | 3.4.4     | Dedicated SSE response helper if plain `StreamingResponse` proves awkward for named-event framing. [CITED: https://pypi.org/project/sse-starlette/] | Use only if the spike shows that raw `StreamingResponse` makes event formatting or keepalive behavior harder to reason about. [CITED: https://www.starlette.io/responses/] |
| `pytest` [ASSUMED]               | [ASSUMED] | Fixture-driven spike verification.                                                                                                                  | Use if the implementation phase chooses Python and wants golden fixture regression tests from the start. [ASSUMED]                                                         |

### Alternatives Considered

| Instead of                   | Could Use                                                          | Tradeoff                                                                                                                                                                                      |
| ---------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FastAPI spike path           | Lower-level Starlette-only ingress                                 | More control over streaming details, but less ergonomic for rapid spike output. [ASSUMED]                                                                                                     |
| LiteLLM-backed adapter spike | Direct provider-specific adapter spike first                       | Less abstraction ambiguity, but weaker evidence about whether LiteLLM can satisfy R14 without leakage. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md] |
| `sse-starlette` helper       | Hand-built `text/event-stream` formatting over `StreamingResponse` | Fewer dependencies, but more custom event framing surface in a protocol-sensitive phase. [CITED: https://www.starlette.io/responses/] [ASSUMED]                                               |

**Installation:**

```bash
pip install fastapi litellm sse-starlette
```

**Version verification:**

```bash
pip index versions fastapi
pip index versions litellm
pip index versions sse-starlette
```

## Package Legitimacy Audit

| Package       | Registry | Age                                                      | Downloads | Source Repo                                                 | slopcheck | Disposition                       |
| ------------- | -------- | -------------------------------------------------------- | --------- | ----------------------------------------------------------- | --------- | --------------------------------- |
| fastapi       | PyPI     | [ASSUMED: mature package; exact age not re-queried]      | [ASSUMED] | [ASSUMED: official repo exists, not re-verified in-session] | OK        | Approved for spike only           |
| litellm       | PyPI     | [ASSUMED: mature package; exact age not re-queried]      | [ASSUMED] | [ASSUMED: official repo exists, not re-verified in-session] | OK        | Approved for spike only           |
| sse-starlette | PyPI     | [ASSUMED: established package; exact age not re-queried] | [ASSUMED] | [ASSUMED: official repo exists, not re-verified in-session] | OK        | Approved as optional spike helper |

**Packages removed due to slopcheck [SLOP] verdict:** none
**Packages flagged as suspicious [SUS]:** none

Slopcheck scan result: `slopcheck install -e pypi fastapi litellm sse-starlette` reported `3 OK` before the actual install step failed under an externally-managed Python environment. The legitimacy signal is still useful for planning, but package installs should happen inside a project venv during implementation. [VERIFIED: slopcheck CLI output]

## Architecture Patterns

### System Architecture Diagram

```text
Claude Code / client
        |
        v
Anthropic-compatible POST /v1/messages ingress
        |
        v
Request normalizer -> canonical transcript builder -> advisor tool injector
        |
        v
Provider adapter selector ---> capability metadata
        |
        v
Executor provider call (stream or non-stream)
        |
        +--> if no advisor tool use ------------------------------+
        |                                                         |
        v                                                         v
stream passthrough / normalized response                  final API response
        |
        +--> if advisor tool invocation detected
                |
                v
         hold/pause executor stream
                |
                v
         advisor subcall (non-streaming in v1)
                |
                v
         transcript result/history injection
                |
                v
         resume executor stream
                |
                v
         emit ordered SSE events / final completion
```

### Recommended Project Structure

```text
.planning/
├── phases/phase-0/
│   ├── RESEARCH.md              # planning research
│   ├── docs/                    # contract and risk drafts
│   ├── fixtures/                # protocol fixture drafts
│   ├── scripts/                 # planning validation scripts
│   └── *-VALIDATION.md          # phase validation artifact
src/ [ASSUMED]
├── ingress/                     # Anthropic-compatible HTTP boundary
├── orchestration/               # advisor state machine + transcript logic
├── adapters/                    # provider adapter interface + implementations
└── tests/                       # golden contract fixtures and replay tests
```

### Pattern 1: Fixture-First Protocol Boundary

**What:** Define request/response and stream event fixtures before implementation so every later task works against fixed protocol examples. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]
**When to use:** Immediately in Phase 0 for R1, R3, R9, R10, R13, R14, and R22. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]

### Pattern 2: Gateway-Owned Streaming State Machine

**What:** Treat advisor interception as an explicit state machine: `pre_executor -> streaming_executor -> hold -> advisor_subcall -> inject -> resume -> complete`. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/research/SUMMARY.md]
**When to use:** For all streaming paths where advisor invocation can interrupt executor output. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]

### Pattern 3: Transport-Agnostic Provider Adapter

**What:** Define one normalized interface below ingress and orchestration. Provider-specific SDK semantics stay inside adapters. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]
**When to use:** Before any provider implementation begins. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]

### Anti-Patterns to Avoid

- Ingress knows provider details: this violates R13 and makes future adapters expensive. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]
- Streaming implemented before event fixtures exist: this invites accidental event-order drift that golden tests cannot later explain. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]
- Custom transcript shapes per provider: this breaks advisor result injection and multi-turn replay semantics. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]
- Locking FastAPI/LiteLLM before spike evidence: project docs explicitly say the stack is still candidate-only. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]

## Don't Hand-Roll

| Problem                               | Don't Build                                      | Use Instead                                                                | Why                                                                                                                                                                                                                            |
| ------------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| HTTP streaming primitive              | Custom socket-level HTTP streamer                | FastAPI/Starlette response primitives                                      | Official docs already document `StreamingResponse`; Phase 0 should test semantics, not invent transport plumbing. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/] [CITED: https://www.starlette.io/responses/] |
| SSE response helper                   | Custom SSE framing utility before spike evidence | Start with `StreamingResponse`; escalate to `sse-starlette` only if needed | Keeps Phase 0 narrow and makes the need for a helper an evidence-based decision. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/] [CITED: https://www.starlette.io/responses/]                                  |
| Multi-provider abstraction in ingress | Provider-specific branching in route handlers    | Dedicated adapter interface                                                | R13-R14 already require this seam. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]                                                                                               |
| Transcript merging logic per test     | Ad hoc inline transcript patches                 | Canonical transcript model + fixtures                                      | The project has already identified transcript semantics as foundational. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]                                                              |

**Key insight:** The expensive mistakes in this phase are semantic, not algorithmic. Hand-rolling framing, adapter branching, or transcript shape early will create fake progress while hiding protocol ambiguity. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]

## Common Pitfalls

### Pitfall 1: Mixing transport events with transcript semantics

**What goes wrong:** The planner treats raw SSE chunks as the source of truth instead of maintaining a canonical transcript. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]
**Why it happens:** Streaming work starts before the canonical model is defined. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]
**How to avoid:** Define transcript fields and event-to-transcript mapping in P0 before implementation. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]

### Pitfall 2: Pause/resume without explicit ordering invariants

**What goes wrong:** Advisor insertion resumes the stream in a way clients cannot consume deterministically. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]
**Why it happens:** Hold/resume is treated as an implementation detail rather than a contract behavior. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md]
**How to avoid:** Require fixtures for pre-hold, hold marker, advisor injection, resume marker, and terminal event ordering. [ASSUMED]

### Pitfall 3: Assuming FastAPI implies first-class SSE semantics by itself

**What goes wrong:** The team underestimates the gap between generic streaming and protocol-specific SSE event framing. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/] [CITED: https://www.starlette.io/responses/]
**Why it happens:** FastAPI officially documents `StreamingResponse`, while Starlette documents `EventSourceResponse` as a third-party response class; that means SSE ergonomics still need a spike decision. [CITED: https://fastapi.tiangolo.com/advanced/custom-response/] [CITED: https://www.starlette.io/responses/]
**How to avoid:** Add a P0 verification command that emits named events and reconnection-safe framing before stack lock-in. [ASSUMED]

### Pitfall 4: Assuming LiteLLM satisfies the full adapter contract automatically

**What goes wrong:** The adapter interface leaks provider-specific behavior because LiteLLM coverage is broader than the project’s normalized contract. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md]
**Why it happens:** LiteLLM’s docs confirm streaming and usage/budget-oriented features, but not this project’s exact advisor/state-machine semantics. [CITED: https://docs.litellm.ai/docs/completion/stream] [CITED: https://docs.litellm.ai/docs/proxy/virtual_keys]
**How to avoid:** Validate each R14 capability against a thin adapter spike and mark unsupported behavior explicitly. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]

## Open Questions (RESOLVED FOR PLANNING)

1. **What is the minimum Anthropic Messages-compatible contract boundary for Phase 0?**
   - Resolution for planning: Phase 0 may lock only project-owned draft boundaries and acceptance fixture families. It may not lock unverified Anthropic field-level facts as final truth.
   - Proceeding rule: Every field or event used by the drafts must be tagged through a gap ledger as `verified`, `assumed`, or `gap`; `assumed` and `gap` items stay provisional and must be re-validated before production implementation.
   - Planning consequence: This does not block planning. It creates a mandatory Phase 0 deliverable and a caveat carried into P1 entry criteria.

2. **Should the spike use raw `StreamingResponse` or an SSE helper?**
   - Resolution for planning: Keep both `StreamingResponse` and an optional SSE helper in candidate scope, but do not lock either as the final production choice in Phase 0.
   - Proceeding rule: Phase 0 must record the framing decision criteria, risks, and verification method in the risk register and technology viability summary; implementation phases choose only after spike evidence exists.
   - Planning consequence: This is a candidate-stack caveat, not a blocker.

3. **Can LiteLLM satisfy the full adapter contract without leakage?**
   - Resolution for planning: Treat LiteLLM as a candidate adapter substrate, not as proof that R14 is already satisfied.
   - Proceeding rule: Phase 0 must define the adapter conformance checklist and capability boundary in project-owned drafts; any LiteLLM-specific mismatch remains a named risk until later validation closes it.
   - Planning consequence: Planning proceeds with an explicit conformance gap and no provider-specific commitments in ingress/orchestrator contracts.

## Environment Availability

| Dependency         | Required By                     | Available | Version        | Fallback                                        |
| ------------------ | ------------------------------- | --------- | -------------- | ----------------------------------------------- |
| Python             | Candidate implementation stack  | ✓         | 3.12.9         | —                                               |
| Node.js            | Context7 CLI / tooling fallback | ✓         | v26.1.0        | —                                               |
| npm                | Context7 CLI / tooling fallback | ✓         | 10.33.0        | —                                               |
| pip                | Python package verification     | ✓         | 24.3.1         | —                                               |
| ctx7 CLI           | Preferred docs lookup fallback  | ✗         | —              | `npx ctx7@latest ...` worked but quota exceeded |
| Context7 API quota | Official docs verification      | ✗         | quota exceeded | manual official URL fetches where accessible    |
| slopcheck          | Package legitimacy gate         | ✓         | 0.6.1          | —                                               |

**Missing dependencies with no fallback:**

- None for research writing.

**Missing dependencies with fallback:**

- Context7 quota prevented normal doc retrieval; planner should use the recorded manual verification commands before implementation. [VERIFIED: ctx7 CLI output]
- Anthropic docs were region-blocked during this session; planner should rerun verification from an allowed environment. [VERIFIED: environment fetch attempt]

## Security Domain

### Applicable ASVS Categories

| ASVS Category         | Applies | Standard Control                                                                                                                                                                         |
| --------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| V2 Authentication     | yes     | Header/auth allowlist at ingress boundary. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]                                                 |
| V3 Session Management | no      | Phase 0 does not implement persistent session behavior; only define ledger boundary. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]       |
| V4 Access Control     | yes     | Restrict provider base URL/model/routing targets. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]                                          |
| V5 Input Validation   | yes     | Normalize and validate ingress payload shape before adapter use. [ASSUMED]                                                                                                               |
| V6 Cryptography       | no      | No custom crypto should appear in Phase 0; secret handling is policy and redaction design only. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md] |

### Known Threat Patterns for this stack

| Pattern                              | STRIDE                 | Standard Mitigation                                                                                                                                   |
| ------------------------------------ | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Header passthrough leaks secrets     | Information Disclosure | Allowlist ingress headers and redact logs/errors. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]       |
| Arbitrary provider routing           | Tampering              | Restrict base URLs/models and normalize adapter config. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md] |
| Advisor result inserted out of order | Tampering              | Canonical transcript plus golden event ordering fixtures. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md]    |
| Provider error data exposed verbatim | Information Disclosure | Normalize error shapes before returning to clients. [CITED: /home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md]     |

## Suggested Verification Commands

```bash
# Anthropic docs re-validation when network/region permits
npx ctx7@latest library "Anthropic" "Messages API request response streaming fields stop_reason usage"
npx ctx7@latest docs /anthropic/anthropic-docs "Messages API request response streaming fields stop_reason usage"

# FastAPI / Starlette streaming verification
python - <<'PY'
from fastapi.responses import StreamingResponse
print(StreamingResponse)
PY
python - <<'PY'
from sse_starlette import EventSourceResponse
print(EventSourceResponse)
PY

# LiteLLM capability spot-check
python - <<'PY'
from litellm import completion
print('litellm import ok')
PY

# Registry and legitimacy checks
pip index versions fastapi
pip index versions litellm
pip index versions sse-starlette
slopcheck install -e pypi fastapi litellm sse-starlette
```

## Sources

### Primary (HIGH confidence)

- [FastAPI custom responses](https://fastapi.tiangolo.com/advanced/custom-response/) - verified `StreamingResponse` documentation.
- [Starlette responses](https://www.starlette.io/responses/) - verified `StreamingResponse` and `EventSourceResponse` documentation.
- [LiteLLM streaming docs](https://docs.litellm.ai/docs/completion/stream) - verified streaming support via `stream=True`.
- [LiteLLM virtual keys docs](https://docs.litellm.ai/docs/proxy/virtual_keys) - verified docs exposure of usage/cost/budget-related proxy concepts.
- [PyPI fastapi](https://pypi.org/project/fastapi/) - verified current package version metadata.
- [PyPI litellm](https://pypi.org/project/litellm/) - verified current package version metadata.
- [PyPI sse-starlette](https://pypi.org/project/sse-starlette/) - verified current package version metadata.
- [/home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md](file:///home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/PROJECT.md) - project scope and constraints.
- [/home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md](file:///home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/REQUIREMENTS.md) - requirement baselines.
- [/home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md](file:///home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/ROADMAP.md) - Phase 0 goals and deliverables.
- [/home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md](file:///home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/STATE.md) - current risks and blockers.
- [/home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/research/SUMMARY.md](file:///home/seven/data/coding/projects/seven/llm-gateway-advisor/.planning/research/SUMMARY.md) - prior consolidated findings and verification gaps.

### Secondary (MEDIUM confidence)

- Context7 CLI fallback attempt - quota exceeded during this session, confirming a live verification gap rather than a completed doc lookup. [VERIFIED: ctx7 CLI output]

### Tertiary (LOW confidence)

- Anthropic Messages field-level contract details remain unverified in-session because the official docs page was region-blocked here. [VERIFIED: environment fetch attempt]

## Metadata

**Confidence breakdown:**

- Standard stack: MEDIUM - FastAPI/Starlette/LiteLLM package and doc signals were verified, but the project explicitly has not locked the stack.
- Architecture: HIGH - Core architectural boundaries are strongly anchored in project docs.
- Pitfalls: MEDIUM - Major risks are documented by project state, but some warning-sign details are assumption-level heuristics.

**Research date:** 2026-05-29
**Valid until:** 2026-06-05
