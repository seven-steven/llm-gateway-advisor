# STATE

Last updated: 2026-05-29

## Current Stage

项目处于初始化规划阶段，已完成目标、范围、关键约束与 coarse roadmap 收敛；尚未进入正式实现阶段。

## Current Snapshot

- 项目定位已明确：advisor-orchestrating local gateway，而非 generic LLM proxy。
- v1 首发 ingress 已明确：Anthropic Messages API-compatible `POST /v1/messages`。
- v1 首发 Agent 已明确：Claude Code 优先。
- v1 必须支持 non-streaming 与 SSE streaming，且 streaming advisor orchestration 是核心验收路径。
- advisor 兼容目标已明确：行为等价优先，不追求官方 schema 完整兼容。
- provider integration 原则已明确：必须经由 adapter boundary。
- transcript/state 原则已明确：需要 canonical transcript 与 gateway-owned advisor state machine。

## Completed In This Init

- 建立规划配置基线：`.planning/config.json`
- 收敛项目定位与边界：`.planning/PROJECT.md`
- 定义 v1 requirement 集：`.planning/REQUIREMENTS.md`
- 定义 coarse roadmap 与 requirement traceability：`.planning/ROADMAP.md`
- 汇总研究结论与验证缺口：`.planning/research/SUMMARY.md`

## Active Risks

1. streaming advisor orchestration 的暂停/恢复与 SSE event ordering 是最高风险技术点。
2. FastAPI 是否适合作为目标 ingress 层，仍需通过 spike 重新验证。
3. LiteLLM 是否足以承载 adapter 边界所需能力，仍需通过 spike 重新验证。
4. Anthropic advisor/messages 文档细节当前存在 quota 验证缺口，实施前需补验证。
5. session ledger 的边界如果在早期定义不清，后续 multi-turn correctness 容易返工。

## Assumptions

- v1 中 advisor 子调用按非流式实现，可以降低编排复杂度并聚焦主要风险。
- fake provider 与 golden fixtures 能覆盖大部分协议与事件顺序验证需求。
- 先服务 Claude Code 的兼容闭环，比一开始追求多 Agent 广覆盖更有价值。

## Next Checkpoints

### Checkpoint A - Protocol Spike Ready

完成以下事项后可认为进入 P0 可执行状态：

- 明确 request/response fixture
- 明确 SSE event fixture
- 明确 provider adapter interface 草案
- 明确 transcript model 草案

### Checkpoint B - Tech Validation

完成以下事项后才建议锁定实现栈：

- 重新验证 FastAPI 相关文档与最小实现方式
- 重新验证 LiteLLM 相关文档与能力边界
- 重新验证 Anthropic Messages / advisor 文档

### Checkpoint C - Implementation Start

满足以下条件后再进入编码：

- P0 退出条件满足
- 假上游测试策略已确定
- acceptance fixtures 已准备

## Blockers

当前没有阻断初始化文档产出的 blocker，但存在以下实现前 blocker：

- 文档验证 quota 恢复前，不能把第三方文档细节当成最终事实锁定。
- streaming 事件模型未 spike 前，不应开始实现最终 advisor state machine。

## Decision Log

- 2026-05-29: 确认 v1 必须支持 streaming。
- 2026-05-29: 确认 v1 advisor tool 追求行为等价，不追求官方 schema 完整兼容。
- 2026-05-29: 确认技术栈暂不锁定，Python/FastAPI/LiteLLM 仅作为候选。
- 2026-05-29: 确认 v1 首发 Agent 范围以 Claude Code 为优先。
