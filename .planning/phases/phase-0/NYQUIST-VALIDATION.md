# Phase 0 Nyquist Validation

## Purpose

This artifact validates that Phase 0 is planned as a split protocol-spike deliverable phase, not as a meta-repair exercise. It proves that each in-scope requirement and each roadmap deliverable is produced by a concrete task with an automated verification path while each executable plan remains below the GSD scope-sanity task threshold.

## Split Plan Structure

| Plan         | Wave | Task count | Owns                                                                                                                                       | Scope boundary                                        |
| ------------ | ---- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| `PLAN-01.md` | 1    | 3 tasks    | Contract gap ledger, API contract draft, config shape draft, provider adapter interface draft, canonical transcript model draft            | Draft protocol/config/interface artifacts only        |
| `PLAN-02.md` | 2    | 3 tasks    | Fixture manifest and draft fixtures, fixture verifier script, streaming orchestration risk register, `SUMMARY.md`, this Nyquist validation | Fixture/risk/recommendation/validation artifacts only |

Both executable plans contain fewer than five `<task type=` entries. `PLAN.md` is an index/routing file and intentionally contains no executable tasks.

## Requirement Coverage

| Requirement | Deliverable support                                                                                                                                                                                                                                                                            | Owning split plan/task(s)                                | Verification path                                                                                     |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| R1          | `.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md`, `.planning/phases/phase-0/docs/api-contract-draft.md`                                                                                                                                                                        | `PLAN-01.md` Task 1, Task 2                              | PLAN-01 ledger token check; PLAN-01 API draft token check                                             |
| R3          | `.planning/phases/phase-0/docs/api-contract-draft.md`, `.planning/phases/phase-0/fixtures/manifest.yaml`, `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml`, `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md`                                                | `PLAN-01.md` Task 2; `PLAN-02.md` Task 1, Task 2         | PLAN-01 API draft token check; PLAN-02 fixture verifier; PLAN-02 risk-register token check            |
| R9          | `.planning/phases/phase-0/docs/api-contract-draft.md`, `.planning/phases/phase-0/docs/canonical-transcript-model-draft.md`, `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml`, `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md`                              | `PLAN-01.md` Task 2, Task 3; `PLAN-02.md` Task 1, Task 2 | PLAN-01 API/transcript token checks; PLAN-02 fixture verifier; PLAN-02 risk-register token check      |
| R10         | `.planning/phases/phase-0/docs/canonical-transcript-model-draft.md`, `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml`                                                                                                                                                               | `PLAN-01.md` Task 3; `PLAN-02.md` Task 1                 | PLAN-01 transcript draft token check; PLAN-02 fixture verifier                                        |
| R13         | `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md`                                                                                                                                                                                                                            | `PLAN-01.md` Task 3                                      | PLAN-01 adapter draft token check                                                                     |
| R14         | `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md`, `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md`, `.planning/phases/phase-0/SUMMARY.md`                                                                                                           | `PLAN-01.md` Task 3; `PLAN-02.md` Task 2                 | PLAN-01 adapter draft token check; PLAN-02 risk and summary token checks                              |
| R22         | `.planning/phases/phase-0/fixtures/manifest.yaml`, `.planning/phases/phase-0/fixtures/non-stream-basic.yaml`, `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml`, `.planning/phases/phase-0/scripts/verify_phase0_fixtures.py`, `.planning/phases/phase-0/docs/api-contract-draft.md` | `PLAN-01.md` Task 2; `PLAN-02.md` Task 1                 | PLAN-01 API fixture marker check; `python .planning/phases/phase-0/scripts/verify_phase0_fixtures.py` |

## Deliverable Coverage

