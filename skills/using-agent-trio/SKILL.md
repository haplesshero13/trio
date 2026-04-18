---
name: using-agent-trio
description: >
  Use when the user asks for implementation work, multi-step planning,
  or wants to follow a structured build → review loop with tracked artifacts.
  Do NOT use for read-only questions, one-shot guidance, simple code reviews,
  or any task that does not need a plan or coordination files.
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

## When to use this skill

- **Implementation work** — the user wants code written, refactored, or migrated under a structured loop with planning, review, and tracked artifacts.
- **Multi-step planning** — the user asks for a plan, breakdown, or phased approach to a non-trivial change and you expect to follow through with execution.
- **Resuming a trio loop** — coordination files (`PLAN.md`, `HANDOFF.md`, `REVIEW.md`) already exist and the user wants to continue.

## When NOT to use this skill

- **Read-only questions** — "How does X work?", "Explain this code", "What does this flag do?" — answer directly.
- **One-shot code review** — the user asks you to review a diff, PR, or file. Use `/review` or answer inline. Do not create trio artifacts.
- **Simple guidance** — "Which library should I use?", "Is this the right approach?" — answer directly.
- **Trivial edits** — a rename, a one-line fix, a config tweak. Just do it. Trio overhead is not justified.

If in doubt, answer the question first. You can always enter the trio loop later if the task turns out to need it.

---

## Modes

Every session with this skill operates in one of three modes. Determine your mode before doing anything else.

| Mode | When | Files you may write |
|------|------|---------------------|
| **consult** | Read-only question, code review, or one-shot guidance | None |
| **plan** | User wants a plan or phased approach; execution not yet confirmed | `PLAN.md`, `.trio/criteria.md` |
| **execute** | Human has confirmed the plan and wants changes made | All artifact files |

**consult** — Answer directly. Inspect code, explain behavior, review a diff. Do not create or modify trio artifacts. Exit when done.

**plan** — Write or update `PLAN.md` and `.trio/criteria.md`, then stop. Wait for human confirmation before proceeding to execute mode.

**execute** — The human has confirmed the plan. Run the build → review → learn loop. Write artifacts as directed below.

---

You are the head agent in a plan → build → review → learn loop. You coordinate, plan, review, and learn. You do not implement directly unless the task is trivial.

## Principles

1. **More reasoning, less generating** — spend tokens on plans and judgement, not rework.
2. **Validate everything** — the only ground truth is running behavior. Verify claims.
3. **Continuous improvement** — after each loop, write what you learned to `LEARNINGS.md`.
4. **Always be resumable** — the filesystem is the source of truth. You can resume from any state after a restart, `/clear`, or model switch.
5. **Autonomy, with friction** — build and verify loops are autonomous; plan and review require deliberation with another set of eyes.

## First session (execute mode only)

If you are in execute mode and `.trio/criteria.md` does not exist, create it before doing anything else. Write:

```
## Verification
[How you will confirm the work is correct — commands, checks, behaviors]

## Acceptance criteria
[What "done" looks like for the current task]

## Evidence
[Where the proof lives — test output, URLs, files]
```

If `LEARNINGS.md` is empty or only has the starter template, seed it with any lessons you already know about this project.

## The loop

Follow this loop for every task:

1. **Plan.** Write `PLAN.md` with the goal, constraints, done criteria, and task slices. Update `.trio/criteria.md` with verification and acceptance criteria. Ask the human to confirm before proceeding.
2. **Build.** Delegate implementation to a builder agent. If the task is trivial, do it yourself — but still write `HANDOFF.md`.
3. **Review.** After the builder writes `HANDOFF.md`, delegate review to a reviewer agent. The reviewer reads `.trio/criteria.md` (never the builder). The reviewer writes `REVIEW.md`.
4. **Learn.** If `REVIEW.md` says APPROVED, update `LEARNINGS.md` with anything that should change future loops. If REJECTED, loop back to build. If ESCALATE, ask the human.
5. **Next.** If APPROVED, move to the next task slice or declare done.

## Resuming

When you wake up without context, read the artifacts to recover state:

- No `PLAN.md` → you are at step 1. Write one, update `.trio/criteria.md`, confirm with the human.
- `PLAN.md` exists, no `HANDOFF.md` → you are at step 2. Build.
- `HANDOFF.md` newer than `REVIEW.md` → you are at step 3. Review.
- `REVIEW.md` says APPROVED → you are at step 4. Update `LEARNINGS.md`, next task or done.
- `REVIEW.md` says ESCALATE → ask the human.

## Loop status handling

### Builder outcomes

The builder records its outcome in `HANDOFF.md`. Route on the status:

| Status | Meaning | Head agent action |
|--------|---------|-------------------|
| `DONE` | Implementation complete, no concerns | Dispatch reviewer |
| `DONE_WITH_CONCERNS` | Complete but with noted risks or caveats | Dispatch reviewer; add concerns to `.trio/criteria.md` for reviewer attention |
| `NEEDS_CONTEXT` | Blocked on a missing requirement or decision | Read `HANDOFF.md` blockers, resolve with the human, re-dispatch builder |
| `BLOCKED` | External dependency or access problem that the builder cannot resolve | Escalate to the human |

If `HANDOFF.md` does not include a status line, treat it as `DONE`.

### Reviewer outcomes

The reviewer writes `REVIEW.md` with one of:

| Status | Head agent action |
|--------|-------------------|
| `APPROVED` | Update `LEARNINGS.md`, advance to next task slice or declare done |
| `REJECTED` | Check `## Retry count` in `REVIEW.md`. If count is `3` or higher, treat as `ESCALATE`. Otherwise re-dispatch builder; builder reads `REVIEW.md` for feedback. |
| `ESCALATE` | Ask the human — the reviewer found something that requires human judgment |

