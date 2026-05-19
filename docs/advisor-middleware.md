# advisor-middleware 项目分析

本文基于用户提供的公开仓库 `https://github.com/emanueleielo/advisor-middleware` 的源码快照编写。当前本地工作区本身没有该项目源码，分析事实来自临时拉取的上游仓库文件，包括 `README.md`、`pyproject.toml`、`advisor_middleware/`、`examples/` 和 `tests/`。

## 1. 项目作用

`advisor-middleware` 是一个 Python 包，目标是把 Anthropic 的 Advisor Strategy 封装成可直接插入 DeepAgents 的 middleware。

Advisor Strategy 的核心思想是：

- 主流程由较快、较便宜的 executor 模型执行，例如 Sonnet、Haiku 或其它 provider 的模型。
- executor 在普通读写、搜索、工具调用、明确下一步操作时独立工作。
- 只有当 executor 遇到架构取舍、复杂调试、跨文件推理、多步策略规划或不确定决策时，才调用更强的 advisor 模型，例如 Opus。
- advisor 不直接执行工具，也不产出面向用户的最终回答，而是返回计划、纠偏建议、风险提示或停止信号。

因此，该项目解决的问题不是“再创建一个子代理框架”，而是把强模型从“每一轮都参与”变成“关键决策时按需参与”。README 将其定位为 DeepAgents 的单一 import：负责 provider 检测、Anthropic 原生 advisor tool 注入、跨 provider fallback 调用、调用次数限制、上下文裁剪和 advisor 使用情况 introspection。

`pyproject.toml` 中的包元数据也支持这个定位：包名为 `advisor-middleware`，描述为 “Anthropic's Advisor Strategy as a drop-in DeepAgents middleware — pair a powerful advisor with a fast executor”，Python 版本要求为 `>=3.11`，项目状态分类为 Alpha。

## 2. 适用场景

该 middleware 适合以下场景：

- 希望用较低成本模型跑大部分 agent 工作，但在关键点借助更强模型提升质量。
- 编码 agent 会遇到跨文件调试、架构设计、安全权衡或迁移计划等复杂任务。
- 使用 DeepAgents，并希望 middleware 以组合方式接入，而不是重写 agent 调度逻辑。
- 希望限制高成本 advisor 的使用次数，避免强模型在每轮都消耗 token。
- executor 和 advisor 可能来自不同 provider，需要非 Anthropic executor 也能咨询 advisor。

不适合的场景：

- 每一步都必须由最强模型直接执行的任务。
- 完全不使用 LangChain / DeepAgents 生态的应用，除非愿意手动维护消息上下文并使用 fallback tool。
- 需要已经稳定生产化的库；该项目在 `pyproject.toml` 中标记为 `Development Status :: 3 - Alpha`。

## 3. 包结构与关键文件

上游仓库的核心目录结构如下：

```text
advisor_middleware/
├── __init__.py        # 公共 API 导出
├── config.py          # AdvisorConfig 与 ContextCurationConfig
├── middleware.py      # AdvisorMiddleware 核心实现
├── providers.py       # provider 检测、原生 tool spec、fallback 调用
├── prompts.py         # executor 注入提示词与 fallback advisor system prompt
├── state.py           # AdvisorEvent 与 AdvisorState 类型
└── py.typed           # PEP 561 类型标记

examples/
├── basic_usage.py     # 基础使用、跨 provider、与 compaction middleware 组合示例
└── benchmark.py       # Haiku solo vs Haiku + Opus Advisor 的 benchmark 示例

tests/
├── test_config.py
├── test_middleware.py
└── test_providers.py
```

公共 API 由 `advisor_middleware/__init__.py` 导出：

- `AdvisorMiddleware`
- `AdvisorConfig`
- `ContextCurationConfig`
- `AdvisorState`
- `AdvisorEvent`
- `is_anthropic_model`

## 4. 核心实现原理

### 4.1 AdvisorMiddleware 是 DeepAgents middleware

`advisor_middleware/middleware.py` 定义 `AdvisorMiddleware`，继承自 `langchain.agents.middleware.types.AgentMiddleware`。它通过 `wrap_model_call` 和 `awrap_model_call` 拦截每次模型调用，在请求进入 executor 模型前注入 advisor 能力。

核心流程是：

1. 取得当前 thread ID。
2. 根据 per-thread 状态计算本 turn 还剩多少 advisor 调用额度。
3. 判断 executor 是否为 Anthropic provider。
4. 如果可以使用 native 模式，则移除 fallback `advisor` tool，并注入 Anthropic `advisor_20260301` server-side tool spec。
5. 如果不能使用 native 模式，则保留 fallback `advisor` tool；额度耗尽时移除该 tool。
6. 如果仍有 advisor 调用额度，则向 executor 的 system message 追加一段提示，告诉 executor 何时该调用 advisor。
7. 调用原始 handler 执行模型请求。
8. 请求结束后重置本 turn 的 advisor 使用次数，但保留 session 总次数。

