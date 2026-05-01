# Learnings

Keep only lessons that change how the next update is planned, built,
reviewed, or validated. Rewrite, merge, and prune continuously.

Source control will track versions of this document, so keep it fresh.

## Current learnings

### One copy per concept

The workflow contract lives in `skills/using-agent-trio/SKILL.md` — nowhere else. Agent definitions live in `agents/` — nowhere else. Wrappers are one-line pointers, not copies. Duplication creates drift and confusion about which version is authoritative.
**Why:** Multiple copies of builder/reviewer content led to a 5-way duplication problem that the reviewer flagged. Collapsing to one copy eliminated an entire class of bugs.
**How to apply:** Never duplicate content into wrappers. If a wrapper needs more than "Read X and follow it exactly," the content belongs in the file.

### Skill must be imperative, not descriptive

The original `trio-agents/AGENTS.md` described the loop in third person. The reviewer flagged that this meant the plugin could load and still do nothing — the agent reads a description but doesn't know what to do. Rewriting as imperative ("You are the head agent. Write .trio/plan.md. Delegate to builder.") made the skill effective.
**Why:** A plugin that loads but doesn't change agent behavior is dead weight.
**How to apply:** When writing agent instructions, address the agent directly with commands, not descriptions of how things work.

### Plugin manifests are per-platform, not generic

`.claude-plugin/plugin.json` does not support an `agents` field — it caused validation to fail. Each platform discovers agents differently (Claude Code from `.claude/agents/`, Codex from `~/.codex/skills/`). Don't assume cross-platform consistency in manifest schemas.
**Why:** A single validation failure blocks the entire plugin from loading.
**How to apply:** Run `claude plugin validate .` after any manifest change. Don't add fields that aren't in the platform's schema.

### Hooks with no runtime purpose are dead code

The original `hooks/` directory had a `run-hook.cmd` that was a documented no-op. The real injection happened via the plugin system. Three files and two manifest entries for something that does nothing violates the minimalism goal.
**Why:** Dead code confuses contributors and bloats the repo.
**How to apply:** Don't ship forward-compatible placeholders. Add the hook when it actually does something.

### AGENTS.md is for developers, README is for consumers

AGENTS.md describes how to modify this repo. README describes why you'd use it and how to install it. Mixing setup-agent instructions into AGENTS.md made it unclear who the audience was.
**Why:** The repo is now a skill/plugin repo, not just a template. The audience for "how to work on this" is different from "how to use this."
**How to apply:** AGENTS.md = developer guide. README = consumer-facing install and rationale.

### Codex works as a peer reviewer via `codex exec`

A one-shot `codex exec --full-auto --skip-git-repo-check` with a reviewer prompt (read the loop artifacts, verify claims by running commands, write .trio/review.md) produces a real adversarial review. It caught a criterion/repo mismatch the same-model reviewer would likely have missed.
**Why:** Adversarial review across model families surfaces a different class of findings than same-family review.
**How to apply:** For loops where the stakes or scope justify it, dispatch Codex (or any peer runtime) as the reviewer instead of `@reviewer`. The head keeps owning PLAN, criteria, and LEARNINGS.

### Skill triggers need negative cases, not just positive ones

The original `using-agent-trio` description said "Use when starting any conversation," which caused it to grab read-only and one-shot sessions where trio overhead adds no value. The fix was to rewrite the description with explicit "when not to use" cases and add a `## Modes` section that gates artifact creation by mode.
**Why:** A skill with only a positive trigger fires too broadly. Without explicit exclusions, agents apply the workflow to questions that just need a direct answer.
**How to apply:** Every skill description should include at least one "do NOT use for X" clause. When the skill governs file creation, add a mode table that maps intent → allowed files so the agent can gate artifact writes before doing anything else.

### Delegation should be conditional, not automatic

Claiming "dispatch the builder and reviewer as subagents" without qualification implies delegation always happens, which is false in environments without agent discovery (e.g., outside a checkout). The REJECT caught this; the fix was to add explicit conditions: agents discoverable **and** user explicitly asked.
**Why:** Unconditional delegation language creates false expectations and fails silently when agents aren't available.
**How to apply:** Write delegation rules as conditionals ("dispatch when X and Y; otherwise do Z locally"). Always describe the local fallback so the loop doesn't stall.

### Vendored wrappers are incomplete without their templates

`.codex/agents/*.toml` wrappers hard-code paths to `agents/builder.md` and `agents/reviewer.md`. If someone copies only the `.toml` files into their own repo, the agents fail to load. The docs initially said "copy the `.toml` files" without mentioning the markdown templates.
**Why:** Thin wrappers are only thin if the reader knows what they depend on.
**How to apply:** When documenting how to vendor agent wrappers, always list every file that must be copied, including the files the wrappers reference.

### Codex skill install path is ~/.codex/skills/, not ~/.agents/skills/

