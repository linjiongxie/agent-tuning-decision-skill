---
name: agent-tuning-decision
description: "Use when deciding which layer should own Agent behavior changes: prompts, tool/workflow orchestration, framework-native ownership, runtime-specific evidence, repeated fix loops, state recovery, event/UI ownership, replay behavior, fallback logic, root-cause tradeoffs, evolution candidates, reusable tuning rules, or 采纳候选."
---

# Agent Tuning Decision

## Core Rule

Track the root cause to completion before debating fixes. Do not treat fallback code as the default answer to an Agent behavior problem.

Prefer Prompt, workflow orchestration, and contract changes when the issue is about intent, sequencing, tool selection, event ownership, or Agent-visible reasoning. Use deterministic fallback logic only for safety, compatibility, latency protection, external uncertainty, or explicit user-visible recovery.

When another debugging skill also applies, use it for evidence gathering and root-cause tracing, then use this skill for layer choice, recommended path, tradeoff framing, and backtest acceptance. General debugging decides what happened; this skill decides which Agent behavior layer should change.

## Root Cause Workflow

1. Locate the behavior source.
   - Check whether the behavior came from the current Agent turn, Prompt instruction, tool result, workflow branch, runtime state or run history, backend payload default, saved history/replay state, frontend fallback, or UI renderer.
   - If the user says "不是本轮 Agent 输出" or asks why something appeared by default, treat provenance as the main question.

2. Compare past and present.
   - Ask why this issue appears.
   - Check whether it appeared before.
   - If it did not appear before, explain why it appears now.
   - If it did appear before, explain why it was hidden, tolerated, or not fixed.
   - Use current code, recent stored turns, event history, tests, and logs as evidence before proposing a fix.

3. Classify the right layer.
   - Prompt problem: the Agent misunderstood intent, chose the wrong reasoning frame, picked the wrong tool policy, or failed to produce the expected customer-facing response.
   - Orchestration problem: the workflow skipped a continuation, called tools in the wrong order, used stale context, ended the turn too early, or mishandled a blocking UI wait.
   - Contract problem: Agent, tool, backend, and UI disagree on event ownership, payload shape, field semantics, or replay compatibility.
   - State/replay problem: saved rows, replay fixtures, slots, or historical events are being mistaken for live behavior.
   - Fallback problem: deterministic code is synthesizing behavior that looks like Agent output.
   - UI problem: rendering, suppression, dedupe, routing, or copy presentation changes what the user sees.

4. Recommend one path first.
   - State the recommended path directly.
   - Explain the selection logic.
   - Include short-term benefit, long-term benefit, risk, verification method, and the evidence that would change the recommendation.
   - Do not present three options by default. Compare multiple options only when they are genuinely different strategies.

## Layer Choice

- Use Prompt changes for intent framing, response policy, tool-selection criteria, and expected customer-facing language.
- Use workflow orchestration changes for turn continuation, tool ordering, blocking UI waits, event sequencing, and state transitions.
- Use contract changes for public event names, payload ownership, DTO shape, replay fixtures, and cross-owner boundaries.
- Use fallback changes only to preserve invariants, provide explicit recovery, handle legacy data, or protect users from external failures.
- Remove hidden defaults when they make UI behavior look like current Agent output without a current Agent decision.

## Historical Backtesting

For Prompt or orchestration changes, prefer testing against real historical conversation inputs when available.

Choose samples in this order:

1. Recent true turns, especially turns immediately before and after the user noticed the issue.
2. Older true turns for the same behavior family, covering success, failure, and boundary cases.
3. Checked-in replay fixtures.
4. Artificial cases only when real turns or fixtures are unavailable.

Use a small eval set first: 3-10 representative turns. Write the expected behavior change before judging the result.

Compare baseline and candidate behavior across:

- customer-visible answer content
- tool calls and tool order
- event sequence
- UI requests or cards emitted
- slot/state changes
- whether the user intent advanced or stalled
- whether old failure modes disappeared
- whether new regressions appeared

Distinguish deterministic replay from live rerun:

- deterministic replay or history diff checks event contracts, rendering, saved payloads, and compatibility
- live rerun or A/B eval checks whether a new Prompt or workflow actually changes Agent behavior

When reporting backtest results, give a judgment: better, worse, no meaningful improvement, fixes the narrow case but weakens general behavior, or inconclusive because the eval set or instrumentation is insufficient.

## Response Shape

For non-trivial Agent tuning decisions, use this shape:

