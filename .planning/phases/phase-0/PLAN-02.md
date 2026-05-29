---
phase: phase-0
plan: 02
type: execute
wave: 2
depends_on:
  - phase-0-01
files_modified:
  - .planning/phases/phase-0/PLAN-02.md
  - .planning/phases/phase-0/fixtures/manifest.yaml
  - .planning/phases/phase-0/fixtures/non-stream-basic.yaml
  - .planning/phases/phase-0/fixtures/stream-hold-resume.yaml
  - .planning/phases/phase-0/scripts/verify_phase0_fixtures.py
  - .planning/phases/phase-0/docs/streaming-orchestration-risk-register.md
  - .planning/phases/phase-0/SUMMARY.md
  - .planning/phases/phase-0/NYQUIST-VALIDATION.md
  - .planning/phases/phase-0/phase-0-02-SUMMARY.md
autonomous: true
requirements:
  - R3
  - R9
  - R10
  - R14
  - R22
must_haves:
  truths:
    - "A tester can identify fixture families and event-order expectations for non-streaming and streaming hold/resume behavior, covering R9 and R22."
    - "A reviewer can determine whether the candidate stack remains viable for Phase 1 and which streaming/orchestration caveats remain open before runtime implementation begins."
    - "NYQUIST-VALIDATION.md proves coverage across PLAN-01.md and PLAN-02.md for R1, R3, R9, R10, R13, R14, and R22."
  artifacts:
    - path: ".planning/phases/phase-0/fixtures/manifest.yaml"
      provides: "Golden-fixture inventory for non-stream and stream acceptance cases"
    - path: ".planning/phases/phase-0/fixtures/non-stream-basic.yaml"
      provides: "Draft non-streaming request/response fixture"
    - path: ".planning/phases/phase-0/fixtures/stream-hold-resume.yaml"
      provides: "Draft streaming advisor hold/resume ordering fixture"
    - path: ".planning/phases/phase-0/scripts/verify_phase0_fixtures.py"
      provides: "Static fixture completeness verifier"
    - path: ".planning/phases/phase-0/docs/streaming-orchestration-risk-register.md"
      provides: "Named streaming/pause/resume/orchestration risks and disposition"
    - path: ".planning/phases/phase-0/SUMMARY.md"
      provides: "Technology viability summary and Phase 1 entry recommendation"
    - path: ".planning/phases/phase-0/NYQUIST-VALIDATION.md"
      provides: "Split-plan requirement and deliverable coverage validation"
    - path: ".planning/phases/phase-0/phase-0-02-SUMMARY.md"
      provides: "Executor summary for plan 02"
  key_links:
    - from: ".planning/phases/phase-0/fixtures/manifest.yaml"
      to: ".planning/phases/phase-0/docs/api-contract-draft.md"
      via: "fixture IDs referenced from contract examples"
      pattern: "fixture_id:"
    - from: ".planning/phases/phase-0/docs/streaming-orchestration-risk-register.md"
      to: ".planning/phases/phase-0/SUMMARY.md"
      via: "Phase 1 go/no-go caveats summarized from risk dispositions"
      pattern: "Phase 1 recommendation|blocking|caveat"
    - from: ".planning/phases/phase-0/NYQUIST-VALIDATION.md"
      to: ".planning/phases/phase-0/PLAN-01.md"
      via: "split-plan coverage mapping"
      pattern: "PLAN-01"
    - from: ".planning/phases/phase-0/NYQUIST-VALIDATION.md"
      to: ".planning/phases/phase-0/PLAN-02.md"
      via: "split-plan coverage mapping"
      pattern: "PLAN-02"
---

<objective>
Create Phase 0 fixture, streaming risk, summary recommendation, and Nyquist validation artifacts.

Purpose: Convert draft protocol boundaries from PLAN-01 into fixture baselines, orchestration risk decisions, and an explicit Phase 1 entry recommendation without building runtime gateway code.
Output: Fixture manifest and drafts, fixture verifier, streaming orchestration risk register, SUMMARY.md, NYQUIST-VALIDATION.md, and plan summary under `.planning/phases/phase-0/`.
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
@.planning/phases/phase-0/PLAN-01.md
@.planning/phases/phase-0/docs/api-contract-draft.md
@.planning/phases/phase-0/docs/provider-adapter-interface-draft.md
@.planning/phases/phase-0/docs/canonical-transcript-model-draft.md

