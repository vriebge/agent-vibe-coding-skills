# Finance Agent Web Coding 示范总提示词

这是一份一次性给 AI Coding Agent 的总控提示词。

注意：一次性给提示词，不等于让 AI 一口气生成完整项目。正确做法是：一次性设定工程流程，然后让 Agent 按阶段推进。每一阶段有交付物，有闸门，没过闸门不能进入下一步。

你要构建的是一个**教学级可运行 Finance Agent 原型**，用来给学员示范：在 Codex、Claude Code 这类 AI coding 环境里，如何借助 Skill、Superpowers、worktree、TDD、Spec-first 和评测，把 Web 项目做成一个真正的 Agent 工程，而不是让 AI 直接糊一堆代码。

如果你当前环境没有安装下面这些能力，先停下来提示用户安装，不要假装已经加载：

- `pi-coding-agent` skill：用于参考 Pi 的工具注册、Provider 抽象、JSONL session、RPC、Skill / Prompt 机制。
- `craft-agent-template` skill：用于参考 Craft Agents 的 Bun workspace、Electron / React、Jotai 会话隔离、WebSocket RPC、MCP、AppShell 分层。
- Superpowers 插件：至少使用 `writing-plans`、`using-git-worktrees`、`test-driven-development`、`subagent-driven-development`、`verification-before-completion`。

这两个自定义 skill 从这里下载：

https://github.com/helsome/agent-vibe-coding-skills

Codex 可用下面的方式安装：

```bash
git clone https://github.com/helsome/agent-vibe-coding-skills.git
mkdir -p ~/.codex/skills
cp -R agent-vibe-coding-skills/skills/pi-coding-agent ~/.codex/skills/
cp -R agent-vibe-coding-skills/skills/craft-agent-template ~/.codex/skills/
```

Claude Code 或其他 Agent 环境，把仓库里 `skills/pi-coding-agent` 和 `skills/craft-agent-template` 两个目录复制到对应的 skills 目录即可。

---

## 第一段：先搭 Agent Harness 底座，不碰业务花活

第一步不考虑金融业务细节，只把一个最小 Agent Harness 跑起来。

你要先写 Spec，不要先写代码。Spec 里至少说明：

- 这个项目是教学级 Finance Agent 原型，不是面向真实金融业务的终端。
- 目标用户是学习 Agent Web Coding 的学员。
- 初版只做 Web 原型；如果项目已有 Electron / Bun workspace 基础，再参考 Craft Agents 扩展。
- 不做真实交易、不接支付、不做万能助手、不重造 Claude Code。
- 必须有 Agent Loop、工具调用、会话状态、上下文管理、评测集和浏览器验证。

底座只保留这些能力：

- `engine`：跑 ReAct 循环，用户输入 -> 模型回复 -> 工具调用 -> 工具结果 -> 终止判断。
- `provider`：做一个可替换 LLM Provider 接口，不扩展供应商矩阵。
- `tools`：做工具注册、schema 校验、工具执行、输出上限。
- `session`：保存 messages、tool trace、run state，支持恢复。
- `context`：维护 recent symbols、portfolio snapshot、tool history、summary。
- `eval`：保存 3-5 个可自动判定的任务。
- `ui`：展示会话、工具调用过程、工具结果、错误状态。

Agent Loop 必须体现这些工程点：

- 状态机：`WAITING_INPUT`、`WAITING_CONFIRM`、`RUNNING`、`DONE`、`ERROR`。
- `tool_calls` 后面必须紧跟对应 `tool_result`，中间不要插其他消息。
- 工具输出要有限制，超限返回摘要或外部存储引用。
- 业务规则不要全塞进 system prompt，要在工具调用前结构化注入。
- 可观测性埋在决策点：触发执行、条件不满足、触发但拦截、执行出错。
- 上下文压缩分两层：先清旧工具结果，再做 summary；压缩后重新注入任务目标和身份。

第一阶段只允许产出：

- `docs/specs/finance-agent-demo-spec.md`
- `docs/evaluation/finance-agent-eval-set.md`

第一阶段禁止：

- 写业务源码。
- 搭 UI。
- 实现工具。
- 安装依赖。
- 一口气生成完整项目。

