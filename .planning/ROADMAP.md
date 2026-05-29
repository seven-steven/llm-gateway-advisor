# ROADMAP

Last updated: 2026-05-29

## Planning Approach

本 roadmap 采用 coarse-grained phases，但每个 phase 都要求有明确的产出物与 requirement traceability。排序原则如下：

1. 先验证协议与 streaming 风险，再收敛实现。
2. 先完成 Claude Code + Anthropic Messages 的窄路径闭环，再扩 provider 广度。
3. 先保证可测性与可观测性，再讨论大范围兼容。

## Phase 0 - Planning / Protocol Spike

### Goals

- 确认 Anthropic Messages-compatible contract 的最小实现边界。
- 确认 SSE streaming 事件模型、暂停/恢复策略与 golden fixtures。
- 定义 provider adapter interface、canonical transcript model、acceptance fixtures。
- 重新验证 FastAPI / LiteLLM / Anthropic 相关文档与可行性。

### Deliverables

- API contract 草案
- config shape 草案
- provider adapter interface 草案
- canonical transcript model 草案
- acceptance fixture 清单
- streaming/advisor orchestration 风险记录

### Requirement Traceability

- R1, R3, R9, R10, R13, R14, R22

### Exit Criteria

- 可以用 fixtures 明确描述 non-stream 与 stream 请求/响应形状。
- 可以描述 advisor orchestration 对 SSE event ordering 的约束。
- 能说明候选技术栈是否继续推进，或需要替代方案。

## Phase 1 - Anthropic Messages Gateway Skeleton

### Goals

- 实现 `POST /v1/messages` 基础入口。
- 跑通 non-streaming response shape。
- 建立 provider config、auth 基础、logging 基础与 fake provider 测试闭环。

### Deliverables

- ingress skeleton
- request/response shape normalization
- fake provider 测试替身
- 基础错误映射
- provider config 初版
- structured logging 初版

### Requirement Traceability

- R1, R2, R4, R13, R21, R23

### Exit Criteria

- non-streaming 假上游路径可稳定返回。
- fake provider 测试可在本地稳定重放。
- ingress 与 adapter boundary 已分离。

## Phase 2 - Streaming + Advisor Orchestration MVP

### Goals

- 实现 SSE pass-through 基础能力。
- 实现 advisor tool 注入、拦截、pause/hold、advisor 非流式子调用、result/history 注入与 stream 恢复。
- 通过 golden event tests 验证 streaming advisor orchestration。

### Deliverables

- SSE streaming path
- advisor state machine MVP
- advisor invocation normalization
- result injection / stream resume 逻辑
- golden event fixtures 与 tests

### Requirement Traceability

- R3, R5, R6, R7, R8, R9, R22

### Exit Criteria

- streaming 模式下可完成一次完整 advisor orchestration。
- executor stream 暂停与恢复后事件顺序仍可被客户端正确消费。
- golden tests 能覆盖关键 happy path 与主要边界。

## Phase 3 - Multi-turn Ledger + Provider Adapters

### Goals

- 引入 canonical transcript 与 multi-turn correctness 保障。
- 明确 session ledger 策略。
- 增加 usage/budget 基础能力。
- 补齐 Anthropic adapter，并在 feature flag 后提供一个 OpenAI-compatible 或 LiteLLM-backed adapter 路径。

### Deliverables

- canonical transcript implementation plan 或实现
- multi-turn history handling
- session ledger strategy
- usage/cost aggregation 基础
- secondary adapter behind feature flag

### Requirement Traceability

- R10, R11, R12, R14, R15, R19, R20

### Exit Criteria

- 多轮请求中 advisor 相关 history 可保持语义自洽。
- 至少两个 adapter 路径具备受控接入能力。
- usage/cost 数据可跨 executor/advisor 聚合。

## Phase 4 - Hardening + Claude Code Acceptance

### Goals

- 增强 auth/header allowlist、secret redaction、provider URL/model controls。
- 完善错误 taxonomy、dry-run config validation、可观测性。
- 完成 Claude Code setup 与 smoke acceptance 文档及验证路径。

### Deliverables

- auth/header allowlist
- redaction filters
- provider routing controls
- error taxonomy hardening
- dry-run config validator
- Claude Code smoke test guide

### Requirement Traceability

- R4, R16, R17, R18, R19, R20, R21, R22, R24, R25

### Exit Criteria

- 安全边界与配置验证达到本地发布前最低要求。
- Claude Code 可以按文档完成 smoke path。
- 主要错误路径具备可诊断日志与一致响应。

## Phase 5 - Post-v1 Expansion

### Goals

- 扩展更多 Agent 与 provider。
- 评估 OpenAI ingress。
- 提升 advisor schema 兼容度。
- 探索持久化存储与分发形态。

### Requirement Traceability

- 对应 v2 candidates，不计入 v1 承诺。

## Requirement-to-Phase Matrix

| Requirement | P0  | P1  | P2  | P3  | P4  | P5  |
| ----------- | --- | --- | --- | --- | --- | --- |
| R1          | x   | x   |     |     |     |     |
| R2          |     | x   |     |     |     |     |
| R3          | x   |     | x   |     |     |     |
| R4          |     | x   |     |     | x   |     |
| R5          |     |     | x   |     |     |     |
| R6          |     |     | x   |     |     |     |
| R7          |     |     | x   |     |     |     |
| R8          |     |     | x   |     |     |     |
| R9          | x   |     | x   |     |     |     |
| R10         | x   |     |     | x   |     |     |
| R11         |     |     |     | x   |     |     |
| R12         |     |     |     | x   |     |     |
| R13         | x   | x   |     |     |     |     |
| R14         | x   |     |     | x   |     |     |
| R15         |     |     |     | x   |     |     |
| R16         |     |     |     |     | x   |     |
| R17         |     |     |     |     | x   |     |
| R18         |     |     |     |     | x   |     |
| R19         |     |     |     | x   | x   |     |
| R20         |     |     |     | x   | x   |     |
| R21         |     | x   |     |     | x   |     |
| R22         | x   |     | x   |     | x   |     |
| R23         |     | x   |     |     |     |     |
| R24         |     |     |     |     | x   |     |
| R25         |     |     |     |     | x   |     |

## Dependencies and Risk Notes

- P0 是 P1-P4 的前置阶段，尤其决定 streaming 与 adapter 抽象能否按当前路线推进。
- P2 是 v1 成败关键，因为 streaming advisor orchestration 是核心验收链路。
- P3 与 P4 可以部分并行，但前提是 P2 已经稳定定义 transcript 与 event semantics。
- 若 FastAPI 或 LiteLLM spike 结果不满足需要，应在 P0 结束时调整技术栈，而不是进入 P2 后再返工。

## Non-commitments

本 roadmap 不承诺：

- 所有 provider 在 v1 都能工作。
- OpenAI ingress 会在 v1 一并完成。
- 官方 advisor beta schema 在 v1 全量兼容。
- 在未完成文档重新验证前锁定 FastAPI/LiteLLM 为最终实现栈。
