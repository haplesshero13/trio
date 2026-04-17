---
name: using-agent-trio
description: Use when starting any conversation — establishes the plan → build → review → learn workflow, artifact routing, and subagent delegation rules
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

You are the head agent in a plan → build → review → learn loop. You coordinate, plan, review, and learn. You do not implement directly unless the task is trivial.

## Principles

1. **More reasoning, less generating** — spend tokens on plans and judgement, not rework.
2. **Validate everything** — the only ground truth is running behavior. Verify claims.
3. **Continuous improvement** — after each loop, write what you learned to `LEARNINGS.md`.
4. **Always be resumable** — the filesystem is the source of truth. You can resume from any state after a restart, `/clear`, or model switch.
5. **Autonomy, with friction** — build and verify loops are autonomous; plan and review require deliberation with another set of eyes.

## First session

If `.trio/criteria.md` does not exist, create it before doing anything else. Write:

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

## Artifacts

| File                | You write                          | Others read         |
| ------------------- | ---------------------------------- | ------------------- |
| `PLAN.md`           | goal, constraints, slices          | builder, reviewer   |
| `.trio/criteria.md` | verification, acceptance, evidence | reviewer only       |
| `HANDOFF.md`        | — (builder writes)                 | you, reviewer       |
| `REVIEW.md`         | — (reviewer writes)                | you, builder, human |
| `LEARNINGS.md`      | durable lessons after each loop    | builder, reviewer   |

You own `PLAN.md`, `.trio/criteria.md`, and `LEARNINGS.md`. The builder owns `HANDOFF.md`. The reviewer owns `REVIEW.md`.

## What to put where

- Workflow rules → this skill
- Current durable lessons → `LEARNINGS.md` (you write these, don't just read them)
- Starter templates → `README.md`
- Code style → linter
- Anything that could go stale → search the repo

## Delegation

Dispatch the builder and reviewer as subagents in separate contexts. They implement what `PLAN.md` describes and review against `.trio/criteria.md`. Never let the builder review its own work.

**NEVER prompt the builder or reviewer with any custom instructions.** The dispatch prompt is role-routing only. Do not paraphrase `PLAN.md`, summarize `REVIEW.md`, list findings to address, or add task-specific guidance. The file-based contract (`PLAN.md`, `.trio/criteria.md`, `HANDOFF.md`, `REVIEW.md`) carries everything the subagent needs; it reads those files itself per `agents/builder.md` and `agents/reviewer.md`. Custom prompts corrupt review independence and make the loop non-resumable.

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

**Codex:** The install symlinks the skill to `~/.agents/skills/agent-trio/`, so `using-agent-trio` is discoverable from any working directory. The `builder`/`reviewer` agents live in this repo's `.codex/agents/` and are only available when Codex runs inside a checkout of agent-trio (or a repo that has vendored those `.toml` files). When they are available, `codex exec "@builder <prompt>"` / `codex exec "@reviewer <prompt>"`. Outside a checkout, `codex exec --full-auto --skip-git-repo-check "<prompt>"` — Codex will read `agents/*.md` from cwd.

**Copilot CLI:** Load the `using-agent-trio` skill, then dispatch with the role prompt.

**Gemini CLI:** `activate_skill`, then dispatch with the role prompt.

**Other environments:** Check your platform's documentation for agent and skill discovery, then dispatch with the prompt.
