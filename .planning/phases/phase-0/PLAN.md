---
phase: phase-0
plan: index
type: index
wave: 0
depends_on: []
files_modified:
  - .planning/phases/phase-0/PLAN.md
  - .planning/phases/phase-0/PLAN-01.md
  - .planning/phases/phase-0/PLAN-02.md
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
    - "Phase 0 execution is split into smaller plans with fewer than five tasks each."
    - "Protocol-spike scope remains limited to planning artifacts, fixtures, risk analysis, and recommendation output."
  artifacts:
    - path: ".planning/phases/phase-0/PLAN-01.md"
      provides: "Contract, config, adapter, and transcript drafting execution plan"
    - path: ".planning/phases/phase-0/PLAN-02.md"
      provides: "Fixture, risk, summary, and Nyquist validation execution plan"
  key_links:
    - from: ".planning/phases/phase-0/PLAN.md"
      to: ".planning/phases/phase-0/PLAN-01.md"
      via: "routing reference"
      pattern: "PLAN-01.md"
    - from: ".planning/phases/phase-0/PLAN.md"
      to: ".planning/phases/phase-0/PLAN-02.md"
      via: "routing reference"
      pattern: "PLAN-02.md"
---

<objective>
Route Phase 0 execution to two smaller GSD execution plans.

Purpose: Keep each executable plan below the GSD scope sanity task threshold while preserving full Phase 0 protocol-spike coverage.
Output: Use `PLAN-01.md` and `PLAN-02.md` for execution; this file is an index only and intentionally contains no executable tasks.
</objective>

<context>
@.planning/ROADMAP.md
@.planning/REQUIREMENTS.md
@.planning/STATE.md
@.planning/phases/phase-0/RESEARCH.md
@.planning/phases/phase-0/NYQUIST-VALIDATION.md
</context>

<plan_routing>

| Plan         | Wave | Purpose                                                                                                                                                            | Requirements                   |
| ------------ | ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------ |
| `PLAN-01.md` | 1    | Draft contract gap ledger, API contract, config shape, provider adapter interface, and canonical transcript model.                                                 | R1, R3, R9, R10, R13, R14, R22 |
| `PLAN-02.md` | 2    | Create fixture manifest and draft fixtures, fixture verifier, streaming orchestration risk register, SUMMARY/P1 recommendation, and split-plan Nyquist validation. | R3, R9, R10, R14, R22          |

</plan_routing>

<scope_guardrails>

- Phase 0 produces protocol-spike planning artifacts only.
- Do not implement production gateway runtime, real provider adapters, or an SSE endpoint in Phase 0.
- Do not use stale isolated worktree paths; all paths are relative to the main project workspace.

</scope_guardrails>

<verification>
Run the validation commands listed in `PLAN-01.md`, `PLAN-02.md`, and `NYQUIST-VALIDATION.md`.
</verification>

<success_criteria>

- `PLAN-01.md` and `PLAN-02.md` each contain fewer than five executable tasks.
- All Phase 0 deliverables are owned by one of the split plans.
- R1, R3, R9, R10, R13, R14, and R22 are covered across the split plans.

</success_criteria>