闸门：Spec 和 evaluation set 没写完，不能进入第二段。

---

## 第二段：再把通用底座收窄成 Finance Agent

通用底座跑通后，再做金融场景收窄。不要做万能金融助手，初版只保留 3-5 个工具。

这些 Tool 不要求学员一开始就接真实金融 API。教学版默认先用**本地 mock tool** 跑通 Tool Calling 链路，再把工具实现替换成真实数据源。

Tool 获取方式分三层：

| 层级 | 方式 | 适合谁 | 要求 |
|---|---|---|---|
| Level 1 | 本地 mock tools | 所有学员默认使用 | 不需要 API key，不需要 LangChain，不需要外部服务 |
| Level 2 | 公开 HTTP API / npm SDK | 想接真实数据的学员 | 需要自己申请 key，并把 key 放进 `.env` |
| Level 3 | LangChain / MCP / 自定义工具包 | 已有工具链的学员 | 需要按课程说明额外安装 |

默认实现 Level 1。不要因为没有真实行情 API 就停下来。

本地 mock tools 至少提供这些固定数据：

```typescript
const mockQuotes = {
  "AAPL.US": { price: 189.98, change: 1.24, changePercent: 0.66, volume: 53120000 },
  "TSLA.US": { price: 177.42, change: -2.16, changePercent: -1.2, volume: 88100000 },
  "NVDA.US": { price: 924.79, change: 12.31, changePercent: 1.35, volume: 42190000 },
};

const mockPortfolio = [
  { symbol: "AAPL.US", quantity: 20, costBasis: 165.2 },
  { symbol: "NVDA.US", quantity: 8, costBasis: 710.0 },
  { symbol: "TSLA.US", quantity: 10, costBasis: 210.5 },
];
```

工具接口要稳定，数据源可以替换。也就是说，`get_quote`、`get_kline`、`get_portfolio`、`search_news`、`risk_summary` 的参数和返回结构不能因为 mock / real source 切换而改变。

如果要接真实数据源，只允许作为可选增强：

- 可以接 Alpha Vantage、Finnhub、Polygon、Yahoo Finance 非官方包、LongPort / LongBridge、Tushare 等。
- 必须写清楚安装命令、环境变量、失败 fallback。
- 没有 key 时自动 fallback 到 mock tools。
- 真实数据源失败不能阻塞教学主流程。

工具范围固定为：

| 工具 | 用途 | 参数 |
|---|---|---|
| `get_quote` | 查实时行情 | `symbol`，格式如 `AAPL.US` |
| `get_kline` | 查 K 线数据 | `symbol`、`period`，只允许 `1d` / `1w` / `1m` |
| `get_portfolio` | 读取示例持仓 | 无 |
| `search_news` | 查相关新闻 | `keyword`、`days` |
| `risk_summary` | 分析组合风险 | 无，内部读取持仓 |

其他工具显式禁用，不要放在上下文里让模型自己“不调用”。工具越多，模型越容易选错。

每个工具描述必须写清楚：

- 必填字段。
- 字段类型。
- 枚举值。
- 输入格式。
- 返回值包含什么。
- 失败时返回什么错误码。

错误写法：

```typescript
description: "获取股票报价"
```

正确写法：

```typescript
description: "获取实时股票报价。输入 symbol 必须是 SYMBOL.MARKET 格式，例如 AAPL.US。返回 price、change、changePercent、volume。"
```

评测集必须先于优化存在。至少准备 5 个任务：

| 任务 | 用户输入 | 预期工具调用 | 通过标准 |
|---|---|---|---|
| 行情查询 | `AAPL 现在多少钱？` | `get_quote("AAPL.US")` | 输出价格和涨跌幅 |
| K 线分析 | `给我看 TSLA 一个月的 K 线` | `get_kline("TSLA.US", "1m")` | 输出至少 20 根 OHLCV 或摘要 |
| 持仓查询 | `我现在的持仓是什么？` | `get_portfolio()` | 列出代码、数量、成本价、盈亏 |
| 新闻搜索 | `找一下 NVDA 最新新闻` | `search_news("NVDA", 7)` | 输出至少 3 条带日期新闻 |
| 风险摘要 | `分析一下我的组合风险` | `risk_summary()` | 输出集中度、波动风险、风险提示 |