<interfaces>
PLAN-02 depends on PLAN-01 outputs:
- `api-contract-draft.md` supplies fixture IDs and stream/non-stream protocol expectations.
- `provider-adapter-interface-draft.md` supplies normalized adapter capability expectations for R14 risk analysis.
- `canonical-transcript-model-draft.md` supplies R10 transcript/event invariants used by stream hold/resume fixture assertions.

Phase 0 scope rule:

- Produce planning/protocol-spike artifacts only.
- Do not build gateway runtime, real provider adapters, or SSE endpoint implementation in this phase.
  </interfaces>
  </context>

<tasks>

<task type="auto">
  <name>Task 1: Create acceptance fixture manifest and draft fixture files</name>
  <files>.planning/phases/phase-0/fixtures/manifest.yaml, .planning/phases/phase-0/fixtures/non-stream-basic.yaml, .planning/phases/phase-0/fixtures/stream-hold-resume.yaml, .planning/phases/phase-0/scripts/verify_phase0_fixtures.py</files>
  <action>Create `.planning/phases/phase-0/fixtures/manifest.yaml` as the fixture inventory for future golden contract tests, including fixture IDs, scenario names, covered requirements, draft input/output files, and ordering assertions. Add at least two concrete draft fixtures: `.planning/phases/phase-0/fixtures/non-stream-basic.yaml` for a minimal non-streaming request/response case and `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml` for a streaming advisor hold/resume ordering case. Add `.planning/phases/phase-0/scripts/verify_phase0_fixtures.py` to statically verify that each manifest entry references an existing fixture file, lists requirement IDs, and includes stream-order assertions where `stream: true`. Keep the script lightweight and planning-only; it validates draft artifact completeness, not runtime behavior.</action>
  <verify>
    <automated>python .planning/phases/phase-0/scripts/verify_phase0_fixtures.py</automated>
  </verify>
  <done>The manifest, draft fixtures, and lightweight verifier exist so R22 has a concrete planning baseline and R9 stream-order expectations are encoded as fixture metadata instead of prose alone.</done>
</task>

<task type="auto">
  <name>Task 2: Record streaming orchestration risks and finalize Phase 1 recommendation</name>
  <files>.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md, .planning/phases/phase-0/SUMMARY.md</files>
  <action>Create `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md` listing named risks for SSE framing, advisor interception, hold/resume ordering, transcript/event divergence, candidate SSE helper choice, and LiteLLM conformance gaps. For each risk, include trigger, impact, mitigation probe, deliverable dependency, owner artifact, and disposition: blocking, caveated, or accepted-for-Phase-1. Create or complete `.planning/phases/phase-0/SUMMARY.md` with a technology viability section covering FastAPI, generic StreamingResponse versus SSE helper choice criteria, LiteLLM adapter-conformance caveats from RESEARCH.md, deliverables completed, open caveats, and an explicit `Phase 1 recommendation` section that allows only: continue, continue with caveats, or reassess stack.</action>
  <verify>
    <automated>python - <<'PY'
from pathlib import Path
risk = Path('.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md').read_text()
summary = Path('.planning/phases/phase-0/SUMMARY.md').read_text()
for token in ['# Streaming Orchestration Risk Register','SSE framing','hold/resume ordering','transcript/event divergence','LiteLLM','Disposition:','blocking','caveated']:
    if token not in risk:
        raise SystemExit(f'missing risk token: {token}')
for token in ['# Phase 0 Summary','Technology viability','FastAPI','LiteLLM','Deliverables completed','Open caveats','Phase 1 recommendation']:
    if token not in summary:
        raise SystemExit(f'missing summary token: {token}')
print('risk register and summary recommendation ok')
PY</automated>
  </verify>
  <done>The risk register names the spike-critical streaming/orchestration uncertainties, and SUMMARY.md turns contract drafts, fixture baseline, and risk dispositions into an explicit Phase 1 entry recommendation.</done>
</task>

<task type="auto">
  <name>Task 3: Update Nyquist validation for split plans</name>
  <files>.planning/phases/phase-0/NYQUIST-VALIDATION.md</files>
  <action>Rewrite `.planning/phases/phase-0/NYQUIST-VALIDATION.md` so it validates the split Phase 0 plan structure. Include fixed sections for requirement coverage, roadmap deliverable coverage, split plan structure, task-to-deliverable mapping, verification methods, known caveats, and Phase 1 entry conditions. Each of R1, R3, R9, R10, R13, R14, and R22 must map to one or more concrete draft artifacts and to PLAN-01 or PLAN-02 tasks. Include the Anthropic contract gap ledger as a prerequisite input to any draft that relies on provisional third-party details.</action>
  <verify>
    <automated>python - <<'PY'
