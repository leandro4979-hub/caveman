---
name: cavecrew
description: >
  Decision guide for delegating to caveman-style subagents. Tells the main
  thread WHEN to spawn `cavecrew-investigator` (locate code), `cavecrew-builder`
  (1-2 file edit), or `cavecrew-reviewer` (diff review) instead of doing the
  work inline or using vanilla `Explore`. Subagent output is caveman-compressed
  so the tool-result injected back into main context is ~60% smaller — main
  context lasts longer across long sessions.
  Trigger: "delegate to subagent", "use cavecrew", "spawn investigator/builder/reviewer",
  "save context", "compressed agent output".
---

Cavecrew = three subagent presets that emit caveman output. Same job as Anthropic defaults (`Explore`, edit-style agents, reviewer); difference is the tool-result they return is compressed, so main context shrinks per delegation.

## When to use cavecrew vs alternatives

| Task | Use |
|---|---|
| "Where is X defined / what calls Y / list uses of Z" | `cavecrew-investigator` |
| Same but you also want suggestions/architecture commentary | `Explore` (vanilla) |
| Surgical edit, ≤2 files, scope obvious | `cavecrew-builder` |
| New feature / 3+ files / cross-cutting refactor | Main thread or `feature-dev:code-architect` |
| Review diff, branch, or file for bugs | `cavecrew-reviewer` |
| Deep code review with rationale + alternatives | `Code Reviewer` (vanilla) |
| One-line answer you already know | Main thread, no subagent |

Rule of thumb: **if you'd want the subagent's output in 1/3 the tokens, pick cavecrew. If you'd want prose, pick vanilla.**

## Before delegation

Main thread stays accountable. Give every specialist a bounded task envelope:

- `Objective` — one observable outcome.
- `Owner` — one agent; no shared accountability.
- `Inputs` — exact files, symbols, diff, or facts.
- `Outputs` — expected artifact plus status contract.
- `Boundaries` — allowed tools/files/mutations and exclusions.
- `Failure behavior` — stop or return safe partial evidence.
- `Completion criteria` — checks that make the task complete.
- `Approval boundary` — actions that require user approval.

If objective, owner, inputs, output, or completion criteria are unclear, keep
work in main thread or ask user. Do not delegate an open-ended mission.

## Why this exists (the real win)

Subagent tool results get injected into main context verbatim. A vanilla `Explore` that returns 2k tokens of prose costs 2k tokens of main-context budget every time. The same finding from `cavecrew-investigator` returns ~700 tokens. Across 20 delegations in one session that's the difference between context exhaustion and finishing the task.

## Output contracts

Every agent returns the same status envelope. Values stay terse; fields never
disappear:

```text
Objective: <assigned outcome>
Owner: <agent name>
Inputs: <paths, symbols, diff, or facts used>
Current state: <not-started | in-progress | blocked | failed | complete>
Evidence: <verified citations, checks, or none>
Result: <agent-specific payload, partial result, or none>
Blockers: <specific blocker or none>
Approval required: <exact action and reason, or none>
Next milestone: <next observable checkpoint or none — complete>
```

`complete` requires the task's completion criteria and evidence. Refusal and
failure still use all fields.

Agent-specific `Result` payloads:

**`cavecrew-investigator`**
```
<Header>:
- path:line — `symbol` — short note
totals: <counts>.
```
Or `No match.` Always file-path-first, line-number-attached, backticked symbols. Safe to grep with `path:\d+`.

**`cavecrew-builder`**
```
<path:line-range> — <change ≤10 words>.
verified: <re-read OK | mismatch @ path:line>.
```
On refusal, put `too-big.`, `needs-confirm.`, `ambiguous.`, or `regressed.` in
`Result`; set state to `blocked` or `failed` and name the next checkpoint.

**`cavecrew-reviewer`**
```
path:line: <emoji> <severity>: <problem>. <fix>.
totals: N🔴 N🟡 N🔵 N❓
```
Or `No issues.` Findings sorted file → line ascending.

## Chaining patterns

**Locate → fix → verify** (most common):
1. `cavecrew-investigator` returns site list.
2. Main thread picks 1-2 sites, hands paths to `cavecrew-builder`.
3. `cavecrew-reviewer` audits the diff.

**Parallel scout** (when investigation is broad):
Spawn 2-3 `cavecrew-investigator` calls in one message (different angles: defs vs callers vs tests). Aggregate in main thread.

Parallel only when tasks have no write overlap, result dependency, shared
approval decision, or coupled verification. Otherwise sequence them.

**Single-shot edit** (when site is already known):
Skip investigator. Hand exact path:line to `cavecrew-builder` directly.

## What NOT to do

- Don't use `cavecrew-builder` when you don't already know the file. Spawn investigator first or main thread will eat tokens passing context.
- Don't chain `cavecrew-investigator → cavecrew-builder` for a 5-file refactor. Builder will return `too-big.` and you'll have wasted a turn.
- Don't ask `cavecrew-reviewer` for "general feedback" — it returns findings only, no architecture opinions. Use `Code Reviewer` for that.
- Don't expect prose. Cavecrew output is structured, sometimes terse to the point of cryptic. If a human will read it directly, paraphrase.

## Authority and recovery

- User is final decision-maker. Main thread coordinates and remains accountable.
- Delegation transfers task work, never authority or final ownership.
- Approval required before destructive, privileged, security-sensitive,
  financial, externally visible, or out-of-envelope action.
- Denial or missing approval is not permission. Never retry the same action via
  another tool, fallback, account, browser, or subagent.
- Reuse completed results while inputs and evidence remain current. Repeat only
  after changed inputs, stale evidence, failed downstream validation, or an
  explicit rerun request.
- On failure: report evidence, classify blocker, return safe partial result,
  and escalate the smallest required decision. Never broaden scope silently.

## Auto-clarity (inherited)

Subagents drop caveman → normal English for security warnings, irreversible-action confirmations, and any output where fragment ambiguity could be misread. Resume caveman after.
