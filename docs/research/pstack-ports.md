# pstack ports to other harnesses

- **Checked 2026-10-13.**
- **Compared against:** upstream pstack 0.15.20, cursor/plugins@`1b6d44f` ([plugin.json:4](https://github.com/cursor/plugins/blob/1b6d44ff26d4e242f4e8f68c391e6d2f7ca16def/pstack/plugin.json#L4)). The local copy is `/tmp/pstack-src/plugins/pstack/`, written `U/` below. Upstream main has since moved to 0.15.21 (`997ffc7`, 2026-10-12 17:53 UTC, `gh api 'repos/cursor/plugins/commits?path=pstack&per_page=5'`).
- **Similarity method:** the share of upstream's 52 `skills/*/SKILL.md` files that have a counterpart in the port scoring ≥ 0.90 on `difflib.SequenceMatcher(None, a, b, autojunk=False).ratio()` over lines. A counterpart is the same folder name, or one of p3's renames (`p3-mode`, `setup-p3`, `p3-help`). A missing file scores 0. The script is `/tmp/pstack-compare-scripts/skill_similarity.py`, run with `python3 -I`. Its output is in `similarity-output.txt` and `similarity-own-pin.txt` in the same folder.
- **Paths:** `<label>/` is the clone at `/tmp/pstack-ports/<owner>_<repo>/repo/`, at the SHA in the table. For example, `pstack-claude/` is `/tmp/pstack-ports/michael-denyer_pstack-claude/repo/`. The same path is on GitHub at `https://github.com/<owner>/<repo>/blob/<SHA>/<path>`. "Docs" means https://code.claude.com/docs/en/skills, fetched with `curl -sSL https://code.claude.com/docs/en/skills.md`.
- **GitHub numbers** come from these commands: `gh repo view <o/r> --json stargazerCount,pushedAt,description`, `gh issue list -R <o/r> --state open --limit 200 | wc -l`, `git -C <clone> rev-parse --short HEAD` and `gh api repos/<o/r>/commits/HEAD`.

## In one paragraph

Eight public repos port or mirror pstack. Two target Claude Code: pstack-claude, and open-pstack, which grew out of it. Two target Codex: pstack-codex and pstack-for-codex. One targets T3 Code (p3-stack), one is a mirror (backnotprop), and two are old and have drifted (potetos, oh-my-pstack). pstack-claude is still the best maintained. It has the most stars and few open issues, and its remote already tracks 0.15.21. Every port scores lower under the stated method than in the earlier table. Part of the gap is lag: against the upstream commit each port pins, scores rise 6 to 38 points. The real design split is routing. Upstream never routes on its own. `poteto-mode` is a Cursor mode the user turns on, and 51 of 52 skills are user-only. The two Claude Code ports replace that with a SessionStart hook that tells the model to enter poteto-mode, and they make almost every skill auto-invocable. The Codex ports and p3-stack keep routing explicit.

## Comparison

| Port (clone SHA) | For | Upstream pin | Similar: old → now (vs own pin) | Stars / open issues / last push | Hooks and mandates |
|---|---|---|---|---|---|
| michael-denyer/pstack-claude `1bfdab4` (remote `058a348`) | Claude Code, Codex, Copilot, Pi, OpenCode, Gemini, Prime Agent | 0.15.15 `df58112`. Remote: 0.15.21 | 78% → 33% (69%) | 1878 / 2 / 2026-10-13 | SessionStart mandate, ~246 tokens, has an off switch |
| ericlitman/open-pstack `a41d063` (remote `4f2b9b0`) | Claude Code + Codex | 0.15.17 `978ca6e`. Remote: 0.15.20 | 90% → 65% (75%) | 408 / 19 / 2026-10-12 | SessionStart mandate, ~315 tokens, no off switch |
| ScriptedAlchemy/pstack-codex `44e3d19` | Codex | 0.15.17 `11713c6` | 89% → 56% (62%) | 70 / 1 / 2026-10-12 | None. 53 of 54 skills explicit-only |
| Aqua-123/pstack-for-codex `3d2770d` | Codex | 0.15.15 `ccb5507` | 71% → 10% (18%) | 82 / 0 / 2026-10-09 | 4 hook events, silent until the user types `$poteto-mode` |
| uzairansaruzi/p3-stack `09909eb` | T3 Code | Not recorded. Base looks like `4e5b1cf` (0.15.10) | 76% → 33% (71%) | 146 / 1 / 2026-10-05 | None. `p3-mode` is user-only |
| backnotprop/pstack `fce11a3` | Mirror in Cursor layout | 0.15.20 | 99% → 92% | 1685 / 1 / 2026-10-12 | None, same as upstream |
| TheOnlyFusionCube/potetos-for-everyone `f494e1a` | Claude Code, Codex, Gemini, Pi | 0.15.0, baseline `71ed0d1` | 13% → 0% | 11 / 0 / 2026-09-10 | SessionStart mandate, ~358 tokens, no off switch |
| shrimpwtf/oh-my-pstack `fc0eabd` | Pi, OMP, Claude Code, Codex, OpenCode | 0.14.3, `bdf7aa3` | 76% → 13% | 32 / 2 / 2026-09-01 | None |

Where the cells come from:
- **For:** each repo's GitHub description, from the `gh repo view` command above.
- **Pins:** `pstack-claude/tools/upstream.json:7` and `CHANGES.md:41`, `open-pstack/UPSTREAM.md:11-12`, `pstack-codex/UPSTREAM.json:4-5`, `pstack-for-codex/upstream.lock.json:6-7`, `backnotprop/plugin.json:4`, `potetos/.claude-plugin/plugin.json:4` and `UPSTREAM.json:6`, `oh-my-pstack/package.json:3` ("0.14.3-pi.1") and `upstream.lock.json:5`.
- **p3-stack's base:** p3-stack records no version; its README and AGENTS.md only link to `main`. It ships `p3-help`, its port of `poteto-help`. Upstream added that skill in `4e5b1cf`, which is the last upstream pstack commit before p3's head commit (2026-10-05 07:40 UTC). Command: `gh api 'repos/cursor/plugins/commits?path=pstack&until=2026-10-05T07:40:27Z'`.
- **Remote pins:** `gh api 'repos/michael-denyer/pstack-claude/contents/tools/upstream.json?ref=058a348'` gives `997ffc7`. `gh api 'repos/ericlitman/open-pstack/contents/UPSTREAM.md?ref=4f2b9b0'` gives 0.15.20. Both hook files are unchanged at those heads.
- **Own-pin scores:** each pinned upstream was fetched with `gh api repos/cursor/plugins/tarball/<sha>` into `/tmp/pstack-compare-scripts/data/`. Ports without a remote SHA in the table are at their remote HEAD.

## Hooks and mandates

- **Upstream.** No hooks. The manifest lists only skills and agents (`U/.cursor-plugin/plugin.json:29-30`). `poteto-mode` sets `disable-model-invocation: true`, plus Cursor's sticky-mode keys `mode: true` and `reminder` (`U/skills/poteto-mode/SKILL.md:4-8`). 51 of 52 skills are user-only, and only `setup-pstack` is listed for the model (`/tmp/pstack-compare-scripts/skill_listing_size.py`). The user turns the mode on.
- **pstack-claude.** A SessionStart hook runs on startup, resume, clear and compact (`plugins/pstack/hooks/hooks.json:3-13`). It prints `hooks/session-start-context.md`: 983 chars, about 246 tokens. The text is an `<EXTREMELY_IMPORTANT>` block. It says to invoke `pstack:poteto-mode` when a task touches more than one file, involves a design choice, or is a bug of unknown cause or a perf issue (`:4-8`). It also says CLAUDE.md, AGENTS.md and direct requests take precedence (`:14`).
  - **Off switch:** the line `session hook: off` in `${CLAUDE_CONFIG_DIR:-~/.claude}/pstack-models.md` makes the script exit before it prints (`hooks/session-start.sh:8,32-34`). `/setup-pstack` asks about it and writes that line (`skills/setup-pstack/SKILL.md:44-46,80`).
  - **Other harnesses:** the same hook runs on Codex once the user trusts it through `/hooks` (`skills/poteto-mode/references/codex-tools.md:57`). Copilot adds a PreToolUse hook (`hooks/copilot-hooks.json:14-25`). Pi puts the mandate in the system prompt (`skills/poteto-mode/references/pi-tools.md:67`).
  - **Skill listing:** all 58 skills are model-invocable. Their names and descriptions come to 13,474 chars, about 3.4k tokens, before Claude Code trims the listing.
- **open-pstack.** A SessionStart hook runs on startup, clear and compact (`plugins/pstack/hooks/hooks.json:3-13`). The script is one unconditional `exec cat` (`hooks/session-start:8`), so the only way to turn it off is to disable the plugin. It injects 1,259 chars, about 315 tokens, with a wider trigger: "any non-trivial engineering task" (`hooks/session-start-context.md:4`). It tells dispatched subagents to ignore it (`:8`). The hook is Claude Code only; Codex gets none (`docs/reference.md:88`). 56 of 59 skills are model-invocable, about 3.1k tokens of listing. The port drops upstream's `disable-model-invocation` lines on purpose, because "the flag blocks that route on Claude Code" (`UPSTREAM.md:20`).
- **potetos.** An unconditional SessionStart `cat` that injects 1,433 chars, about 358 tokens (`hooks/hooks.json:3-15`).
- **pstack-codex.** No hooks (`README.md:33`). 53 of 54 skills set `allow_implicit_invocation: false` (`plugins/pstack/skills/*/agents/openai.yaml:5`). The one exception, `setup-pstack`, has no policy file.
- **pstack-for-codex.** Four hook events: SessionStart (on resume and compact), UserPromptSubmit, SessionEnd and SubagentStart (`hooks/hooks.json:4-52`). They stay silent until a prompt starts with `$poteto-mode`, and `disable $poteto-mode` turns them off (`hooks/scripts/poteto-mode-state.mjs:13-15`). While on, each turn gets 131 chars, about 33 tokens (`:251`). The `pstack-poteto-agent` subagent gets an 812-char persona (`hooks/scripts/poteto-subagent-context.mjs:22-26`). Codex must trust the hooks first (`README.md:79`). Of all the ports, this is the closest to upstream's sticky mode.
- **p3-stack, backnotprop and oh-my-pstack** have no hooks (a `find` for `hooks*` finds none). `p3-mode` keeps `disable-model-invocation: true` (`skills/p3-mode/SKILL.md:4`).
- **Setup skills that write user-level files.** These run only when the user invokes setup. The isolated install at `/tmp/pstack-install/claude-open-pstack/` has no CLAUDE.md.
  - Upstream writes an always-applied `~/.cursor/rules/pstack-models.mdc` (`U/skills/setup-pstack/SKILL.md:8,39`).
  - pstack-claude appends `@~/.claude/pstack-models.md` to `~/.claude/CLAUDE.md` (`plugins/pstack/skills/setup-pstack/SKILL.md:85`).
  - open-pstack does the same (`plugins/pstack/skills/setup-pstack/SKILL.md:12-16,127`), and also copies the sheet into `~/.codex/AGENTS.md` (`:20-26`).
  - pstack-codex writes `~/.codex/pstack-models.md` and copies two agents into `~/.codex/agents/` (`plugins/pstack/skills/setup-pstack/SKILL.md:10,62`).
  - p3-stack writes `p3-models.md` at the project root by default (`skills/setup-p3/SKILL.md:9`).
  - The sheets hold a model per role, not routing rules.

## What reef-stack can borrow (our reading)

1. **Keep the entry point user-invoked, as upstream does.** Then nothing competes with the orchestrator unless Rith asks for it. First, test whether another skill can still load a user-only skill in Claude Code. open-pstack says it cannot (`open-pstack/UPSTREAM.md:20`). The docs only say Claude "can't invoke the skill on its own" (Docs, "Control who invokes a skill").
2. **If a sticky mode is wanted, copy pstack-for-codex's shape.** Off by default. A typed phrase turns it on. Each turn gets a one-line reminder. A plain phrase turns it off. State is kept per session.
3. **Give any injected text an off switch in a file,** like pstack-claude's `session hook: off`. Have the text say that CLAUDE.md wins (`pstack-claude/plugins/pstack/hooks/session-start-context.md:14`).
4. **Use p3-stack's T3 mapping on reef-stack's T3 side.**
   - Subagents → `delegate_task` with `mode: "async"`, drained with `task_status` (`p3-stack/skills/p3-mode/SKILL.md:93`).
   - Each sub-coordinator, and any worker that needs its own worktree, preview or device → `t3_thread_launch` (`p3-stack/skills/p3-mode/playbooks/orchestrate.md:21-22`).
   - `/loop` and polling → `watch_pull_request`, with a long `schedule_task` heartbeat as a fallback (`p3-stack/skills/p3-mode/playbooks/autonomous-run.md:6`).
   - Transcript mining → `t3_thread_read` and `t3_thread_search` (`p3-stack/README.md:11`).
   - Model roles → resolved from `orchestrator_capabilities` into `p3-models.md` (`p3-stack/README.md:26`).
5. **Keep a model sheet per role, scoped to the project by default** with a global fallback, as p3-stack does (`p3-stack/skills/setup-p3/SKILL.md:9`). Don't add an import line to `~/.claude/CLAUDE.md`.
6. **Pin upstream in a file and sync with a tool.** Options: pstack-claude's `tools/upstream.json` plus `tools/sync.mjs`, pstack-for-codex's hashed `upstream.lock.json`, or backnotprop's `upstream` branch merged into `main` (`backnotprop/MIRROR.md:5-8`).
7. **Watch the skill listing.** Claude Code caps it at about 1% of the context window. When it overflows, the descriptions of the least-used skills are dropped (Docs, "Skill descriptions are cut short"). 58 auto-invocable pstack skills would crowd out reef-stack's own. `skillOverrides` can set imported skills to `"name-only"` or `"user-invocable-only"` (Docs, same section and "Override skill visibility from settings").

## Decision recorded earlier

The decision was to install nothing at user level, because the hook mandates would compete with a home-grown orchestrator. **The evidence still supports it, but the main reason has shifted.**
- Both Claude Code ports inject a routing mandate at every startup, clear and compact. open-pstack's cannot be turned off.
- Both make nearly every skill auto-invocable, so 56 to 58 descriptions sit in every session's listing. At about 3.1k to 3.4k tokens, the listing now costs more than the hook (about 250 to 315 tokens).
- Both setup skills write into `~/.claude/CLAUDE.md`.
- p3-stack's installer defaults to `~/.agents/skills` (`p3-stack/install.sh:6`). If it is used at all, pass `--project` (`:8-10`).

Our reading: pstack-claude at project scope with `session hook: off` would be tolerable. Copying chosen SKILL.md files into reef-stack carries even less risk.

## Corrections and unverified

Corrections to the earlier table:
- **Similarity:** every number drops under the stated method (see the table). The earlier method is unknown, and its numbers don't reproduce. Lag explains part of the drop: against each port's own pin, the scores are 69, 75, 62, 18 and 71%.
- **pstack-claude:** the hook is about 246 tokens, not 3.5k. The 3.5k figure matches the hook plus the skill listing (about 3.4k). It also targets OpenCode, Gemini and Prime Agent. The remote is ahead of the clone: 0.9.83, on upstream 0.15.21.
- **open-pstack:**
  - It has 19 open issues, not 24.
  - It is not a GitHub fork (`isFork: false`), but its history starts from pstack-claude's import commit `053ed78` (`open-pstack/NOTICE.md:3`).
  - The CLAUDE.md edit is real, but it comes from `/setup-pstack`, not from install, and pstack-claude does the same.
  - "Grok defaults" is only partly true. Grok 4.7 covers feature, refactoring, swarm and how-explorer. GPT-6.1-Sol covers bug-fix, perf and hillclimb. Opus covers judgment, and the panels use three vendors (`plugins/pstack/skills/setup-pstack/SKILL.md:106-120`). Upstream itself defaults code roles to Grok 4.7 (`U/skills/setup-pstack/SKILL.md:49-65`).
  - The remote is ahead of the clone: 1.7.0, on upstream 0.15.20.
- **pstack-codex:** confirmed. It has no hooks and explicit-only skills. Its defaults are Luna for code roles, Astra for judgment, and 6.1-Sol for explorers and investigators (`plugins/pstack/skills/setup-pstack/SKILL.md:39-55`).
- **pstack-for-codex:** the 4 hook events are confirmed, but they are opt-in, the least intrusive hook design of all the ports. "Skip" still holds for its content (10%).
- **p3-stack:** workers default to `delegate_task`. `t3_thread_launch` is for sub-coordinators and for workers that need their own thread or worktree. The dropped helper scripts and guide are confirmed: there is no `p3-mode/scripts/`, `docs/` or `automations/`. The only `scripts` folder left is `skills/show-me-your-work/scripts` (`find . -type d -name scripts`).
- **backnotprop:** 92%, not 99%. It carries harness-neutral edits (`backnotprop/MIRROR.md:3`).
- **potetos and oh-my-pstack:** they pin 0.15.0 and 0.14.3. The old table had no version for them.
- **`.claude/skills` vs `.agents/skills`:** the docs list only `.claude/skills` locations, plugins and claude.ai (Docs, "Choose where skills load"). New nuance: Claude Code 2.1.296 has a feature-gated `/import [codex|gemini|cursor]` that copies `.agents/skills` into `.claude/skills`. Found with `grep -a -b -F '.agents/skills' ~/.local/share/claude/versions/2.1.296`. It is an import, not a load path, and it doesn't appear in the docs index (`https://code.claude.com/docs/llms.txt`).

**Unverified:**
- **The earlier live test.** `/tmp/pstack-install/p3-project/.agents/skills/` and `p3-project-claude/.claude/skills/` each hold 52 symlinks into the p3 clone. Both have empty git repos, and no output was saved. The `.claude/skills`-only claim rests on the docs.
- That T3 Code reads `~/.agents/skills` (`p3-stack/README.md:22`).
- The real token cost per session. The figures here are chars ÷ 4, and `/context` was not run.
- How Cursor applies `mode: true` and `reminder`. This was read from frontmatter, not tested in Cursor.
- Whether `disable-model-invocation: true` stops one skill from loading another in Claude Code, as open-pstack claims.
- Whether ported text behaves the same as upstream. The similarity score measures text overlap only.