源码注释特别说明：该实现把 per-turn 状态存储在 middleware 实例上，并按 thread ID 分隔，而不是依赖 LangGraph state update。注释给出的原因是 LangChain 1.2.x 会忽略 middleware 返回的 `Command(update={...})`。

### 4.2 Native mode

当满足以下条件时使用 native mode：

- `AdvisorConfig.prefer_native=True`。
- executor model 被检测为 Anthropic provider。

检测逻辑位于 `advisor_middleware/providers.py` 的 `is_anthropic_model(model)`：它调用模型对象的 `_get_ls_params()`，并检查返回 dict 中的 `ls_provider` 是否等于 `anthropic`。

如果进入 native mode，middleware 会：

- 从请求工具列表里移除 fallback `advisor` tool。
- 调用 `build_native_advisor_tool_spec(config)` 生成 Anthropic 原生 tool spec。
- 注入形如 `type=advisor_20260301`、`name=advisor`、`model=<advisor_model>`、`max_uses=<remaining>`、`max_tokens=<config.max_tokens>` 的 tool dict。

README 和源码注释声称这种方式由 Anthropic API 在 server-side 处理 advisor 路由，简单 turn 没有额外 round-trip。本次分析没有联网验证 `advisor_20260301` 的当前公开 API schema，因此这里按仓库源码与 README 记载呈现。

### 4.3 Fallback mode

以下情况使用 fallback mode：

- executor 不是 Anthropic provider。
- 或者显式设置 `AdvisorConfig(prefer_native=False)`。

fallback mode 下，`AdvisorMiddleware` 在 `self.tools` 中暴露一个 `advisor` StructuredTool。该 tool 的输入 schema 是 `AdvisorToolSchema`，只有一个字段：

- `question: str`：executor 要咨询 advisor 的具体问题或决策点。

当 executor 调用 fallback `advisor` tool 时，middleware 会：

1. 检查本 turn 和 session 是否还有 advisor 调用额度。
2. 通过 `_resolve_advisor()` 懒加载 advisor 模型。
3. 读取当前对话上下文。
4. 调用 `invoke_advisor_fallback()` 或 `ainvoke_advisor_fallback()`。
5. 记录 advisor 使用次数、token 数和事件。
6. 把 advisor 返回的建议文本以及本 turn 剩余额度返回给 executor。

如果调用失败，fallback tool 会返回失败说明，并提示 executor 继续根据自身判断推进。

### 4.4 上下文裁剪

fallback 调用会通过 `providers.py` 中的 `_curate_context()` 构造发给 advisor 的消息列表。裁剪策略由 `ContextCurationConfig` 控制：

- `include_system_prompt`：是否把 executor 的 system prompt 作为上下文发给 advisor。
- `include_tool_results`：是否包含 tool result 消息。
- `max_context_messages`：限制传给 advisor 的历史消息数量。
- `max_context_chars`：限制传给 advisor 的上下文字数预算。

默认情况下会包含 executor system prompt 和 tool results，不限制消息数与字符数。若设置了消息数量或字符预算，代码会保留较新的消息，避免 fallback advisor 调用成本不可控。

### 4.5 提示词注入

`advisor_middleware/prompts.py` 包含两类提示词：

- `EXECUTOR_ADVISOR_PROMPT`：追加到 executor 的 system message 中，说明 advisor tool 的角色、适合调用的场景、不适合调用的场景，以及本 turn 剩余调用次数。
- `ADVISOR_SYSTEM_PROMPT`：fallback mode 下 advisor 自己使用的 system prompt，要求 advisor 给出简洁可执行的战略建议，不调用工具，不生成面向用户的最终输出。

executor prompt 中列出的适合咨询 advisor 的情况包括：架构决策、复杂调试、权衡评估、多步规划、卡住或不确定最佳路径。明确不建议为 routine tool calls、简单明确操作、下一步显而易见的任务调用 advisor。

### 4.6 状态与 introspection

`advisor_middleware/state.py` 定义了：

- `AdvisorEvent`：记录单次 advisor 咨询，包括 turn、question、advice、advisor_tokens、strategy。
- `AdvisorState`：扩展 `AgentState`，用 `PrivateStateAttr` 标记 advisor 相关私有字段，避免泄露到父图。

`AdvisorMiddleware` 实际维护了一个按 thread ID 分隔的 `_ThreadAdvisorState`，包含：

- `turn_uses`：当前 turn 已用 advisor 次数。
- `total_uses`：session 内累计 advisor 次数。
- `events`：advisor 咨询历史。
- `turn_counter`：turn 计数器。

对外提供的 introspection 方法包括：

