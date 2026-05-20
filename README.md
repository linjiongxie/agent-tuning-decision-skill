# Agent Tuning Decision Skill

`agent-tuning-decision` is a Codex skill for Agent behavior tuning. It helps Codex trace root cause to completion, compare past and present behavior, prefer Prompt/workflow/contract fixes before fallback code, and verify changes with recent real conversation turns when possible.

## Install

Manual install:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R agent-tuning-decision "${CODEX_HOME:-$HOME/.codex}/skills/"
```

Install from GitHub after publishing this repo:

```bash
$skill-installer install https://github.com/linjiongxie/agent-tuning-decision-skill/tree/main/agent-tuning-decision
```

Restart Codex after installing so the skill metadata is loaded.

## Use

Invoke it explicitly:

```text
$agent-tuning-decision Trace why this Agent behavior appeared and recommend whether to tune the Prompt, orchestration, contract, or fallback layer.
```

The skill can also trigger automatically when a request involves Agent tuning, Prompt changes, workflow orchestration, state recovery, event/UI ownership, replay behavior, fallback logic, root-cause analysis, or historical turn backtesting.

## Contents

```text
agent-tuning-decision/
├── SKILL.md
└── agents/
    └── openai.yaml
```