- Root cause: one sentence with the cause; if unknown, name the missing evidence.
- Past vs now: explain whether the issue existed before, and why it appears or disappears now.
- Recommended path: one recommended fix layer and action.
- Why this layer: explain why Prompt, orchestration, contract, fallback, or UI is the right layer.
- Tradeoff: short-term benefit, long-term benefit, risk, and switch condition.
- Verification: include code checks, runtime-specific evidence when applicable, recent true-turn backtesting, and fixture or artificial cases only when needed.

## Self-Evolution Protocol

After a non-trivial Agent tuning session, decide whether this skill learned a reusable rule from the current triggered conversation. Do not scan unrelated historical sessions by default, and do not edit this skill automatically.

If there is a strong candidate, add one `Evolution candidate` section after the main answer:

- Trigger: the repeated decision bias, correction pattern, verification method, or skill coordination issue that appeared.
- Rule: the reusable instruction that could improve this skill.
- Evidence: concrete current-session behavior or user correction that supports the rule.
- Boundary: when the rule should not apply.
- Suggested location: where the rule belongs in this skill.
- Minimal acceptance case: the smallest future prompt or scenario that should behave differently after adoption.

Propose at most one candidate per response. If there is not enough evidence, write `No evolution candidate`.

Only promote a candidate into `SKILL.md` when the user explicitly asks to adopt it, for example "补进去", "update skill", or "采纳这个候选". Before promotion, check that the rule generalizes beyond one project or one bug, does not include private project names, local paths, database table names, or internal endpoints, and does not weaken the existing root-cause-first, Prompt/orchestration/contract-preferred, fallback-limited, and recent-true-turn backtesting rules.

## Review Loop Extraction Gate

When the same behavior family goes through two or more `review -> fix -> eval -> new issue` cycles, stop before adding another Prompt rule, runtime guard, retry, or fallback. Classify the loop source first:

- Missing contract: add or tighten an executable test, schema, fixture assertion, event contract, or tool payload/order check.
- Wrong owner layer: move the fix to Prompt, orchestration, contract, state/replay, fallback, or UI according to the root cause instead of patching whichever layer failed last.
- Model variance: avoid encoding one stochastic miss as permanent runtime behavior unless it protects an invariant or user-visible recovery path.
- Eval gap: repair the scenario, instrumentation, baseline, or acceptance criteria before judging the behavior.

Promote the loop result only when it leaves a durable contract, clearer ownership boundary, or reusable verification method. If it only adds another narrow retry or guard, report the guard-creep risk and name the evidence that would justify keeping it.

When the baseline runtime or historical fixture has not changed, prefer candidate-only verification against frozen scenario contracts for repeated tuning loops. Rerun a slow or volatile baseline only when the baseline changed, the comparison itself is disputed, or the acceptance gate depends on a fresh live A/B result.

## Project-Specific Evidence

In each project, inspect current repo sources before assuming exact tool names. Look for saved turns, event logs, replay fixtures, history diff tools, orchestration replay scripts, benchmark tools, or A/B eval harnesses. Treat saved history as historical evidence, not live recomputation. For recent behavior questions, start from the newest relevant true turns before falling back to fixtures or artificial cases.

## Runtime-Specific Evidence

When a project uses an external agent runtime, workflow engine, or harness, identify whether the behavior is owned by the app framework, runtime, harness, or project glue before choosing Prompt, orchestration, contract, fallback, or UI changes.

When the behavior may belong to an open-source framework's native responsibilities, check the official docs, Wiki, or source entry points for the installed or target version before designing project-level replacements. Look for native primitives for state, routing, checkpoints, interrupt/resume, streaming, retries, waits/signals, middleware, tool calls, durability, and replay. Prefer those primitives and keep project code as thin glue.

Add project-level wrappers, guards, retries, or fallbacks only when framework primitives cannot meet the product contract, compatibility boundary, safety requirement, latency budget, observability need, or migration constraint. State that reason explicitly.

Use runtime docs only as evidence navigation, not as copied policy. For stateful graph runtimes, inspect graph state, checkpoints or persistence, interrupt/resume semantics, stream events, durability mode, and replay boundaries. For durable function or event runtimes, inspect trigger/event identity, step identity, memoized results, retries, waits/signals, concurrency, idempotency, and run history.

Do not add vendor how-to details, API recipes, or project-specific runtime names to this public skill unless the same runtime-specific mistake repeats across projects.

## Avoid

- Adding fallback before tracing provenance.
- Auto-editing this skill from an evolution candidate without explicit user adoption.
- Repeatedly debating options after the root-cause path is discoverable from evidence.
- Treating historical replay rows as current Agent behavior.
- Hiding Agent failures with backend or frontend defaults.
- Offering three options when one recommendation is clearly stronger.
