# 15-Minute AI PR Review Checklist

> **New here?** Start with the [worked example](example_agent_pr_walkthrough.md) — a synthetic agent PR reviewed section by section.

Use this when a pull request was substantially generated or modified by an AI coding agent.

This is not a replacement for human accountability. It is a fast review pass for the failure modes that show up when AI-generated code looks polished before anyone has inspected it closely.

## 0. The 60-Second Context Check

Before reviewing code, answer these three questions:

1. What was the agent asked to do?
2. Which files did it change?
3. What would break if the agent misunderstood the task?

If you cannot answer all three, stop and request more context before reviewing.

## 1. Scope Drift - 2 Minutes

Look for changes outside the requested task.

Check:

- Files unrelated to the original request
- Auth, billing, permissions, migrations, secrets, deployment config, or data deletion
- Large rewrites where a small change was requested
- Behavior changes without matching tests
- Test changes without a clear behavior change

Slow down if the PR is broader than the task.

## 2. Behavior And Edge Cases - 3 Minutes

Read the core changed path.

Ask:

- Does the code actually implement the requested behavior?
- What happens with empty, malformed, missing, duplicate, or unauthorized input?
- Are previous behaviors preserved?
- Are retries, timeouts, and failure paths handled?
- Are user-visible errors understandable?

Slow down if the code only handles the happy path.

## 3. Security And Data Risk - 3 Minutes

Scan for:

- New or changed authorization checks
- Permission bypasses
- Trust in client-provided data
- Secret or token logging
- Unsafe file or path handling
- Raw SQL or query construction
- Cross-tenant data access
- Changed CORS, cookies, sessions, or token handling
- PII in logs, prompts, fixtures, or tests

Slow down if security-adjacent code changed without explicit tests.

## 4. Test Reality - 2 Minutes

Do not just check whether tests exist.

Ask:

- Would the tests fail if the important behavior were removed?
- Is there at least one negative or edge-case test?
- Is there a regression test for the bug or request?
- Did the agent delete, weaken, skip, or over-mock tests?
- Are tests asserting behavior, or merely matching the current implementation?

Slow down if tests look performative.

## 5. System Fit - 2 Minutes

Ask what the agent may not have seen.

Check:

- Runtime config
- Environment variables
- External APIs
- Generated files
- Schema, client, or API-doc updates
- Deploy order
- Backward compatibility

Slow down if the code compiles locally but changes a contract elsewhere.

## 6. Agent Self-Review - 2 Minutes

Paste this into the coding agent before final human approval:

```text
Review your own changes as if they were submitted by another developer. Focus only on likely defects, security risks, missed edge cases, stale tests, and scope drift. Do not praise the code.

Return:
1. The three highest-risk assumptions in this PR.
2. Any changed behavior not covered by tests.
3. Any security, auth, data, migration, or compatibility risk.
4. The smallest additional check or test that would most reduce risk.
```

Slow down if the agent identifies a real risk the PR has not addressed.

## Merge Decision

Merge only if:

- The scope matches the task.
- The core behavior is clear.
- Security and data risks are absent or tested.
- Tests protect the intended behavior.
- Integration impacts are understood.

Revise, split, or escalate if:

- The PR touched high-risk code.
- The diff is broader than requested.
- The agent cannot explain edge cases.
- Tests do not prove meaningful behavior.
- Review depends on "the agent probably handled it."

## Reviewer Note Template

```text
AI PR review result:

- Scope drift: none / minor / material
- Security or data risk: none / needs follow-up
- Test confidence: low / medium / high
- Integration concern: none / needs follow-up
- Decision: merge / revise / split / escalate

Main reason:
```

## Reminder

AI-generated code is still your code once it merges. The checklist exists to make that responsibility easier to discharge, not to move it onto the tool.