from pathlib import Path
text = Path('.planning/phases/phase-0/NYQUIST-VALIDATION.md').read_text()
for token in ['# Phase 0 Nyquist Validation','## Requirement Coverage','## Deliverable Coverage','## Split Plan Structure','## Task-to-Deliverable Mapping','## Verification Method','## Known Caveats','## Phase 1 Entry Conditions','PLAN-01','PLAN-02','R1','R3','R9','R10','R13','R14','R22','anthropic-contract-gap-ledger.md','api-contract-draft.md','config-shape-draft.md','provider-adapter-interface-draft.md','canonical-transcript-model-draft.md','manifest.yaml','streaming-orchestration-risk-register.md','SUMMARY.md']:
    if token not in text:
        raise SystemExit(f'missing validation token: {token}')
print('nyquist validation updated for split plans')
PY</automated>
  </verify>
  <done>The validation artifact proves every Phase 0 roadmap deliverable and every in-scope requirement is supported by a concrete split-plan task and verification path.</done>
</task>

</tasks>

<threat_model>

## Trust Boundaries

| Boundary                                | Description                                                                                            |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| fixture manifest -> future golden tests | Missing or weak fixture metadata can cause later tests to validate the wrong event ordering semantics. |
| risk register -> Phase 1 start decision | Missing caveats can make a candidate stack look production-ready before spike evidence exists.         |
| split plans -> coverage validation      | Requirement coverage can be lost if Nyquist validation is not updated for PLAN-01/PLAN-02.             |

## STRIDE Threat Register

| Threat ID    | Category               | Component                                         | Disposition | Mitigation Plan                                                                                                          |
| ------------ | ---------------------- | ------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| T-phase-0-04 | Tampering              | `.planning/phases/phase-0/fixtures/manifest.yaml` | mitigate    | Add a verifier script that checks referenced files, requirement IDs, and stream-order assertions.                        |
| T-phase-0-05 | Information Disclosure | `.planning/phases/phase-0/SUMMARY.md`             | mitigate    | Summarize only planning-safe viability conclusions; do not present unresolved external contract details as settled fact. |
| T-phase-0-06 | Repudiation            | `.planning/phases/phase-0/NYQUIST-VALIDATION.md`  | mitigate    | Map each roadmap deliverable and requirement to PLAN-01/PLAN-02 tasks and automated verification.                        |
| T-phase-0-07 | Denial of Service      | Phase 1 start decision                            | mitigate    | Force an explicit Phase 1 recommendation with blocking/caveated outcomes derived from the risk register.                 |
| T-phase-0-SC | Tampering              | npm/pip/cargo installs                            | accept      | This Phase 0 plan creates planning artifacts only and performs no package installation.                                  |

</threat_model>

<verification>
Run each task automated verification, then run:

- `python - <<'PY'\nfrom pathlib import Path\ntext = Path('.planning/phases/phase-0/PLAN-02.md').read_text()\nif '.claude/worktrees/' + 'agent-' in text:\n    raise SystemExit('stale worktree path found')\nif text.count('<task type=') >= 5:\n    raise SystemExit('too many tasks in PLAN-02.md')\nfor token in ['fixtures/manifest.yaml','fixtures/non-stream-basic.yaml','fixtures/stream-hold-resume.yaml','scripts/verify_phase0_fixtures.py','streaming-orchestration-risk-register.md','SUMMARY.md','NYQUIST-VALIDATION.md']:\n    if token not in text:\n        raise SystemExit(f'missing deliverable path: {token}')\nprint('PLAN-02 structural checks ok')\nPY`
  </verification>

<success_criteria>

- PLAN-02.md has fewer than five executable tasks.
- Fixture manifest, draft fixtures, fixture verifier, risk register, SUMMARY.md, and NYQUIST-VALIDATION.md are produced under `.planning/phases/phase-0/`.
- R3, R9, R10, R14, and R22 have concrete artifact and verification traceability in this plan; R1 and R13 remain covered through PLAN-01 and are referenced in NYQUIST-VALIDATION.md.
- Phase 0 remains a protocol spike and does not schedule production gateway runtime, real adapters, or an SSE endpoint.

</success_criteria>

<output>
Create `.planning/phases/phase-0/phase-0-02-SUMMARY.md` when done.
</output>
