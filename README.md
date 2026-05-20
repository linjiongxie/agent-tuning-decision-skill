# Agent Tuning Decision Skill

`agent-tuning-decision` is a Codex skill for making better Agent tuning decisions. It helps Codex avoid jumping straight to fallback code, trace the root cause of an Agent behavior issue, compare past and current behavior, choose the right fix layer, and verify Prompt or orchestration changes with real historical turns when possible.

Use it when the hard part is not just "what code should change?", but "which layer should own this behavior?"

## What It Is For

Agent behavior bugs often look deceptively simple: a reply option appears by default, a workflow stops early, a tool is called when it should not be, a UI card looks like current Agent output, or a Prompt change seems right but is hard to verify.

This skill makes Codex slow down at the right moment and answer:

- What is the real source of the behavior?
- Did this happen before, or is it new?
- If it is new, why did it appear now?
- If it happened before, why was it hidden or tolerated?
- Should the fix live in the Prompt, workflow orchestration, event/API contract, UI rendering, state/replay layer, or fallback code?
- Can the change be checked against recent real conversation turns instead of only judged by intuition?

## Core Behavior

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

## Historical Backtesting

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

## Expected Answer Shape

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

## Example Use Cases

### A default reply option appears

User:

```text
$agent-tuning-decision Why does this reply option appear even though it was not emitted by the current Agent turn?
```

Expected behavior:

Codex should first trace whether the option came from current Agent output, Prompt instructions, backend payload defaults, replay state, frontend fallback, or UI rendering. If the option was synthesized by hidden fallback code, the recommendation should be to remove or make that source explicit rather than tune the Prompt or add more dedupe logic.

### An Agent Prompt change feels plausible but unproven

User:

```text
$agent-tuning-decision We changed the Prompt so the Agent should ask a clarification question before calling a tool. How do we know it is better?
```

Expected behavior:

Codex should recommend a small backtest set starting with recent true turns, define expected behavior before judging output, compare baseline and candidate tool calls, and report whether the Prompt change is better, worse, neutral, narrow-only, or inconclusive.

### A workflow stalls after UI input

User:

```text
$agent-tuning-decision The user answered a UI card, but the Agent did not continue the conversation. Should we add fallback text?
```

Expected behavior:

Codex should inspect orchestration first: whether the UI response path updated state, whether continuation was invoked, whether stale context was used, and whether the event sequence ended too early. Fallback copy should only be considered after the orchestration source is understood.

## Install

### From This Repository

Clone the repo and copy the skill folder into Codex's skill directory:

```bash
git clone https://github.com/linjiongxie/agent-tuning-decision-skill.git
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R agent-tuning-decision-skill/agent-tuning-decision "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Restart Codex after installing so the skill metadata is loaded.

### With Codex Skill Installer

In Codex, ask:

```text
Use $skill-installer to install https://github.com/linjiongxie/agent-tuning-decision-skill/tree/main/agent-tuning-decision
```

Restart Codex after installing.

## Explicit Invocation

You can force the skill by naming it:

```text
$agent-tuning-decision Trace why this Agent behavior appeared and recommend whether to tune the Prompt, orchestration, contract, or fallback layer.
```

It can also trigger automatically when a request involves Agent tuning, Prompt changes, workflow orchestration, state recovery, event/UI ownership, replay behavior, fallback logic, root-cause analysis, or historical turn backtesting.

## Repository Structure

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

## Validate

If you have Codex's `skill-creator` system skill installed, validate the skill folder with:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py agent-tuning-decision
```

## License

MIT
