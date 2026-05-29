# PROJECT

Last updated: 2026-05-29

## What This Is

`llm-gateway-advisor` 是一个面向 Coding Agent 的本地 API gateway。它位于 Agent 与上游 LLM provider 之间，对外优先提供 Anthropic Messages API-compatible 的 `POST /v1/messages` 入口，对内负责把 executor model 与 advisor model 的调用编排为一个可用、可测试、可观测的请求闭环。

v1 的产品定位不是 generic LLM proxy，而是一个以 advisor orchestration 为中心的 local gateway：优先让 Claude Code 通过兼容 Anthropic Messages API 的方式接入，并在网关内部完成 advisor tool 注入、拦截、子调用、结果回填与流恢复。

## Core Value

1. 让 Claude Code 等 Coding Agent 在不直接依赖单一 provider 原生 advisor 能力的前提下，获得跨 provider 的 advisor 升级路径。
2. 把复杂的 provider 差异、streaming 编排、usage 统计、错误归一化与安全边界收敛到 gateway 层。
3. 用行为等价而非 schema 全量兼容的方式，优先达成可工作的 v1 验收闭环。

## Requirements Validated

以下方向已经被确认并进入有效范围：

- v1 必须支持 non-streaming 与 SSE streaming。
- v1 首发 ingress 为 Anthropic Messages API-compatible `POST /v1/messages`。
- v1 advisor tool 目标是行为等价，不追求官方 schema 完整兼容。
- streaming advisor orchestration 是核心验收路径，不是可选增强项。
- provider 调用必须经由 adapter boundary，不能把 provider 细节泄漏到 ingress/orchestrator。
- 需要 canonical transcript model，保留 roles、content block order、block type、IDs、tool_use_id、stop_reason、usage metadata。
- 需要 auth/header allowlist、secret redaction、usage/cost controls、structured observability 与 golden contract tests。
- v1 首发 Agent 范围以 Claude Code 为先。

## Active Scope

当前初始化阶段的 Active Scope：

- 明确 v1 API contract、配置形状、provider adapter interface、transcript model 与 acceptance fixtures。
- 规划 gateway-owned advisor state machine：
  - 注入行为等价 advisor tool
  - 识别完整 advisor tool invocation
  - 暂停或 hold executor stream
  - 发起 advisor 非流式子调用
  - 注入 advisor result/history
  - 恢复 executor stream
  - 保证 SSE event order
- 将 provider 能力限定为：non-streaming messages、streaming messages、capability metadata、usage extraction、error normalization。
- 以 fake provider、golden fixtures 与 Claude Code smoke path 作为早期验收主线。

## Out of Scope

以下内容不属于 v1 初始化与实施承诺：

- 通用 OpenAI ingress 兼容层。
- 官方 advisor schema 的完整逐字段兼容与所有 beta 细节复刻。
- 面向所有 Coding Agent 的一次性全面兼容；Codex/OpenCode 留到 post-v1。
- 一开始就支持所有 provider 的最优特性；v1 先走窄路径。
- 持久化 session storage、复杂多租户控制面、托管 SaaS 形态。
- 包装、安装器、桌面应用或 IDE plugin 形态。

## Context

目标用户是希望把 Claude Code 等 Coding Agent 接到多家 provider，同时保留 advisor-style 多模型协作能力的开发者或团队。现实问题在于：

- Agent 通常绑定某一入口协议，而 provider 能力与事件语义并不一致。
- 原生 advisor 能力具有 provider 绑定性，不利于多 provider 组合。
- streaming 过程中插入 advisor 子调用时，最难的是事件暂停、结果注入与恢复顺序。

因此，本项目优先解决的是“在一个 Anthropic-compatible gateway 中，以行为等价方式复现 advisor 编排”，而不是“把所有 provider 简单透传”。

## Constraints

- 技术栈暂不锁定；Python/FastAPI/LiteLLM 仅为候选，必须先完成协议与 streaming spike 验证。
- v1 必须同时支持 non-streaming 与 SSE streaming。
- advisor 子调用在当前规划中按非流式处理，以降低 orchestration 复杂度。
- 需要严格控制 header/auth 透传边界，避免 secret 泄露与未授权 provider 参数透传。
- 需要把 usage/cost 统计纳入 gateway 责任域，而不是依赖上游 provider 自行表达。
- Context7/FastAPI/LiteLLM/Anthropic 文档现阶段存在 quota 限制导致的验证缺口，相关实现前需要重新验证。

## Key Decisions

1. 产品定位：v1 是 advisor-orchestrating local gateway，不是 generic LLM proxy。
2. 首发入口：Anthropic Messages API-compatible `POST /v1/messages`。
3. 首发 Agent：Claude Code 优先，其他 Agent 后置。
4. streaming：v1 必须支持，且 advisor orchestration 的 streaming 路径为核心验收。
5. advisor 兼容目标：行为等价优先，不追求官方 schema 完整兼容。
6. provider 集成方式：所有 provider 调用必须走 adapter boundary。
7. 状态模型：需要 canonical transcript 与 gateway-owned advisor state machine。
8. 安全与治理：需要 allowlist、redaction、usage/cost controls、structured observability。

## Evolution

### 当前阶段

项目仍处于 new-project 初始化与 coarse roadmap 阶段，重点是收敛需求、边界与验收路径，而不是承诺已完成的实现。

### 下一步演化

- Phase 0：协议与 streaming spike，收敛 contract 与接口。
- Phase 1：Anthropic Messages gateway skeleton。
- Phase 2：streaming + advisor orchestration MVP。
- Phase 3：multi-turn ledger + provider adapters。
- Phase 4：hardening + Claude Code acceptance。
- Phase 5：post-v1 expansion。

### 演化原则

- 先走窄而深的 Claude Code + Anthropic Messages 闭环。
- 先验证 streaming 编排正确性，再扩展 provider 广度。
- 对无法从仓库内证实的第三方文档细节，先标记验证缺口，不预设最终实现承诺。
