---
phase: phase-0
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - .planning/phases/phase-0/PLAN-01.md
  - .planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md
  - .planning/phases/phase-0/docs/api-contract-draft.md
  - .planning/phases/phase-0/docs/config-shape-draft.md
  - .planning/phases/phase-0/docs/provider-adapter-interface-draft.md
  - .planning/phases/phase-0/docs/canonical-transcript-model-draft.md
  - .planning/phases/phase-0/phase-0-01-SUMMARY.md
autonomous: true
requirements:
  - R1
  - R3
  - R9
  - R10
  - R13
  - R14
  - R22
must_haves:
  truths:
    - "A Phase 1 implementer can read project-owned drafts for POST /v1/messages and know the minimum non-streaming and streaming request/response surface required for R1 and R3."
    - "A Phase 1 implementer can build against a provider adapter seam and canonical transcript model without coupling ingress logic to provider-specific details, satisfying R10, R13, and R14."
    - "A reviewer can distinguish verified Anthropic-compatible contract details from assumed details and open gaps before implementation starts."
  artifacts:
    - path: ".planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md"
      provides: "Verified/assumed/gap ledger for external contract details"
    - path: ".planning/phases/phase-0/docs/api-contract-draft.md"
      provides: "Draft Anthropic Messages-compatible ingress contract for non-stream and stream paths"
    - path: ".planning/phases/phase-0/docs/config-shape-draft.md"
      provides: "Draft Phase 1 configuration surface and scoping boundaries"
    - path: ".planning/phases/phase-0/docs/provider-adapter-interface-draft.md"
      provides: "Draft adapter seam and normalized provider capability contract"
    - path: ".planning/phases/phase-0/docs/canonical-transcript-model-draft.md"
      provides: "Draft canonical transcript fields, invariants, and event mapping"
    - path: ".planning/phases/phase-0/phase-0-01-SUMMARY.md"
      provides: "Executor summary for plan 01"
  key_links:
    - from: ".planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md"
      to: ".planning/phases/phase-0/docs/api-contract-draft.md"
      via: "field/event certainty tags referenced by contract sections"
      pattern: "verified|assumed|gap"
    - from: ".planning/phases/phase-0/docs/provider-adapter-interface-draft.md"
      to: ".planning/phases/phase-0/docs/canonical-transcript-model-draft.md"
      via: "transcript/event types consumed by adapter outputs"
      pattern: "canonical transcript|usage metadata|tool_use_id"
---

<objective>
Create Phase 0 contract, config, adapter, and transcript draft artifacts.

Purpose: Lock project-owned protocol boundaries before Phase 1 implementation begins, while keeping all external contract uncertainty explicit.
Output: Draft contract gap ledger, API contract, config shape, provider adapter interface, canonical transcript model, and plan summary under `.planning/phases/phase-0/`.
</objective>

<execution_context>
@/home/seven/data/coding/projects/seven/llm-gateway-advisor/.claude/get-shit-done/workflows/execute-plan.md
@/home/seven/data/coding/projects/seven/llm-gateway-advisor/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/ROADMAP.md
@.planning/REQUIREMENTS.md
@.planning/STATE.md
@.planning/phases/phase-0/RESEARCH.md

<interfaces>
From `.planning/REQUIREMENTS.md`:
- R1: provide an Anthropic Messages-compatible `POST /v1/messages` ingress.
- R3: support SSE streaming with consumable event sequences.
- R9: preserve correct SSE event order after advisor hold/resume.
- R10: preserve roles, content block order, block type, IDs, tool_use_id, stop_reason, and usage metadata in the canonical transcript.
- R13: keep all provider calls behind an adapter boundary.
- R14: adapter v1 scope covers non-stream messages, streaming messages, capability metadata, usage extraction, and error normalization.
- R22: define golden contract tests for key non-streaming and streaming behavior.

Phase 0 scope rule:

- Produce planning/protocol-spike artifacts only.
- Do not build gateway runtime, real provider adapters, or SSE endpoint implementation in this phase.
  </interfaces>
  </context>

<tasks>

<task type="auto">
  <name>Task 1: Draft official-contract gap ledger</name>
  <files>.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md</files>
  <action>Create `.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md` as the Phase 0 field/event ledger for Anthropic-compatible planning inputs. Record the minimum request fields, response fields, streaming event names/shapes, stop reasons, usage fields, and advisor-relevant semantics used by later drafts. Mark every entry as `Status: verified`, `Status: assumed`, or `Status: gap`; include source notes and `Planning consequence:` per entry. Keep unverified Anthropic details provisional.</action>
  <verify>
    <automated>python - <<'PY'
from pathlib import Path
ledger = Path('.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md').read_text()
for token in ['# Anthropic Contract Gap Ledger','Status: verified','Status: assumed','Status: gap','Planning consequence:','stream','usage']:
    if token not in ledger:
        raise SystemExit(f'missing ledger token: {token}')
print('phase 0 contract gap ledger ok')
PY</automated>
  </verify>
  <done>The gap ledger exists with explicit verified/assumed/gap tagging for external contract details used by later drafts.</done>
</task>

<task type="auto">
  <name>Task 2: Draft API contract and config shape artifacts</name>
  <files>.planning/phases/phase-0/docs/api-contract-draft.md, .planning/phases/phase-0/docs/config-shape-draft.md</files>
  <action>Create `.planning/phases/phase-0/docs/api-contract-draft.md` as the project-owned draft contract for `POST /v1/messages`, covering request envelope, non-streaming response shape, streaming response/event framing expectations, required versus optional fields, known gaps from the ledger, and traceability back to R1, R3, R9, and R22. Create `.planning/phases/phase-0/docs/config-shape-draft.md` defining the minimum Phase 1 configuration surface for ingress behavior, provider routing selection, candidate adapter settings, stream/orchestration knobs, and fixture/test toggles. Include which fields are required for Phase 1 versus deferred beyond Phase 1. Describe draft shapes, constraints, and examples only.</action>
  <verify>
    <automated>python - <<'PY'
