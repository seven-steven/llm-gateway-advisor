# Advisor Proxy - 设计决策总结

> 日期: 2026-05-19 | 状态: GSD questioning 阶段完成，待确认 4 项剩余决策后进入 PROJECT.md

## 项目定位

在 Claude Code 和 LLM provider 之间添加一层代理，实现 **Advisor 顾问模式**的跨供应商支持。核心是本地代理服务（开源产品），不是通用协议转发。

### 核心概念: Advisor 模式

弱模型（executor）主导日常任务，遇到复杂问题时通过 tool call 请求强模型（advisor）帮助，在单次请求内完成咨询。这是 Anthropic 官方 `advisor_20260301` beta tool 的模式，我们需要在代理层本地实现。

## 已确认的设计决策

### 1. 架构优先级

- **Advisor orchestration 优先** — v1 核心是代理内实现跨供应商 Advisor，不是通用协议转发
- 目标用户: 开源产品，先做本地代理服务
- 后续考虑 Claude Code 插件形式

### 2. API 兼容

- **v1 只做 Anthropic Messages API 入口**（Claude Code 的原生协议）
- OpenAI 入口作为后续版本考量
- v1 验收闭环: Claude Code → Anthropic-compatible 代理 → executor/advisor → 返回

### 3. Advisor 触发方式

- **模型主动触发** — executor 通过 tool call 请求 advisor 帮助
- 代理拦截 `server_tool_use` (name: "advisor")，构造 advisor 上下文，调用强模型

### 4. 双模型配置

- `executor_model`: 日常执行（如 Haiku 4.5 / Sonnet 4.6）
- `advisor_model`: 复杂咨询（如 Opus 4.7）
- 两者可来自不同 provider

### 5. Auth 策略

- **白名单透传** — 配置优先，allowlist 兜底
- 未配置的上游 auth 参数，根据白名单决定是否透传

### 6. Advisor Tool API 模式（来自 Anthropic 官方文档）

```python
tools = [{
    "type": "advisor_20260301",
    "name": "advisor",
    "model": "claude-opus-4-7",
    "max_uses": 3,
    "caching": {"type": "ephemeral", "ttl": "5m"},
}]
```

- Beta header: `advisor-tool-2026-03-01`
- Executor 发出空 input 的 `server_tool_use`
- 代理拦截后构造 advisor 上下文（含完整对话历史）
- Advisor 返回 `advisor_result`（明文）或 `advisor_redacted_result`（加密）
- `usage.iterations[]` 分别报告 executor 和 advisor 的 token 用量

### 7. Streaming 行为

- advisor sub-inference 不 stream
- executor stream 暂停，advisor 结果一次性到达
- 之后 executor 继续流式输出

### 8. Multi-turn 约束

- 后续请求必须传回完整 content 含 `advisor_tool_result` blocks
- 移除 advisor tool 时需同时清理历史中的 advisor 相关内容

### 9. 技术栈（建议）

- Python + FastAPI + LiteLLM
- LiteLLM 作为 provider-agnostic 调用层

## 待确认的 4 项决策

| #   | 问题                                      | 建议方向                |
| --- | ----------------------------------------- | ----------------------- |
| 1   | v1 是否必须支持 streaming?                | 先不支持                |
| 2   | v1 是否保留 Claude Code 原始 tools?       | 保留并合并 advisor tool |
| 3   | 实现语言确认 Python + FastAPI + LiteLLM?  | 是                      |
| 4   | 是否把官方 advisor tool 兼容作为 v1 需求? | 不作为 v1 必须项        |

## Advisor 代理工作流（拟）

```
Claude Code → [POST /v1/messages]
  → 代理接收请求
  → 注入 advisor tool 到 tools 列表
  → 转发到 executor_model（如 Sonnet）
  → executor 返回 server_tool_use(name="advisor")
  → 代理拦截，构造 advisor 上下文
  → 调用 advisor_model（如 Opus）
  → 将 advisor_result 注入为 advisor_tool_result
  → 继续转发给 executor（含 advisor 回复）
  → executor 完成最终回复
  → 返回给 Claude Code
```

## 官方 Advisor Tool 参考

- 文档来源: `https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool`
- 成功响应结构:
  ```json
  {
    "role": "assistant",
    "content": [
      { "type": "text", "text": "Let me consult..." },
      {
        "type": "server_tool_use",
        "id": "srvtoolu_abc123",
        "name": "advisor",
        "input": {}
      },
      {
        "type": "advisor_tool_result",
        "tool_use_id": "srvtoolu_abc123",
        "content": {
          "type": "advisor_result",
          "text": "Use a channel-based pattern..."
        }
      },
      { "type": "text", "text": "Here's the implementation..." }
    ]
  }
  ```
- Error codes: `max_uses_exceeded`, `too_many_requests`, `overloaded`, `prompt_too_long`, `execution_time_exceeded`, `unavailable`
- Valid model pairs: Haiku 4.5 / Sonnet 4.6 / Opus 4.6 / Opus 4.7 → Opus 4.7

## 目标上游 Provider

- 智谱 (GLM)
- MiniMax
- Qwen (通义千问)
- NewAPI
- Anthropic（直连）
- 其他 OpenAI-compatible provider

## GSD 流程状态

- [x] Round 1 questioning
- [x] Round 2 questioning
- [x] Round 3 questioning
- [x] Decision gate → "Keep exploring"
- [x] 需求/方案总结
- [x] 官方 Advisor 文档调研
- [x] 设计决策总结
- [ ] 确认 4 项剩余决策
- [ ] Write PROJECT.md
- [ ] Collect workflow preferences (config.json)
- [ ] Define REQUIREMENTS.md
- [ ] Create ROADMAP.md
- [ ] Generate CLAUDE.md
