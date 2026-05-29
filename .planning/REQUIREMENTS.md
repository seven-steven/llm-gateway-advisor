# REQUIREMENTS

Last updated: 2026-05-29

## Defined

本文件定义 `llm-gateway-advisor` 的初始化阶段需求基线，用于约束 v1 范围、验收路径与 roadmap traceability。所有 requirement 应满足三个原则：

- 原子：每条 requirement 应能独立验证。
- 可测试：必须能映射到 contract test、golden test、smoke test 或配置验证。
- 不过度承诺：只描述当前确认的目标，不把候选技术或未来扩展写成既定事实。

## Core Value

项目的核心价值是：为 Claude Code 等 Coding Agent 提供一个本地、可控、可观测的 advisor-orchestrating gateway，使 executor/advisor 可跨 provider 协同工作，同时保持 Anthropic Messages-compatible 的接入方式与稳定的 streaming 行为。

## v1 Requirements

### API Compatibility

- R1. 网关必须提供 Anthropic Messages API-compatible 的 `POST /v1/messages` 入口。
- R2. 网关必须支持 `stream=false` 的 non-streaming 请求路径。
- R3. 网关必须支持 SSE streaming 请求路径，并保持可消费的事件序列。
- R4. 网关必须把上游 provider/内部错误映射为一致的 API error shape。

### Advisor Orchestration

- R5. 网关必须能在请求进入 executor 前注入行为等价的 advisor tool 定义。
- R6. 网关必须识别完整的 advisor tool invocation，并把该调用切换到 gateway-owned orchestration。
- R7. 在 streaming 模式下，网关必须支持暂停或 hold executor stream，完成 advisor 子调用后再恢复。
- R8. advisor 子调用在 v1 中必须支持非流式执行，并将结果按 transcript 语义回填。
- R9. 恢复后的 executor stream 必须保持正确的 SSE event order。

### Transcript / State

- R10. 网关必须维护 canonical transcript model，保留 roles、content block order、block type、IDs、tool_use_id、stop_reason、usage metadata。
- R11. 网关必须保证 multi-turn 请求中的 advisor 相关历史在语义上自洽，避免 tool/result 脱节。
- R12. v1 必须明确 session ledger 策略；如暂不持久化，至少要定义进程内行为与边界。

### Provider Routing

- R13. 所有 provider 调用必须经过 adapter boundary，禁止在 ingress/orchestrator 直接耦合 provider 特定实现。
- R14. adapter v1 必须覆盖：non-streaming messages、streaming messages、capability metadata、usage extraction、error normalization。
- R15. v1 必须至少支持一个 Anthropic adapter 和一个受 feature flag 保护的 OpenAI-compatible 或 LiteLLM-backed adapter 路径。

### Policy / Security

- R16. 网关必须支持 auth/header allowlist 机制。
- R17. 网关必须提供 secret redaction，避免日志与错误输出泄露敏感值。
- R18. 网关必须限制可访问的 provider base URL、model 或相关路由目标，避免任意上游穿透。

### Usage / Observability

- R19. 网关必须支持 usage limit 或 budget control 的最小闭环。
- R20. 网关必须聚合 executor 与 advisor 的 usage/cost metadata。
- R21. 网关必须输出 structured observability 数据，至少覆盖 request、provider call、advisor orchestration、error 维度。

### Testing / Developer Experience

- R22. 项目必须提供 golden contract tests，覆盖 non-streaming 与 streaming 的关键协议行为。
- R23. 项目必须提供 fake provider 或等价测试替身，以支持稳定的本地验证。
- R24. 项目必须提供 dry-run config validation，提前发现配置错误。
- R25. 项目必须提供面向 Claude Code 的 setup / smoke test 路径。

## v2 Candidates

以下是明确后置到 v2 或 post-v1 的候选需求：

- V2-1. OpenAI ingress 兼容层。
- V2-2. Codex/OpenCode 等更多 Agent 的首方验收路径。
- V2-3. 官方 advisor schema 的更高兼容度或 exact-schema mode。
- V2-4. 持久化 session ledger / transcript storage。
- V2-5. 更丰富的 provider 矩阵与 provider-specific optimization。
- V2-6. 打包、发行、安装器与更完整的本地开发体验封装。

## Out of Scope

以下内容当前不属于 v1：

- generic LLM proxy 的全协议支持。
- 对所有 provider 的全部参数透传与无约束自由路由。
- 一开始就做 hosted control plane、多租户后台或复杂计费系统。
- UI 产品、桌面端壳或 IDE plugin。
- 对第三方 beta 行为的完整逐项复刻承诺。

## Traceability

| Requirement | Summary                                       | Planned Phase |
| ----------- | --------------------------------------------- | ------------- |
| R1          | Anthropic Messages ingress                    | P0, P1        |
| R2          | non-streaming path                            | P1            |
| R3          | SSE streaming path                            | P0, P2        |
| R4          | error mapping                                 | P1, P4        |
| R5          | advisor tool injection                        | P2            |
| R6          | advisor invocation interception               | P2            |
| R7          | streaming pause/hold/resume                   | P2            |
| R8          | non-stream advisor subcall + result injection | P2            |
| R9          | SSE event order correctness                   | P0, P2        |
| R10         | canonical transcript model                    | P0, P3        |
| R11         | multi-turn advisor history correctness        | P3            |
| R12         | session ledger strategy                       | P3            |
| R13         | provider adapter boundary                     | P0, P1        |
| R14         | adapter capability set                        | P0, P3        |
| R15         | Anthropic + secondary adapter path            | P3            |
| R16         | auth/header allowlist                         | P4            |
| R17         | secret redaction                              | P4            |
| R18         | provider URL/model controls                   | P4            |
| R19         | usage limit / budget control                  | P3, P4        |
| R20         | usage/cost aggregation                        | P3, P4        |
| R21         | structured observability                      | P1, P4        |
| R22         | golden contract tests                         | P0, P2, P4    |
| R23         | fake provider / test double                   | P1            |
| R24         | dry-run config validation                     | P4            |
| R25         | Claude Code smoke path                        | P4            |

## Coverage

### 已覆盖到 roadmap 的 v1 需求

R1-R25 均已映射到至少一个 roadmap phase，没有孤立 requirement。

### 初始化阶段重点验收

初始化完成后，优先验证以下 requirement 是否已被 roadmap 正确收敛：

- streaming 核心链路：R3、R7、R8、R9
- adapter / transcript 基础：R10、R13、R14
- 安全与治理底线：R16、R17、R18、R19、R20、R21
- 开发可验证性：R22、R23、R24、R25

### 当前验证缺口

- FastAPI 对目标 SSE 事件控制方式尚未重新验证。
- LiteLLM 对目标 adapter 能力覆盖度尚未重新验证。
- Anthropic 官方 advisor / messages 文档细节尚需在 quota 恢复后重新验证。