from pathlib import Path
api = Path('.planning/phases/phase-0/docs/api-contract-draft.md').read_text()
cfg = Path('.planning/phases/phase-0/docs/config-shape-draft.md').read_text()
for token in ['# API Contract Draft','POST /v1/messages','Non-streaming','Streaming','R1','R3','R9','R22','fixture_id:']:
    if token not in api:
        raise SystemExit(f'missing api token: {token}')
for token in ['# Config Shape Draft','provider','stream','advisor','required for Phase 1','deferred beyond Phase 1']:
    if token not in cfg:
        raise SystemExit(f'missing config token: {token}')
print('api and config drafts ok')
PY</automated>
  </verify>
  <done>The API contract draft and config shape draft exist, are tied to Phase 0 requirements, and define draft protocol/config boundaries rather than runtime code.</done>
</task>

<task type="auto">
  <name>Task 3: Draft provider adapter and canonical transcript contracts</name>
  <files>.planning/phases/phase-0/docs/provider-adapter-interface-draft.md, .planning/phases/phase-0/docs/canonical-transcript-model-draft.md</files>
  <action>Create `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md` describing the normalized adapter boundary for non-streaming messages, streaming messages, capability metadata, usage extraction, and error normalization per R13 and R14. Name the draft request/response/event types the interface expects and explicitly note what stays provider-specific below the boundary. Create `.planning/phases/phase-0/docs/canonical-transcript-model-draft.md` describing canonical transcript entities, field semantics, ordering invariants, block types, IDs, tool_use_id relationships, stop_reason handling, usage metadata, and how streaming events map into transcript updates. Tie both drafts back to the gap ledger where field/event certainty is provisional.</action>
  <verify>
    <automated>python - <<'PY'
from pathlib import Path
adapter = Path('.planning/phases/phase-0/docs/provider-adapter-interface-draft.md').read_text()
transcript = Path('.planning/phases/phase-0/docs/canonical-transcript-model-draft.md').read_text()
for token in ['# Provider Adapter Interface Draft','non-streaming messages','streaming messages','capability metadata','usage extraction','error normalization','R13','R14']:
    if token not in adapter:
        raise SystemExit(f'missing adapter token: {token}')
for token in ['# Canonical Transcript Model Draft','roles','content block order','block type','tool_use_id','stop_reason','usage metadata','R10']:
    if token not in transcript:
        raise SystemExit(f'missing transcript token: {token}')
print('adapter and transcript drafts ok')
PY</automated>
  </verify>
  <done>The adapter and transcript drafts define the required seams and invariants without leaking provider-specific behavior into ingress/orchestration planning.</done>
</task>

</tasks>

<threat_model>

## Trust Boundaries

| Boundary                                         | Description                                                                                                            |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| external docs -> project-owned drafts            | Unverified third-party fields/events can leak into project contracts if the gap ledger does not mark them provisional. |
| draft adapter boundary -> Phase 1 implementation | Provider-specific details can cross into ingress/orchestration if the adapter contract is ambiguous.                   |

## STRIDE Threat Register

| Threat ID    | Category    | Component                                                           | Disposition | Mitigation Plan                                                                                          |
| ------------ | ----------- | ------------------------------------------------------------------- | ----------- | -------------------------------------------------------------------------------------------------------- |
| T-phase-0-01 | Tampering   | `.planning/phases/phase-0/docs/api-contract-draft.md`               | mitigate    | Require gap-ledger traceability and explicit provisional tagging for any field/event not fully verified. |
| T-phase-0-02 | Tampering   | `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md` | mitigate    | Define the adapter seam so provider-specific behavior stays below the boundary.                          |
| T-phase-0-03 | Repudiation | requirement traceability across drafts                              | mitigate    | Map R1, R3, R9, R10, R13, R14, and R22 in draft artifacts and verification checks.                       |
| T-phase-0-SC | Tampering   | npm/pip/cargo installs                                              | accept      | This Phase 0 plan creates planning artifacts only and performs no package installation.                  |

</threat_model>

<verification>
Run each task automated verification, then run:

- `python - <<'PY'\nfrom pathlib import Path\ntext = Path('.planning/phases/phase-0/PLAN-01.md').read_text()\nif '.claude/worktrees/' + 'agent-' in text:\n    raise SystemExit('stale worktree path found')\nif text.count('<task type=') >= 5:\n    raise SystemExit('too many tasks in PLAN-01.md')\nfor token in ['api-contract-draft.md','config-shape-draft.md','provider-adapter-interface-draft.md','canonical-transcript-model-draft.md','anthropic-contract-gap-ledger.md']:\n    if token not in text:\n        raise SystemExit(f'missing deliverable path: {token}')\nprint('PLAN-01 structural checks ok')\nPY`
  </verification>

<success_criteria>

- PLAN-01.md has fewer than five executable tasks.
- Contract, config, adapter, and transcript draft artifacts are produced under `.planning/phases/phase-0/`.
- R1, R3, R9, R10, R13, R14, and R22 have concrete draft-artifact traceability.
- Phase 0 remains a protocol spike and does not schedule production gateway runtime, real adapters, or an SSE endpoint.

</success_criteria>

<output>
Create `.planning/phases/phase-0/phase-0-01-SUMMARY.md` when done.
</output>