The original install instructions in README.md and .codex/INSTALL.md documented `~/.agents/skills/` as the Codex skills directory. That path does not exist and is never created by Codex. The correct path, confirmed by examining `~/.codex/skills/.system/` where Codex stores built-in skills, is `~/.codex/skills/`. The incorrect path silently caused all smoke tests of global skill discovery to fail.
**Why:** Using a wrong path in install docs means the skill is never discovered globally, making the entire install story non-functional. The smoke test uncovered it because the throwaway-directory test requires the symlink to actually work.
**How to apply:** Verify install paths by checking where the runtime already writes its own files (e.g., `.system/` directory) rather than guessing. Always run the install steps from scratch in a fresh environment as part of smoke testing.

### Agent instructions must be internally consistent with SKILL.md routing

`SKILL.md` defined routing logic that depended on builder status keywords and reviewer retry counts, but neither `builder.md` nor `reviewer.md` had instructions to produce those values. The routing in `SKILL.md` was effectively dead code. The fix was to add `## Status` to the builder's HANDOFF format and make the reviewer read its own prior `.trio/review.md` for the retry count before overwriting it.
**Why:** A head-agent routing table is only as good as the data the subagents actually write. If you add routing logic to `SKILL.md`, always trace back to whether the builder or reviewer is instructed to emit the values that logic depends on.
**How to apply:** When adding a new routing branch to `SKILL.md`, immediately check `builder.md` and `reviewer.md` to confirm they produce the required output. Treat them as a three-file system, not independent documents.

### Hold-out guarantees require architectural separation, not just instructions

The local fallback path instructs the head agent to "not read `.trio/criteria.md` until the reviewer phase." This is a prompt constraint, not a mechanical guarantee — if the agent wrote the criteria in the same context window, it already knows the contents. True holdout only works when builder and reviewer run in genuinely separate contexts (different subagent dispatches).
**Why:** Misrepresenting a prompt constraint as a mechanical guarantee sets false expectations and may cause teams to rely on the local fallback for adversarial review when it cannot deliver it.
**How to apply:** Document the difference explicitly. If the stakes of a task require genuine review independence, delegate to a separate subagent — don't use the local fallback path.

### Smoke tests must run from scratch, not be inherited from prior builds

The previous build cycle's .trio/handoff.md carried forward Tests 4 and 5 as "evidence" without re-running them. The reviewer caught that Tests 1–3 lacked actual command output, and that Test 5 bypassed the head-agent path entirely. The fix required re-running all five tests with real commands and real output.
**Why:** "Carried forward from previous build" is not evidence. The reviewer cannot verify claims that have no command transcript attached.
**How to apply:** Every smoke test in .trio/handoff.md must include the actual command run and the actual output observed. Never summarize or inherit. If re-running a test is impractical, say so explicitly in Blockers — don't quietly forward a prior result.

### Criteria and repo shape drift together

When you delete a concept from the repo, audit `.trio/criteria.md` in the same loop. A stale criterion (e.g. "grep for trio-agents returns nothing across all .md") can fail against a repo that is otherwise correct, because .trio/learnings.md intentionally keeps historical mentions.
**Why:** The reviewer runs the criterion exactly as written; over-broad criteria cause false REJECTs.
**How to apply:** When retiring a concept, scope the matching criterion (and its grep) to where the concept should no longer appear, not everywhere.

### OpenCode npm installs and checkout installs need separate docs

OpenCode can load a published npm plugin from the `plugin` array, while local/git-checkout usage is better represented by a shim in `~/.config/opencode/plugins/` that imports the checkout's plugin file. Treating a git URL as if it were the same as an npm plugin created an install story that did not match the current OpenCode CLI/docs.
**Why:** Users with a normal `~/.config/opencode/opencode.json` keep model/provider config there, but a local plugin shim is discovered from the plugins directory, not from a fake npm package entry.
**How to apply:** Document both paths explicitly: `plugin: ["agent-trio"]` only after npm publish; global shim for local or git-checkout installs. Verify with `opencode debug config`, `opencode debug skill`, and `opencode agent list`.

### Codex plugin install is marketplace registration plus `/plugins`

Codex CLI exposes `codex plugin marketplace add`, not a direct `codex plugin install` command. A repo can still be first-class installable by providing `.codex-plugin/plugin.json` and `.agents/plugins/marketplace.json`; after adding the marketplace, the user installs the plugin from `/plugins`.
**Why:** Symlinking into `~/.codex/skills` works as a fallback, but it bypasses Codex's native plugin UI and does not match the current user-facing install flow.
**How to apply:** For Codex plugin repos, keep `.codex-plugin/plugin.json` at the plugin root, keep marketplace metadata in `.agents/plugins/marketplace.json`, document `codex plugin marketplace add <source>` followed by `/plugins`, and validate local changes with `HOME=/tmp/... codex plugin marketplace add .`.