| Roadmap deliverable                         | Concrete artifact path                                                                                                                                                           | Owning split plan/task | Notes                                                                                        |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- | -------------------------------------------------------------------------------------------- |
| API contract draft                          | `.planning/phases/phase-0/docs/api-contract-draft.md`                                                                                                                            | `PLAN-01.md` Task 2    | Covers `POST /v1/messages` draft contract and stream/non-stream shape.                       |
| config shape draft                          | `.planning/phases/phase-0/docs/config-shape-draft.md`                                                                                                                            | `PLAN-01.md` Task 2    | Captures required versus deferred config surface for Phase 1.                                |
| provider adapter interface draft            | `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md`                                                                                                              | `PLAN-01.md` Task 3    | Defines normalized adapter seam and capability contract.                                     |
| canonical transcript model draft            | `.planning/phases/phase-0/docs/canonical-transcript-model-draft.md`                                                                                                              | `PLAN-01.md` Task 3    | Defines transcript entities, invariants, and event mapping.                                  |
| acceptance fixture list                     | `.planning/phases/phase-0/fixtures/manifest.yaml` plus `.planning/phases/phase-0/fixtures/non-stream-basic.yaml` and `.planning/phases/phase-0/fixtures/stream-hold-resume.yaml` | `PLAN-02.md` Task 1    | Establishes golden-fixture inventory and sample acceptance cases.                            |
| fixture verifier                            | `.planning/phases/phase-0/scripts/verify_phase0_fixtures.py`                                                                                                                     | `PLAN-02.md` Task 1    | Statically verifies manifest references, requirement IDs, and stream-order assertions.       |
| streaming/advisor orchestration risk record | `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md`                                                                                                         | `PLAN-02.md` Task 2    | Records blocking, caveated, and accepted streaming risks.                                    |
| technology viability / P1 recommendation    | `.planning/phases/phase-0/SUMMARY.md`                                                                                                                                            | `PLAN-02.md` Task 2    | Records viability inputs, deliverables completed, caveats, and Phase 1 recommendation.       |
| optional contract validation ledger         | `.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md`                                                                                                                 | `PLAN-01.md` Task 1    | Required input whenever draft contracts depend on provisional external details.              |
| split-plan validation                       | `.planning/phases/phase-0/NYQUIST-VALIDATION.md`                                                                                                                                 | `PLAN-02.md` Task 3    | Proves requirement coverage, deliverable coverage, and split-plan verification completeness. |

## Task-to-Deliverable Mapping

| Split plan/task     | Produces                                                                     | Why it matters                                                                                              |
| ------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `PLAN-01.md` Task 1 | `anthropic-contract-gap-ledger.md`                                           | Establishes what is verified, assumed, or still a gap before other drafts encode external contract details. |
| `PLAN-01.md` Task 2 | `api-contract-draft.md`, `config-shape-draft.md`                             | Defines the ingress contract and configuration surface that Phase 1 implementation will build against.      |
| `PLAN-01.md` Task 3 | `provider-adapter-interface-draft.md`, `canonical-transcript-model-draft.md` | Defines the adapter seam and transcript invariants required by R10, R13, and R14.                           |
| `PLAN-02.md` Task 1 | `manifest.yaml`, draft fixture files, `verify_phase0_fixtures.py`            | Gives R22 a concrete future golden-test baseline and encodes R9 ordering expectations.                      |
| `PLAN-02.md` Task 2 | `streaming-orchestration-risk-register.md`, `SUMMARY.md`                     | Converts protocol caveats into an explicit Phase 1 entry recommendation.                                    |
| `PLAN-02.md` Task 3 | `NYQUIST-VALIDATION.md`                                                      | Independently proves split-plan requirement coverage, deliverable coverage, and verification completeness.  |

## Goal-Backward Truths

1. A Phase 1 implementer can read project-owned drafts and know the minimum protocol surface without inferring hidden scope.
2. A Phase 1 implementer can keep provider-specific concerns behind the adapter boundary and preserve transcript semantics.
3. A tester can identify the fixture inventory and the hold/resume ordering assertions required for later golden tests.
4. A reviewer can distinguish verified contract details from provisional assumptions or gaps.
5. The project can make an explicit go, go-with-caveats, or reassess decision before runtime implementation begins.
6. The GSD checker can verify that Phase 0 execution is split into sub-threshold plans rather than one oversized plan.

## Verification Method