- `get_events(thread_id=None)`
- `get_total_uses(thread_id=None)`
- `get_total_advisor_tokens(thread_id=None)`

这些方法可用于观察 advisor 是否被调用、使用的是 native 还是 fallback，以及 advisor 消耗的 token 数。

## 5. 配置项

### 5.1 AdvisorConfig

`AdvisorConfig` 是主配置 dataclass，定义在 `advisor_middleware/config.py`。

| 参数                    |                    默认值 | 作用                                                               |
| ----------------------- | ------------------------: | ------------------------------------------------------------------ |
| `advisor_model`         |         `claude-opus-4-6` | advisor 模型 ID 或已解析的 `BaseChatModel` 实例                    |
| `max_uses_per_turn`     |                       `3` | 单个 agent turn 内最多咨询 advisor 的次数                          |
| `max_uses_per_session`  |                    `None` | 整个 session 内 advisor 咨询上限；`None` 表示不限制                |
| `prefer_native`         |                    `True` | executor 是 Anthropic 时优先使用原生 `advisor_20260301` tool       |
| `context`               | `ContextCurationConfig()` | fallback mode 下传给 advisor 的上下文裁剪策略                      |
| `max_tokens`            |                    `1024` | 单次 advisor 响应最大 token 数                                     |
| `advisor_system_prompt` |                    `None` | fallback mode 下覆盖默认 advisor system prompt；native mode 下无效 |
| `temperature`           |                     `1.0` | fallback mode 下 advisor 模型调用温度                              |

`AdvisorConfig.__post_init__()` 会在 `context is None` 时自动创建默认的 `ContextCurationConfig`。

### 5.2 ContextCurationConfig

| 参数                    | 默认值 | 作用                                                    |
| ----------------------- | -----: | ------------------------------------------------------- |
| `include_system_prompt` | `True` | 是否把 executor 的 system prompt 作为上下文传给 advisor |
| `include_tool_results`  | `True` | 是否把 tool result 消息传给 advisor                     |
| `max_context_messages`  | `None` | fallback advisor 可见的最大历史消息数                   |
| `max_context_chars`     | `None` | fallback advisor 可见的最大上下文字数                   |

## 6. 如何使用

### 6.1 安装

README 给出的安装方式：

```bash
pip install advisor-middleware
```

或直接从 GitHub 安装：

```bash
pip install git+https://github.com/emanueleielo/advisor-middleware.git
```

如果从源码开发，需要安装开发与可选依赖：

```bash
pip install -e ".[dev,deepagents,anthropic]"
```

### 6.2 Anthropic executor + Opus advisor

`examples/basic_usage.py` 中的 `native_anthropic()` 展示了原生模式：

```python
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend
from advisor_middleware import AdvisorMiddleware

backend = FilesystemBackend(root_dir="/data/workspace")
mw = AdvisorMiddleware(advisor_model="claude-opus-4-6")

agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    system_prompt="You are a senior software engineer.",
    backend=backend,
    middleware=[mw],
)

result = agent.invoke({"messages": [("human", "Redesign the authentication module for better security")]})
print(result)
print(mw.get_total_uses())
print(mw.get_total_advisor_tokens())
```

该配置下，middleware 会检测 Anthropic executor，并尝试注入 `advisor_20260301` 原生 tool spec。

### 6.3 跨 provider fallback

`examples/basic_usage.py` 中的 `cross_provider()` 展示了 fallback 模式：

```python
from advisor_middleware import AdvisorConfig, AdvisorMiddleware, ContextCurationConfig

mw = AdvisorMiddleware(
    config=AdvisorConfig(
        advisor_model="anthropic:claude-opus-4-6",
        prefer_native=False,
        max_uses_per_turn=2,
        temperature=0.7,
        context=ContextCurationConfig(max_context_messages=15),
    ),
)

agent = create_deep_agent(
    model="openai:gpt-4o",
    system_prompt="You are a coding assistant.",
    backend=backend,
    middleware=[mw],
)
```

这里 executor 是 OpenAI 模型，advisor 是 Anthropic 模型。由于 `prefer_native=False`，middleware 会保留 fallback `advisor` tool，并在 executor 调用它时直接调用 advisor model。

### 6.4 与其它 middleware 组合

`examples/basic_usage.py` 中的 `with_compact_middleware()` 展示了与 `compact_middleware` 组合使用：

```python
from advisor_middleware import AdvisorMiddleware
from compact_middleware import CompactionMiddleware, CompactionToolMiddleware

advisor_mw = AdvisorMiddleware(advisor_model="claude-opus-4-6")
compact_mw = CompactionMiddleware(model="anthropic:claude-sonnet-4-6", backend=backend)
compact_tool_mw = CompactionToolMiddleware(compact_mw)

agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    backend=backend,
    middleware=[advisor_mw, compact_mw, compact_tool_mw],
)
```