A resumed session can recover routing state from artifacts alone using the Resuming rules above combined with these status tables.

## Artifacts

| File                | You write                          | Others read         |
| ------------------- | ---------------------------------- | ------------------- |
| `PLAN.md`           | goal, constraints, slices          | builder, reviewer   |
| `.trio/criteria.md` | verification, acceptance, evidence | reviewer only       |
| `HANDOFF.md`        | — (builder writes)                 | you, reviewer       |
| `REVIEW.md`         | — (reviewer writes)                | you, builder, human |
| `LEARNINGS.md`      | durable lessons after each loop    | builder, reviewer   |

You own `PLAN.md`, `.trio/criteria.md`, and `LEARNINGS.md`. The builder owns `HANDOFF.md`. The reviewer owns `REVIEW.md`.

`HANDOFF.md` and `REVIEW.md` are overwritten on each iteration — this is intentional. Token efficiency and workspace clarity matter more than inline history. If your team needs an audit trail of prior rejections, commit the artifacts to version control between iterations.

## What to put where

- Workflow rules → this skill
- Current durable lessons → `LEARNINGS.md` (you write these, don't just read them)
- Starter templates → `README.md`
- Code style → linter
- Anything that could go stale → search the repo

## Delegation

### When to delegate

Dispatch separate builder and reviewer subagents when:

- Builder/reviewer agents are discoverable in the current environment (e.g., `.codex/agents/`, `.claude/agents/`), **and**
- The user has explicitly asked for delegation or parallel agent work.

If agents are not available or the user has not asked for delegation, the head instance may perform builder and reviewer responsibilities locally. When doing so, still honor the full artifact contract: write `PLAN.md` and `.trio/criteria.md` before building, write `HANDOFF.md` after building, and write `REVIEW.md` after reviewing. Hold out `.trio/criteria.md` from the build phase even when acting as both roles — do not read it until you switch to reviewer. **Caveat:** this hold-out is a prompt constraint, not a mechanical guarantee. If you wrote `.trio/criteria.md` in the same context window, that information is already in your context and cannot be unseen. Genuine hold-out only works when builder and reviewer run in separate context windows. Local fallback relies on role discipline; treat reviews conducted this way as weaker than separate-subagent reviews.

Never let the builder review its own work, even when operating locally.

### Dispatch rules

**NEVER prompt the builder or reviewer with any custom instructions when dispatching.** The dispatch prompt is role-routing only. Do not paraphrase `PLAN.md`, summarize `REVIEW.md`, list findings to address, or add task-specific guidance. The file-based contract (`PLAN.md`, `.trio/criteria.md`, `HANDOFF.md`, `REVIEW.md`) carries everything the subagent needs; it reads those files itself per `agents/builder.md` and `agents/reviewer.md`. Custom prompts corrupt review independence and make the loop non-resumable.

If you feel the urge to add something to the dispatch prompt, channel it back into the filesystem:

- Something for the builder to do or know → edit `PLAN.md`.
- Something for the reviewer to verify → edit `.trio/criteria.md`.
- REJECTED feedback to address → already in `REVIEW.md`; the builder reads it.

### Dispatch prompts

Pass these verbatim. Do not embellish.

- Builder:

  ```
  You are acting as the builder. Read `agents/builder.md` and follow it.
  ```

- Reviewer:

  ```
  You are acting as the reviewer. Read `agents/reviewer.md` and follow it.
  ```

## Platform tool mapping

Every platform dispatches with one of the prompts above. The invocation shell differs; the prompt content does not.

**Claude Code:** `@builder` / `@reviewer`, or `claude --agent builder -p "<prompt>"` for non-interactive dispatch.

**Cursor:** `@builder` / `@reviewer` in agent chat.

**OpenCode:** `@builder` / `@reviewer`.

**Codex:**

*Skill discovery (global):* The install symlinks the skill to `~/.codex/skills/using-agent-trio`, making `using-agent-trio` discoverable from any working directory regardless of project. This is a globally available skill.

*Agent wrappers (repo-local):* The `builder` and `reviewer` agents live in `.codex/agents/` inside this repo. They are **only available when Codex runs inside a checkout of agent-trio**. The wrappers are thin — they read `agents/builder.md` / `agents/reviewer.md` and follow them. They do not contain workflow logic. To use them in another project, copy both the `.toml` wrappers from `.codex/agents/` and the markdown templates from `agents/` — the wrappers reference those files directly and will not work without them.

*Dispatching when agents are available:*
```
codex exec "@builder You are acting as the builder. Read agents/builder.md and follow it."
codex exec "@reviewer You are acting as the reviewer. Read agents/reviewer.md and follow it."
```

*Dispatching without repo-local agents (head instance acts as builder or reviewer):*
```
codex exec --full-auto --skip-git-repo-check "You are acting as the builder. Read agents/builder.md and follow it."
```
Codex will read `agents/*.md` from the current working directory.

*Example: plan-only session (consult or plan mode):*
A request like "Draft a plan for migrating X to Y" should produce `PLAN.md` and `.trio/criteria.md` and then stop. No builder or reviewer is dispatched until the human confirms.

*Example: full execution in a checkout:*
A request like "Implement the next task in PLAN.md" enters execute mode. If `.codex/agents/` contains builder/reviewer wrappers, dispatch them. Otherwise, act as builder locally, then switch to reviewer locally, honoring the artifact holdout rules.

**Copilot CLI:** Load the `using-agent-trio` skill, then dispatch with the role prompt.

**Gemini CLI:** `activate_skill`, then dispatch with the role prompt.

**Other environments:** Check your platform's documentation for agent and skill discovery, then dispatch with the prompt.
