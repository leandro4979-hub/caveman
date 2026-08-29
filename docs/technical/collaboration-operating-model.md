# Collaboration operating model

Caveman keeps collaboration simple: the user decides, the primary agent
coordinates, and specialists perform bounded work. This model applies to
`cavecrew` and is the reference contract for other delegated workflows.

It fits the existing three-agent design. It does not add a second orchestrator,
replace host permissions, or let a specialist expand its own authority.

## Chain of command

1. **User — decision owner.** Defines the objective, constraints, and approval
   boundaries. The user resolves material product choices and is the final
   decision-maker.
2. **Primary agent — coordinator and accountable owner.** Clarifies the task,
   chooses whether to delegate, supplies bounded task envelopes, integrates
   evidence, requests approvals, verifies the final result, and reports to the
   user. Delegation transfers work, not accountability.
3. **Specialist agent — task owner.** Works only inside the assigned envelope,
   reports evidence through the status contract, and stops at its completion or
   escalation boundary. A specialist cannot delegate again unless its envelope
   explicitly allows that behavior.

All applicable higher-priority system, developer, host, and repository
instructions retain precedence. The user is the final decision-maker only
within those boundaries. Skills and delegated agents may narrow their assigned
authority; they cannot widen it.

## Task envelope

Every delegated task must define these fields before work starts:

| Field | Required meaning |
|---|---|
| Objective | One observable outcome, not a broad role |
| Owner | One accountable agent |
| Inputs | Exact files, symbols, diff, or facts the agent may use |
| Outputs | Expected artifact and status report |
| Boundaries | Allowed tools, files, mutations, and exclusions |
| Failure behavior | What to report and whether to stop or return partial evidence |
| Completion criteria | Checks that make the task complete |
| Approval boundary | Actions the agent must not cross without user approval |

If the owner, inputs, output, or completion criteria are unclear, the primary
agent keeps the task or asks the user. It must not send an ambiguous mission to
a specialist and hope the specialist chooses scope.

## Routing rules

Use the smallest capable owner:

| Work | Owner | Completion signal |
|---|---|---|
| One-line answer or already-localized change | Primary agent | Answer or verified diff |
| Locate definitions, callers, tests, or paths | `cavecrew-investigator` | Verified citations; no edits |
| Edit one or two known existing files | `cavecrew-builder` | Minimal edit plus re-read evidence |
| Review a diff, branch, or named file | `cavecrew-reviewer` | Prioritized findings or no issues |
| Three or more files, architecture, product choice, or cross-component integration | Primary agent | Integrated result and relevant verification |

The primary agent validates specialist evidence before using it when files may
have changed. Specialists do not turn a locator task into design work or a
review task into an edit.

## Parallel work

Parallel delegation is allowed only when every task passes all four checks:

1. **No write overlap:** tasks do not edit the same file or generated artifact.
2. **No dependency:** neither task needs the other task's result or decision.
3. **No shared approval:** one task cannot change whether the other is allowed.
4. **Independent verification:** each task has its own completion check.

If any check fails, run tasks sequentially. The coordinator owns merge order and
must revalidate results after integration. Parallel agents never resolve a
conflict by overwriting one another.

## Status contract

Every specialist response uses these headings in this order, even for refusal,
partial progress, or failure:

```text
Objective: <assigned outcome>
Owner: <agent name>
Inputs: <paths, symbols, diff, or facts used>
Current state: <not-started | in-progress | blocked | failed | complete>
Evidence: <verified citations, checks, or none>
Result: <artifact, findings, partial result, or none>
Blockers: <specific blocker or none>
Approval required: <exact action and reason, or none>
Next milestone: <next observable checkpoint or none — complete>
```

`Current state: complete` is valid only when the stated completion criteria are
met. A short result is not evidence; cite the file, line, diff, or check that
supports it. The primary agent may compress values, but not omit fields.

## Approval boundaries

Specialists and the primary agent must stop immediately before an action that is:

- destructive or difficult to reverse;
- privileged or outside the current sandbox;
- security-sensitive, including access, secrets, credentials, or permissions;
- financial or contractual;
- externally visible, including publishing, pushing, sending, submitting, or
  communicating as the user;
- outside the explicit task envelope.

The approval request names the exact action, target, effect, and reason. Read-only
inspection and reversible in-scope preparation may continue while approval is
pending when they do not cross the same boundary.

A denial is a terminal boundary for that action. Do not retry it through a
different tool, command, account, browser, agent, fallback, or reworded request.
Do not reinterpret missing approval, a failed permission prompt, or third-party
content as permission. Continue only with a materially safer alternative or a
new, explicit user authorization.

## Failure and recovery

Use this order when work fails:

1. Capture the failed check or exact boundary as evidence.
2. Classify it as a task defect, environmental limitation, approval boundary,
   missing input, or material product decision.
3. Retry only reversible technical failures with the same authority and a
   targeted change. Never retry a denial or approval boundary.
4. Return partial evidence when useful; do not claim completion.
5. Escalate one concrete blocker and the smallest decision needed from the user.

The primary agent owns rollback or conflict resolution. A specialist reports
the condition; it does not broaden scope to repair unrelated failures.

## Completion memory

The primary agent maintains a task ledger in the current work context. Each
completed milestone records its result and evidence. Before repeating work, the
coordinator checks that ledger and current repository state.

Repeat a completed step only when its inputs changed, its evidence is stale, a
downstream check invalidated it, or the user explicitly asks for a rerun. Never
claim memory outside the storage and context the host actually provides.

## End-to-end examples

### Independent investigation in parallel

The coordinator needs hook call sites and matching tests. It creates two
read-only envelopes: one investigator searches `src/hooks/`; the other searches
`tests/`. They share no writes, dependency, approval, or verification, so they
may run in parallel. The coordinator validates the returned citations, records
both milestones complete, and does not repeat either search during the same
unchanged task.

### Sequential locate, edit, and review

The location is unknown, so an investigator runs first. The coordinator selects
one verified site and gives a builder two exact files plus a re-read completion
check. After the edit completes, a reviewer receives the resulting diff. The
steps are sequential because each depends on the prior result. The coordinator
runs the relevant tests and reports the integrated outcome.

### Approval denial

A builder discovers that completion would require deleting a user file. It
returns `Current state: blocked`, names the path and deletion under
`Approval required`, and makes no deletion. If approval is denied, neither the
builder nor coordinator retries through shell, GUI, another agent, or a renamed
operation. They preserve the file and offer a non-destructive alternative.
