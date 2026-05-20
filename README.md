# Agent Tuning Decision Skill

[English](#english) | [中文](#中文)

## English

`agent-tuning-decision` is a Codex skill for making better Agent tuning decisions. It helps Codex avoid jumping straight to fallback code, trace the root cause of an Agent behavior issue, compare past and current behavior, choose the right fix layer, and verify Prompt or orchestration changes with real historical turns when possible.

Use it when the hard part is not just "what code should change?", but "which layer should own this behavior?"

### What It Is For

Agent behavior bugs often look deceptively simple: a reply option appears by default, a workflow stops early, a tool is called when it should not be, a UI card looks like current Agent output, or a Prompt change seems right but is hard to verify.

This skill makes Codex slow down at the right moment and answer:

- What is the real source of the behavior?
- Did this happen before, or is it new?
- If it is new, why did it appear now?
- If it happened before, why was it hidden or tolerated?
- Should the fix live in the Prompt, workflow orchestration, event/API contract, UI rendering, state/replay layer, or fallback code?
- Can the change be checked against recent real conversation turns instead of only judged by intuition?

### Core Behavior

The skill pushes Codex toward one recommended path first, backed by evidence and tradeoffs.

It should prefer:

- Prompt changes for intent framing, tool-selection criteria, response policy, and customer-facing wording expectations.
- Workflow orchestration changes for turn continuation, tool order, blocking UI waits, event sequencing, and state transitions.
- Contract changes when Agent, tools, backend, UI, and replay disagree on event ownership, payload shape, or field semantics.
- Fallback changes only for invariants, compatibility, latency protection, external failures, legacy data, or explicit user-visible recovery.

It should avoid:

- Adding fallback before tracing provenance.
- Hiding Agent failures with backend or frontend defaults.
- Repeatedly discussing options when the root-cause path is discoverable from evidence.
- Presenting three options by default when one recommendation is clearly stronger.

### Historical Backtesting

For Prompt and orchestration changes, the skill encourages Codex to evaluate changes against real past behavior.

Sample priority:

1. Recent true turns, especially the turns immediately before and after the issue was noticed.
2. Older true turns from the same behavior family.
3. Checked-in replay fixtures.
4. Artificial cases only when real turns or fixtures are unavailable.

The comparison should look at customer-visible output, tool calls and order, event sequence, UI cards, slot/state changes, whether the user intent advanced, whether old failures disappeared, and whether new regressions appeared.

The skill distinguishes deterministic replay/history diff from live rerun/A-B eval:

- Replay/history diff checks saved payloads, event contracts, rendering, and compatibility.
- Live rerun/A-B eval checks whether a new Prompt or workflow actually changes Agent behavior.

### Expected Answer Shape

When the skill is used on a non-trivial Agent tuning question, Codex should answer in this shape:

```text
Root cause: ...
Past vs now: ...
Recommended path: ...
Why this layer: ...
Tradeoff: ...
Verification: ...
```

The exact formatting can vary, but the substance should be present: root cause, past/current contrast, a single recommended path, layer choice, tradeoffs, and verification.

### Example Use Cases

#### A default reply option appears

User:

```text
$agent-tuning-decision Why does this reply option appear even though it was not emitted by the current Agent turn?
```

Expected behavior:

Codex should first trace whether the option came from current Agent output, Prompt instructions, backend payload defaults, replay state, frontend fallback, or UI rendering. If the option was synthesized by hidden fallback code, the recommendation should be to remove or make that source explicit rather than tune the Prompt or add more dedupe logic.

#### An Agent Prompt change feels plausible but unproven

User:

```text
$agent-tuning-decision We changed the Prompt so the Agent should ask a clarification question before calling a tool. How do we know it is better?
```

Expected behavior:

Codex should recommend a small backtest set starting with recent true turns, define expected behavior before judging output, compare baseline and candidate tool calls, and report whether the Prompt change is better, worse, neutral, narrow-only, or inconclusive.

#### A workflow stalls after UI input

User:

```text
$agent-tuning-decision The user answered a UI card, but the Agent did not continue the conversation. Should we add fallback text?
```

Expected behavior:

Codex should inspect orchestration first: whether the UI response path updated state, whether continuation was invoked, whether stale context was used, and whether the event sequence ended too early. Fallback copy should only be considered after the orchestration source is understood.

### Install

#### From This Repository

Clone the repo and copy the skill folder into Codex's skill directory:

```bash
git clone https://github.com/linjiongxie/agent-tuning-decision-skill.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R agent-tuning-decision-skill/agent-tuning-decision "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Restart Codex after installing so the skill metadata is loaded.

#### With Codex Skill Installer

In Codex, ask:

```text
Use $skill-installer to install https://github.com/linjiongxie/agent-tuning-decision-skill/tree/main/agent-tuning-decision
```

Restart Codex after installing.

### Explicit Invocation

You can force the skill by naming it:

```text
$agent-tuning-decision Trace why this Agent behavior appeared and recommend whether to tune the Prompt, orchestration, contract, or fallback layer.
```

It can also trigger automatically when a request involves Agent tuning, Prompt changes, workflow orchestration, state recovery, event/UI ownership, replay behavior, fallback logic, root-cause analysis, or historical turn backtesting.

### Repository Structure

```text
agent-tuning-decision-skill/
├── README.md
├── LICENSE
└── agent-tuning-decision/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

The skill itself is only the `agent-tuning-decision/` folder. Root-level files are for sharing and documentation.

### Validate

If you have Codex's `skill-creator` system skill installed, validate the skill folder with:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py agent-tuning-decision
```

### License

MIT

## 中文

`agent-tuning-decision` 是一个用于 Agent 调优决策的 Codex skill。它帮助 Codex 避免一上来就补兜底代码，而是先把 Agent 行为问题的根因追到底，对比过去和现在的行为，选择正确的修复层级，并在可能时用真实历史 turn 验证 Prompt 或工作流编排改动。

当问题的重点不只是“改哪段代码”，而是“这个行为应该由哪一层负责”时，适合使用这个 skill。

### 适用场景

Agent 行为问题经常看起来很简单：某个回复选项默认出现、工作流提前停住、工具被错误调用、某个 UI 卡片看起来像当前 Agent 输出，或者 Prompt 改动看起来合理但无法证明更好。

这个 skill 会让 Codex 在合适的位置慢下来，先回答：

- 这个行为的真实来源是什么？
- 这个问题过去出现过吗，还是新出现的？
- 如果是新问题，为什么现在出现？
- 如果过去就有，为什么之前被隐藏、被容忍，或者没有修？
- 修复应该落在 Prompt、工作流编排、event/API 契约、UI 渲染、状态/replay 层，还是 fallback 代码？
- 这个改动能不能用最近真实对话 turn 验证，而不是只靠直觉判断？

### 核心行为

这个 skill 会推动 Codex 先给一个推荐路径，并用证据和取舍支撑它。

它应该优先考虑：

- 用 Prompt 修改解决意图框定、工具选择标准、回复策略和面向用户的话术预期。
- 用工作流编排修改解决 turn continuation、工具顺序、阻塞式 UI 等待、事件顺序和状态流转。
- 当 Agent、工具、后端、UI 和 replay 对事件归属、payload 形状或字段语义理解不一致时，优先修契约。
- 只有在不变量保护、兼容性、延迟保护、外部失败、历史数据或显式用户可见恢复场景下，才使用 fallback 修改。

它应该避免：

- 没追踪来源就先加 fallback。
- 用后端或前端默认值隐藏 Agent 失败。
- 在根因路径可以通过证据查明时反复讨论选项。
- 当一个推荐方案明显更强时，仍然默认给三个选项。

### 历史回测

对于 Prompt 和工作流编排改动，这个 skill 会鼓励 Codex 用真实历史行为来验证。

样本优先级：

1. 最近的真实 turn，尤其是用户发现问题前后的 turn。
2. 同类行为的更早真实 turn，覆盖成功、失败和边界场景。
3. 已入仓的 replay fixture。
4. 只有在没有真实 turn 或 fixture 时，才使用人工构造 case。

对比时应该关注：用户可见输出、工具调用和顺序、事件序列、UI 卡片、slot/state 变化、用户意图是否继续推进、旧失败是否消失，以及是否引入新回归。

这个 skill 区分 deterministic replay/history diff 和 live rerun/A-B eval：

- Replay/history diff 用于检查历史 payload、事件契约、渲染和兼容性。
- Live rerun/A-B eval 用于检查新 Prompt 或新工作流是否真的改变 Agent 行为。

### 预期回答形态

当这个 skill 用在非平凡的 Agent 调优问题上时，Codex 应该按这个形态回答：

```text
Root cause: ...
Past vs now: ...
Recommended path: ...
Why this layer: ...
Tradeoff: ...
Verification: ...
```

格式可以灵活，但内容必须包含：根因、过去/现在对比、一个推荐路径、层级选择、取舍和验证方式。

### 示例场景

#### 默认回复选项出现

用户：

```text
$agent-tuning-decision Why does this reply option appear even though it was not emitted by the current Agent turn?
```

预期行为：

Codex 应该先追踪这个选项来自当前 Agent 输出、Prompt 指令、后端 payload 默认值、replay 状态、前端 fallback，还是 UI 渲染。如果它是隐藏 fallback 代码合成出来的，推荐路径应该是删除或显式化这个来源，而不是调 Prompt 或继续加去重逻辑。

#### Agent Prompt 改动看起来合理但还没被证明

用户：

```text
$agent-tuning-decision We changed the Prompt so the Agent should ask a clarification question before calling a tool. How do we know it is better?
```

预期行为：

Codex 应该建议从最近真实 turn 开始构造一个小回测集，先写清楚期望行为，再比较 baseline 和 candidate 的工具调用，并判断 Prompt 改动是更好、更差、中性、只修了窄场景，还是因为样本或观测不足而无法判断。

#### 用户回答 UI 卡片后工作流停住

用户：

```text
$agent-tuning-decision The user answered a UI card, but the Agent did not continue the conversation. Should we add fallback text?
```

预期行为：

Codex 应该先检查工作流编排：UI response 路径是否更新了状态、是否触发 continuation、是否用了过期上下文、事件序列是否过早结束。只有在编排来源被查明后，才考虑 fallback 文案。

### 安装

#### 从这个仓库安装

克隆仓库，并把 skill 文件夹复制到 Codex 的 skills 目录：

```bash
git clone https://github.com/linjiongxie/agent-tuning-decision-skill.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R agent-tuning-decision-skill/agent-tuning-decision "${CODEX_HOME:-$HOME/.codex}/skills/"
```

安装后重启 Codex，让新的 skill metadata 生效。

#### 使用 Codex Skill Installer

在 Codex 中输入：

```text
Use $skill-installer to install https://github.com/linjiongxie/agent-tuning-decision-skill/tree/main/agent-tuning-decision
```

安装后重启 Codex。

### 显式调用

你可以通过点名强制使用这个 skill：

```text
$agent-tuning-decision Trace why this Agent behavior appeared and recommend whether to tune the Prompt, orchestration, contract, or fallback layer.
```

当请求涉及 Agent 调优、Prompt 修改、工作流编排、状态恢复、event/UI 归属、replay 行为、fallback 逻辑、根因分析或历史 turn 回测时，它也可以自动触发。

### 仓库结构

```text
agent-tuning-decision-skill/
├── README.md
├── LICENSE
└── agent-tuning-decision/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

真正的 skill 只有 `agent-tuning-decision/` 文件夹。根目录文件用于共享和说明。

### 校验

如果你安装了 Codex 的 `skill-creator` system skill，可以用下面的命令校验 skill 文件夹：

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py agent-tuning-decision
```

### 许可证

MIT