失败不要只记失败，要归因：

- `tool_selection`：工具选错。
- `parameter_parsing`：参数传错。
- `output_synthesis`：最终回答缺信息。
- `business_rule`：业务规则没遵守。
- `context_management`：上下文没用对。

第二阶段只允许产出：

- 完整 Spec。
- 完整 evaluation set。
- `docs/superpowers/plans/finance-agent-demo-plan.md`
- worktree 方案。

创建 worktree 时必须用独立分支，不要多个 worktree 绑同一个 `main`：

```bash
git worktree add -b codex/finance-agent-demo worktrees/finance-agent-demo main
```

如果拆多个并行任务，每个任务单独分支：

```bash
git worktree add -b codex/finance-agent-ui worktrees/finance-agent-ui main
git worktree add -b codex/finance-agent-tools worktrees/finance-agent-tools main
git worktree add -b codex/finance-agent-eval worktrees/finance-agent-eval main
```

使用 `superpowers:writing-plans` 写实现计划。计划必须包含：

- 文件结构。
- 每个文件职责。
- 依赖顺序。
- 每个功能的 TDD 测试点。
- subagent 分工。
- 浏览器验证方式。
- 评测运行方式。

闸门：没有实现计划、没有 worktree、没有 TDD 任务拆分，不能进入第三段。

---

## 第三段：最后按成本结构执行，不要平均用力

现在才开始写代码。

执行顺序固定：

1. 用 `superpowers:test-driven-development`。
2. 每个行为先写失败测试。
3. 确认测试因为目标功能缺失而失败。
4. 写最少代码让测试通过。
5. 重构。
6. 可拆分任务用 `superpowers:subagent-driven-development`。
7. 前端完成后做 browser verification。
8. 跑 evaluation set。
9. 写 failure attribution。
10. 用 `superpowers:verification-before-completion` 做最终检查。

资源投入顺序：

| 优先级 | 先砸什么 | 为什么 |
|---|---|---|
| 1 | 评测集 | 没评测就是闭眼调 prompt |
| 2 | 工具描述 | 字段猜错一次就浪费一轮 |
| 3 | 策略规则注入 | 模型会忘 system prompt，工具前提醒更稳 |
| 4 | 推理模式 | 只给复杂判断、风险分析、数值计算用 |
| 5 | 模型升级 | 放最后，外围机制没调好时换模型没意义 |

简单查询不要开推理模式：

- 单次价格查询。
- 单次新闻查询。
- 基本持仓读取。

复杂任务再开推理模式：

- 组合风险分析。
- 集中度判断。
- 新闻和持仓结合分析。
- 多规则冲突检查。

前端必须浏览器验证，至少检查：

- 页面能打开。
- 工具调用过程能显示。
- 工具结果能渲染。
- 错误状态能显示。
- 深色模式不破版。
- 移动端不重叠。
- 文本不溢出按钮或卡片。

最终交付物：

| 交付物 | 路径 |
|---|---|
| Spec | `docs/specs/finance-agent-demo-spec.md` |
| 实现计划 | `docs/superpowers/plans/finance-agent-demo-plan.md` |
| 评测集 | `docs/evaluation/finance-agent-eval-set.md` |
| 评测结果 | `docs/evaluation/results.md` |
| 可运行原型 | `apps/finance-agent/` |
| 学员说明 | `README.md` |

最终验收不是看关键词有没有出现，而是看流程有没有真的发生：

- 是否先写 Spec？
- 是否先建 evaluation set？
- 是否先写 plan 再写代码？
- 是否使用独立 worktree？
- 是否每个功能都有 red -> green 的 TDD 记录？
- 是否做了 browser verification？
- 是否跑了 evaluation set？
- 是否记录 failure attribution？
- 学员是否能解释：为什么不是让 AI 一口气生成完整项目，而是让 AI 被流程约束成靠谱工程师？

一句话：这份提示词不是让 AI 表演一次性写完项目，而是让 AI 学会按工程流程工作。先搞清楚，再计划，再测试，再实现，再验证。