这个示例体现了项目设计目标：advisor-middleware 不是替代 DeepAgents 或其它 middleware，而是作为可组合能力加入 middleware 列表。

### 6.5 成本控制配置示例

```python
from advisor_middleware import AdvisorConfig, ContextCurationConfig

config = AdvisorConfig(
    advisor_model="anthropic:claude-opus-4-6",
    max_uses_per_turn=2,
    max_uses_per_session=10,
    context=ContextCurationConfig(
        max_context_messages=10,
        include_tool_results=False,
    ),
)
```

这个配置会限制每 turn 最多 2 次 advisor 咨询、整个 session 最多 10 次，并在 fallback mode 下只把最近 10 条消息传给 advisor，同时跳过体积较大的 tool results。

## 7. 开发、测试与质量工具

`pyproject.toml` 声明的构建系统是 Hatchling：

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

核心依赖包括：

- `langchain>=1.2`
- `langchain-core>=0.3`
- `langgraph>=0.4`
- `pydantic>=2`

可选依赖包括：

- `deepagents>=0.1`
- `langchain-anthropic>=1.4`
- 开发依赖：`pytest>=8`、`pytest-asyncio>=0.24`、`ruff>=0.8`、`mypy>=1.13`

README 给出的开发命令：

```bash
pip install -e ".[dev,deepagents,anthropic]"
pytest
ruff check advisor_middleware/
mypy advisor_middleware/
```

`pyproject.toml` 中的质量配置：

- Ruff target version：`py311`
- Ruff line length：`120`
- Ruff lint select：`E`, `F`, `I`, `W`, `UP`, `B`, `SIM`
- Mypy：`strict = true`
- Pytest testpaths：`tests`

测试覆盖范围包括：

- `tests/test_config.py`：默认配置、自定义配置、context 默认初始化。
- `tests/test_middleware.py`：middleware 初始化、thread state、advisor 使用次数、events、native spec。
- `tests/test_providers.py`：Anthropic provider 检测、native tool spec、fallback 上下文裁剪。

本次分析曾尝试在临时仓库运行测试，但环境缺少 `langchain`，测试收集阶段失败，错误类型是 `ModuleNotFoundError: No module named 'langchain'`。因此本文不能声称测试在当前环境通过；只能说明仓库提供了上述测试与开发命令。

## 8. Benchmark 与 README 声称

`examples/benchmark.py` 构造了一个带连接池、circuit breaker、rate limit、retry 逻辑的异步任务队列调试任务，用来比较 Haiku 单独执行与 Haiku + Opus Advisor。

该脚本需要设置 `ANTHROPIC_API_KEY`，并会真实调用 Anthropic 模型。脚本注释中的 usage 写的是 `python examples/benchmark_async.py`，但仓库中实际文件名是 `examples/benchmark.py`；README 中的运行命令是：

```bash
python examples/benchmark.py
```

README 声称的 benchmark 结果包括：

- Haiku solo：11/12 tests passing。
- Haiku + Opus Advisor：12/12 tests passing。
- Advisor calls：1。
- Duration：从 210.7s 降到 90.3s。

这些 benchmark 结果来自 README，未在本次环境复现，因为运行需要依赖安装和有效的 Anthropic API key。

## 9. 限制与注意事项

- 项目成熟度：`pyproject.toml` 标记为 Alpha，适合评估和实验，生产使用前应自行验证。
- Native tool schema：源码生成 `advisor_20260301` tool spec，但本次没有联网核验 Anthropic 当前 API 文档。
- fallback 上下文：fallback mode 下 advisor 能看到的上下文取决于 middleware 能否拿到 messages，以及 `ContextCurationConfig` 的裁剪策略。
- standalone 使用：`AdvisorMiddleware.set_messages(messages, system_message=None)` 可以手动设置 fallback advisor 可见的上下文，适合非 DeepAgents 的手写 agent loop。
- token 统计：`get_total_advisor_tokens()` 依赖响应中的 `usage_metadata`；如果模型响应没有 usage metadata，源码会把 token 数记为 0。
- benchmark 不等于通用保证：README 的结果是特定调试任务上的示例，不能直接推导到所有任务。
- 示例与文件名存在小不一致：`examples/benchmark.py` 注释写 `python examples/benchmark_async.py`，但仓库实际文件是 `examples/benchmark.py`。

## 10. 快速判断

一句话概括：`advisor-middleware` 是一个面向 DeepAgents 的成本/质量折中 middleware，让较便宜的 executor 模型负责执行，让更强的 advisor 模型只在关键决策点提供战略建议。

如果你的 agent 任务以常规执行为主、偶尔需要高阶判断，这种设计值得尝试；如果每一步都需要强推理，或者不能接受 Alpha 阶段依赖，则需要谨慎评估。