| Area                                     | Method                                                                                                                                              | Evidence                                                                           |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Split-plan structural validity           | Static checks over `PLAN*.md` for stale worktree paths, `<task type=` count below 5, deliverable path presence, and output/files_modified alignment | Local validation command over `.planning/phases/phase-0/PLAN*.md`                  |
| Contract certainty control               | Static token checks for `verified`, `assumed`, `gap`, and planning consequences                                                                     | `PLAN-01.md` Task 1 automated verification over `anthropic-contract-gap-ledger.md` |
| Contract and config draft completeness   | Static token checks for required headings, requirement IDs, and stream/non-stream markers                                                           | `PLAN-01.md` Task 2 automated verification                                         |
| Adapter/transcript boundary completeness | Static token checks for adapter responsibilities and canonical transcript fields                                                                    | `PLAN-01.md` Task 3 automated verification                                         |
| Fixture completeness                     | Run the fixture verifier script to confirm manifest references, requirement coverage, and stream ordering assertions                                | `python .planning/phases/phase-0/scripts/verify_phase0_fixtures.py`                |
| Risk and recommendation completeness     | Static token checks for named risks, dispositions, viability sections, and final recommendation sections                                            | `PLAN-02.md` Task 2 automated verification                                         |
| Cross-artifact traceability              | Static token checks that this file references all in-scope requirements, key deliverables, and both split plans                                     | `PLAN-02.md` Task 3 automated verification                                         |

## Known Caveats

1. `anthropic-contract-gap-ledger.md` is required because some Anthropic field/event details remain provisional in this environment.
2. FastAPI versus a dedicated SSE helper remains a Phase 0 viability question, not a locked implementation decision.
3. LiteLLM remains a candidate adapter substrate; Phase 0 may define its conformance checklist but may not claim full runtime coverage is already proven.
4. The fixture drafts and verifier script validate artifact completeness only. They do not execute a gateway runtime or provider adapter.
5. Phase 0 intentionally excludes gateway runtime code, real adapters, and SSE endpoint implementation.
6. `PLAN.md` is a routing index only; executable work is in `PLAN-01.md` and `PLAN-02.md`.

## Phase 1 Entry Conditions

Phase 1 may begin only when all of the following are true:

1. `.planning/phases/phase-0/docs/anthropic-contract-gap-ledger.md` exists and all fields/events used by draft artifacts are tagged `verified`, `assumed`, or `gap`.
2. `.planning/phases/phase-0/docs/api-contract-draft.md` and `.planning/phases/phase-0/docs/config-shape-draft.md` exist with explicit R1/R3/R9/R22 traceability.
3. `.planning/phases/phase-0/docs/provider-adapter-interface-draft.md` and `.planning/phases/phase-0/docs/canonical-transcript-model-draft.md` exist with explicit R10/R13/R14 traceability.
4. `.planning/phases/phase-0/fixtures/manifest.yaml` and the draft fixture files cover both a non-streaming case and a stream hold/resume ordering case.
5. `.planning/phases/phase-0/docs/streaming-orchestration-risk-register.md` clearly marks each risk as blocking, caveated, or accepted-for-Phase-1.
6. `.planning/phases/phase-0/SUMMARY.md` ends with a Phase 1 recommendation that matches the risk dispositions and remaining contract gaps.
7. `PLAN-01.md` and `PLAN-02.md` each remain below five executable tasks, and `PLAN.md` remains an index without executable tasks.

## Nyquist Assessment

Phase 0 passes a Nyquist-style coverage check when each critical outcome is represented by at least two independent signals:

- a split-plan task that produces the artifact, and
- an automated verification path that checks that artifact or its traceability.

The split plan structure satisfies that condition for the contract drafts, adapter/transcript drafts, fixture baseline, risk register, Summary recommendation, and validation artifact while resolving the previous single-plan scope sanity blocker.

## Outcome

The revised Phase 0 planning artifacts route execution through `PLAN-01.md` and `PLAN-02.md`. The split preserves full coverage of R1, R3, R9, R10, R13, R14, and R22; preserves the Phase 0 protocol-spike boundary; and keeps every executable plan below the GSD task-count threshold.
