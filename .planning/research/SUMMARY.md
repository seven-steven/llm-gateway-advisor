# Research Summary

Last updated: 2026-05-29

## Executive Summary

本次研究已足以支持项目初始化，但结论应被视为“用于规划的工作基线”，而不是“第三方文档已全部复核后的最终实现事实”。当前最重要的结论是：v1 应聚焦为一个 advisor-orchestrating local gateway，优先完成 Claude Code -> Anthropic Messages-compatible ingress -> executor/advisor orchestration -> provider adapter 的窄路径闭环。

## Consolidated Findings

### 1. Product Positioning

- v1 不是 generic LLM proxy。
- v1 的中心能力是 gateway-owned advisor orchestration。
- 首发目标用户是需要在本地把 Coding Agent 接到多 provider，同时保留 advisor 升级路径的开发者。

### 2. Ingress and Compatibility

- 首发 ingress 应限定为 Anthropic Messages API-compatible `POST /v1/messages`。
- v1 必须同时覆盖 non-streaming 与 SSE streaming。
- streaming advisor orchestration 不是增强项，而是核心验收路径。

### 3. Advisor Orchestration Model

建议采用 gateway-owned state machine，而不是依赖上游 provider 自动完成：

1. 在进入 executor 前注入行为等价 advisor tool。
2. 识别完整 advisor tool invocation。
3. 暂停或 hold executor stream。
4. 发起 advisor 非流式子调用。
5. 将 advisor result/history 注入 transcript。
6. 恢复 executor stream。
7. 保证 SSE event order 对客户端可消费。

### 4. Provider Architecture

provider 调用必须经过 adapter boundary，v1 需要的最小能力集合包括：

- non-streaming messages
- streaming messages
- capability metadata
- usage extraction
- error normalization

这意味着 ingress、orchestrator 与 provider-specific transport 需要解耦；任何 provider 细节都不应直接扩散到请求入口层。

### 5. Transcript and State

为了正确支持 advisor orchestration 与 multi-turn，必须有 canonical transcript model，至少保留：

- role
- content block order
- block type
- IDs
- tool_use_id
- stop_reason
- usage metadata

如果缺少这一层，streaming 中间插入 advisor result 以及多轮历史重放都容易出现语义错位。

### 6. Security and Governance

研究阶段已经明确以下要求不是“后面再说”的加分项，而是 v1 最低边界：

- auth/header allowlist
- secret redaction
- provider URL/model controls
- usage/cost controls
- structured observability

### 7. Testing Strategy

推荐从一开始就用 golden contract tests 驱动关键协议行为验证，重点覆盖：

- non-streaming response shape
- streaming event sequence
- advisor tool injection/interception
- pause/hold/resume ordering
- error normalization
- Claude Code smoke path

## Recommended v1 Shape

- ingress: Anthropic Messages-compatible
- primary agent: Claude Code
- advisor compatibility target: behavioral equivalence
- advisor subcall mode: non-streaming
- adapter strategy: Anthropic first, one secondary path behind feature flag
- state strategy: canonical transcript + explicit session ledger decision

## Open Questions

1. FastAPI 是否适合承载目标 SSE 事件控制需求？
2. LiteLLM 是否足以覆盖目标 adapter 能力集合？
3. session ledger 在 v1 是纯内存策略，还是先定义可插拔接口？
4. Claude Code 的实际 smoke acceptance 需要哪些最小配置与 headers？

## Verification Gaps

以下内容目前应标记为“待重新验证”，不能直接视为最终实现事实：

- FastAPI 相关实现细节
- LiteLLM 相关实现细节
- Anthropic Messages / advisor 官方文档细节

原因：Context7 对上述文档的本轮验证受 quota 限制，尚未完成重新核验。

## Recommended Immediate Next Step

进入 Phase 0 protocol/streaming spike，并以以下产出物作为直接目标：

- request/response fixtures
- SSE event fixtures
- provider adapter interface 草案
- canonical transcript model 草案
- fake provider 测试策略
