# The DagangNow harness and AI connections Release 1

- **Repo:** `/home/dev/dev/daganoss/DagangNow` (written `DagangNow/` below). Read only: no checkout, fetch, install, build, lint or test was run.
- **Branch:** `staging` (local ref). **Commit:** `7b0a3dc8d`. Every `DagangNow/<path>:<line>` cite is at this commit unless it says otherwise.
- **Feature data is not on staging at 7b0a3dc8d.** Cites under `.claude/features/ai-connections/` are at `be1a22f87` (`feat/ai-connections`, "R1 on staging"). Run evidence lives outside the repo: the event log `~/.dagangnow-qa/ai-connections/events.jsonl` and the R1 report `/tmp/ai-connections-qa/R1-report.md` (in `/tmp`, so it can vanish; a copy taken 2026-10-13 is at [sources/dagangnow-ai-connections-R1-report.md](sources/dagangnow-ai-connections-R1-report.md)).
- **harness/r2-prep** is at `35714caf4`, the same commit as `origin/staging` after the R1 PR (#258) merged. It has no commits of its own yet, and its worktree (`~/.t3/worktrees/DagangNow/harness-r2-prep`) has no uncommitted harness edits. What it adds over 7b0a3dc8d came in with R1. See "Corrections and unverified".
- **Checked 2026-10-13.**

## In one paragraph

Rith built a build loop where scripts decide and agents do the work. An Opus orchestrator reads only one-line command output and launches builders, reviewers and verifiers. A planner script (`next.mjs`) turns the ledger, the event log, the open questions, the servers and the lock into an ordered list of actions. Every task gets a code review (GPT-6-Astra), then a live check (GPT-6.1-Sol). It merges through a gate that checks scope, verdicts and patch identity. At the end of a release a release reviewer and a product reviewer (Fable 5.1) walk what the user should see. The core idea: "The harness is code; a feature is data." (`DagangNow/.claude/harness/README.md:9`). R1 of AI connections ran 18 rows in about 16 hours, and nothing was found after merge. The cost sat in checking, not building: 12 rounds lost to plans and environments, one server for four builders, and auth rules the spec never stated.

## How the harness works

**Flow.** `build-feature` is the owner's guide. It has 11 steps, numbered 0 to 10, and 🧑 marks the owner's steps: 0 Ready, 1 Shape 🧑 (`/grill-with-docs`), 2 Spec (`/to-spec`), 3 Tickets 🧑 (`/to-tickets`), 4 Set up (`tickets.mjs import`), 5 Run, 6 Release, 7 Release PR, 8 Look on staging 🧑, 9 Production 🧑, 10 Retro (`DagangNow/skills/build-feature/SKILL.md:42-102`). Step 7 is the agent's step, not the owner's (`:86-90`). Steps 1 to 3 take "about an hour" of the owner's time (`:46`). Step 10: "A mistake that happened twice becomes a check or a script in `.claude/harness/` (with a test), not a sentence in a prompt" (`:100-101`).

**Roles.**
- Orchestrator: "You coordinate; you do not build, review or read code." (`DagangNow/skills/harness-orchestrate/SKILL.md:8`). Its model is not set in any config. The Puck log records orchestrators as `claude-opus-5-5, high` (`DagangNow/docs/superpowers/plans/2026-10-06-puck-store-builder-log.md:35`).
- Builders: `opus` for anything a person sees, `sonnet` for scripts, migrations and checks (`DagangNow/.claude/features/_template/tasks/_template.mjs:15`).
- Reviewer `gpt-6-astra`, verifier `gpt-6.1-sol`, release reviewer `gpt-6-astra`, product reviewer `claude-fable-5-1` (`DagangNow/.claude/harness/feature.mjs:104-107`). The release reviewer is a fifth role, the seams pass.

**Verdicts.** There are four verdicts. In order of precedence: BLOCKED (the environment stopped a step), BAD_TEST (the test or plan is wrong), FAIL (the code is wrong), PASS (`DagangNow/skills/harness-verify/SKILL.md:20-26`). "A skipped or not-run step is never PASS." (`:19`). Review runs before verify. Only a blocker or a major gets a fix agent. BLOCKED and BAD_TEST fix the cause and re-run, with no fix agent (`DagangNow/.claude/harness/verdict.mjs:1-8,39-49`). The default is one fix round, three at most, and the third runs only if the findings shrank (`DagangNow/.claude/harness/README.md:66-68`). "Minors never hold a merge." (`DagangNow/skills/harness-orchestrate/SKILL.md:65`). A question for the owner is a gate with a default, so the run never waits (`DagangNow/.claude/harness/README.md:100-101`).

**Ledger.** A feature is `.claude/features/<slug>/`. It holds `feature.mjs` (config), `units.tsv`, `gates.tsv`, `preferences.md`, `overview.md` (event lines, newest first, plus handoffs), `tasks/<batch>.mjs` (brief, `sees`, criterion, verify plan, paths) and `run.generated.js` (`DagangNow/.claude/harness/README.md:12-20`). The `units.tsv` columns are `id release deps model effort status branch worktree port sha issue` (`DagangNow/.claude/features/_template/units.tsv:1`). The `gates.tsv` columns are `n asked task question default answer answered`. GitHub issues hold the what and the why: the import copies brief, criteria and `sees` from each ticket, and "the task's criteria win over the ticket's thread" (`README.md:188-191`). The ledger holds the how and the status. Each release ships as one PR from the integration branch into staging (`DagangNow/skills/build-feature/SKILL.md:86-90`). R1 was PR #258. The event log and the scratch files live outside the repo (`README.md:22-23`).

| Script | One line |
|---|---|
| `wake.sh` | The first command on every wake. It prints sync state, actions, gates, overview, lock and servers. It launches nothing. |
| `next.mjs` | The planner. It prints actions in order: `RELAY DECIDE RECORD MERGE UNBLOCK FIX-TEST FIX BLOCK RELAUNCH LAUNCH BUILD …`. |
| `brief.mjs` | Writes the brief or findings file and prints the exact launch input (`WORKFLOW`, Codex `DELEGATE`, or Claude `AGENT`). |
| `run.js` + `sync.mjs` | `run.js` holds the shared prompts and verdict logic. `sync.mjs` builds `run.generated.js` from it and the feature data, and checks that the generated file is fresh. |
| `return.mjs` / `events.mjs` | Record a run and append to the event log: launch, return, verdict, merge, grant, carried, escape, owner-wait and more. |
| `ready.sh` | Checks readiness before every launch: worktree, clean tree, deps done, servers, lock. |
| `verify.sh` / `plan-check.mjs` | `prep` rebases, re-runs the task's tests, starts the server, seeds QA and takes the lock. It then asks the server whether each planned route exists. "it never passes by default". |
| `merge-check.mjs` / `merge.mjs` | The merge gate (scope plus grants, no AI trailers, verdicts on the same patch-id, stamps), then rebase, checks and fast-forward. |
| `release.mjs` | `check` (one heavy job), `brief` (the seams pass), `product` (the walk of every `sees` line). |
| `store.mjs` / `tickets.mjs` | Edit units and gates, log, hand off, commit. Import a spec's GitHub tickets as tasks. |
| `metrics.mjs` | Per-task numbers from the log: wall time, tokens, fix rounds, first pass, lost minutes, escapes. |

**Wake.** The orchestrator runs `wake.sh` and takes the actions in order (`DagangNow/skills/harness-orchestrate/SKILL.md:14-17`). When `next.mjs` prints `WATCH`, it starts `watch.mjs` in the background. The script exits when a verdict file lands on disk, "in case the completion notice never arrives" (`DagangNow/.claude/harness/watch.mjs:2-3`), and its exit wakes the orchestrator. Past `relayAt` (200k tokens) the planner prints `RELAY` and the orchestrator hands off to a successor. The orchestrator's context size is self-reported through `--context` (`DagangNow/.claude/harness/next.mjs:10-11`).

**Capacity.** `server.sh` keeps a registry, and `stop` kills only recorded pids. It refuses a server past `host.maxServers` (`DagangNow/.claude/harness/server.sh:1-5,62`). `lock.sh` is the one tenant lock. Only the orchestrator takes it, for the one agent that writes or drives the browser (`lock.sh:1-5`). `maxBuilders` and `maxServers` are separate knobs, each defaulting to 2 (`feature.mjs:72-73`). The planner checks builders against `maxBuilders` only (`next.mjs:128`). Nothing ties the number of builders to the capacity for live checks.

**Setting up a feature.** Copy `_template`, fill `feature.mjs`, write a tracer-bullet task first, then `store.mjs unit add`, `store.mjs release R1` and `sync.mjs` (`DagangNow/skills/harness-new-feature/SKILL.md:13-23`). For shaped work, run `tickets.mjs import <spec> --dry-run`, then run it for real. It uses read-only `gh` to write `tasks/<batch>.mjs` and the `units.tsv` rows, blockers first. Then fill every `TODO(import)` (`paths`, `owns`, lanes, missing `sees`) and turn outside blockers into gates (`:25-38`, `DagangNow/.claude/harness/README.md:175-201`). Task size: about 12 files and 6 criteria. Set `review: true` for auth, money, concurrent writes and shared contracts (`SKILL.md:42-47`).

## What is portable vs DagangNow-only

Grep over the 25 files in `.claude/harness/` that are not tests, for DagangNow names (`dagangnow|tudung|qa-*|lvh.me|scripts/qa|ai-dev|puck`). Six files match:

| File | What is project-specific |
|---|---|
| `README.md:147-173` | The whole "QA tenant" section: `qa-seller@dagangnow.test`, store `tudungpeople`, `scripts/qa/*`, `~/.dagangnow-qa`, the scrubbed copy of Neon staging. It also tells the Puck migration story. |
| `feature.mjs:53,85` | Defaults: state dir `~/.dagangnow-qa/<slug>` and the Playwright path under `~/.dagangnow-qa`. |
| `plan-check.mjs:5,105` | `lvh.me` counts as a local host. A comment about Puck. |
| `tickets.mjs:40` | `DEFAULT_REPO = 'Daganoss-Digital/DagangNow'`. |
| `events.mjs:36-37` | The Puck log's `astra`/`sol` roles read as `review`/`verify`. |
| `lib.sh:40` | A comment about `lsof` on the ai-dev machine. |

The coupling to the stack is wider than these names. `feature.mjs:41,75-96` defaults to pnpm commands, `drizzle/**` seams and `tests/unit/db`. `worktree.sh`, `server.sh`, `ready.sh` and `run.js` assume a `.env.local` with a `PORT=` line that points at a shared database. `merge.mjs:94` parses vitest output. `merge-check.mjs` needs the `typescript` package to read test imports. `brief.mjs:127-131` emits a T3/Codex `delegate_task` shape, and `sync.mjs` and `run.js` target the Claude Code Workflow tool. `lib.sh` and `feature.mjs:160-163` put mise shims first on the PATH. **Portable as is:** the planner, the ledger and event formats, the verdict rules, the merge gate logic, the gates with defaults, wake/watch, the lock, the server registry, metrics, the ticket import, and the five role skills (with the QA lines removed). **Needs adapters:** check commands, the dev-server command, env and port handling, QA accounts and sessions, reviewer providers and the launch shape.

## Codebase guard rails (audit, staging 7b0a3dc8d)

| Claim | Finding |
|---|---|
| Biome `noExplicitAny`, `noNonNullAssertion` off | Verified: `"noNonNullAssertion": "off"` (`DagangNow/biome.json:46`) and `"noExplicitAny": "off"` (`:49`), on top of `"recommended": true` (`:41`). |
| Layer-import check exists, fails, not wired in | Exists: `DagangNow/.agents/scripts/check-layer-imports.mjs`. Only `agent:check` runs it (`DagangNow/package.json:61`). It is not in `verify` (`:72`), and `pr-gate.yml` runs only `pnpm verify` and `verify:contracts` (`:54,90`). The R1 log says it "fails on a layer import in src/shared/model/order-types.ts" (`.claude/features/ai-connections/overview.md:99`). 15 violations: reported earlier, not re-run. |
| `tsconfig` is `strict` only | Verified: `"strict": true` (`DagangNow/tsconfig.json:7`). None of the three tsconfigs sets `noUncheckedIndexedAccess` or any other extra flag. |
| Only git hook is commit-msg | Verified for active hooks. `prepare` sets `core.hooksPath .githooks` (`package.json:9`), which holds only `commit-msg`. It rejects AI attribution, including `Co-authored-by:` (`DagangNow/.githooks/commit-msg:5`). There is no .husky, lefthook or simple-git-hooks. **But** `.git/hooks/pre-push` exists (untracked, an "ai-dev guard" that refuses pushes to main and master). With `hooksPath` set, git never runs it. |
| No Claude Code hooks | Verified: `.claude/settings.json` has only `permissions` and `enabledPlugins`. There is no `settings.local.json`. |
| CI on PRs and `main` only; agents push staging | Verified: `on: pull_request`, `push: [main]`, `workflow_dispatch` (`DagangNow/.github/workflows/pr-gate.yml:14-19`), and "No push trigger on staging: integration happens through PRs." (`:9`). Agents push staging "on their own", fast-forward only (`DagangNow/AGENTS.md:84-86`), set by commit 7b0a3dc8d itself. So a direct push to staging skips CI. |
| No verify skill or feature map outside the harness | Partly. There is no verify skill but `harness-verify`. `skills/testing/SKILL.md:45-56` has a generic "Browser verification" paragraph. `docs/FEATURES.md` is a feature catalog, with a snapshot dated 1 September 2026 (`:5`), and a "Primary route map" (`:979`). No skill, AGENTS.md line or harness script points to it, and it has no checks or QA accounts. |
| ~1,000 unit tests | **Wrong.** About 1,000 is the number of unit test *files*: 1,000 under `tests/unit` plus 47 under `src` (vitest includes `tests/**/*.test.ts` and `src/**/*.test.ts`, `vitest.config.ts:26`). Method: `git grep -c` at 7b0a3dc8d for lines with `it(` / `test(`, including `.each/.skip/.only/.todo` (266 of those). That gives **8,653** test lines. Real runs give about 10,600: "1043 files / 10602 tests passed" on an AC-02 commit (`overview.md:447`), and 11,148 pass with 66 skipped at the R1 release check (`R1-report.md:34`). |

## R1 lessons

**Headline.** 18 rows, all merged (`/tmp/ai-connections-qa/R1-report.md:5`). The first launch was at 06:43Z (`events.jsonl:3`), the last merge at 22:06Z, and the product PASS that closed R1 at 22:39Z (`events.jsonl:292`). That is 15.9 h. First pass: 6 of 18 merged with no fix round, 33.3%, recomputed from the event log with the definition in `DagangNow/.claude/harness/metrics.mjs:59`. The report itself prints "First-pass 29.4%. 12 non-code verdicts, 266.8 min lost. 0 escapes." (`R1-report.md:41`). That figure is 5 of 17, written before AC-08b merged. Nothing escaped: there are no `escape` events, and the owner "confirmed staging good" (`overview.md:13`).

**Owner-level.**
1. **More builders while checking was the bottleneck.** The load test set "maxBuilders 2->4 (R1's widest level), maxServers 2->1 (a dev server is ~3 GB)" (`overview.md:455`). The agents' memory slice "throttles at 5.84 GB and kills at 8.76 GB" (`overview.md:422`). The owner later raised the cap and set 2 servers and 3 builders (`preferences.md:17`). "About 6.3 h of live checks queued" is **Unverified**. Recomputed from the event log: verify runs add up to 7.2 h on the one server, and tasks waited 4.3 h in total between review PASS and verify launch.
2. **Security behaviour left implicit in the spec.** Five cases, all caught before merge. Revoke then reconnect "revives old access and refresh credentials" (AC-05 review blocker, `overview.md:384`). A consent page for store A "can grant store B" (AC-10 review blocker, `:259`). A magic-link action with "no server-side rate limit" (AC-05b review major, `:321`). A "banned member passes OAuth MCP auth" (R1 final review, `:84`). Disconnect made "the member's other store connection" fail (`:206`). That last one was caught by the live verifier, not a reviewer. Spec #230 never mentions banned members.
3. **Two MCP addresses.** AC-10, the second (plain `/api/mcp`) address, took 3.57 h from first launch to merge (events). It had a review blocker, a CORS gate (`gates.tsv:8`), and an 85-min BAD_TEST (`overview.md:207`). The owner then merged the two into one connection per app per store (`preferences.md:18`).
4. **Known-flaky baseline.** `outline-move.test.ts` was logged as flaky during Puck on 2026-10-09 and 10-10 (`DagangNow/docs/superpowers/plans/2026-10-06-puck-store-builder-log.md:865,596`). In R1 it cost AC-02 most of its 3.47 h: "full pnpm verify on AC-02 kept failing on tests/unit/store-editor/outline-move.test.ts, which is load-flaky by itself" (`overview.md:400`, gate 5 at `gates.tsv:6`).
5. **Harness changes mid-run, on the feature branch.** Five harness commits landed during the run on `feat/ai-connections`: `f42dd7cd0`, `03405f29f`, `a9557f842`, `7f7fcefbc`, `a5547f313` (13:25Z to 18:09Z, `git log`). The final review flagged them as a major: "owner-requested harness commits on feat/ai-connections" (`gates.tsv:9`). The owner's queued work held new launches while it landed (`overview.md:216`).
6. **Slow to open the staging PR.** The handoff said of step 7: "Not started: it needs the owner." (`overview.md:23`). Yet the step has no 🧑, and agents may open and merge the release PR (`preferences.md:10`). PR #258 opened at 01:18Z, 2 h 39 min after the product PASS (`gh pr view 258`).

**Not the owner's.**
- **12 non-code verdicts, 266.8 min (about 4.4 h)** (`R1-report.md:41`). 11 were verify rounds. 1 was a code review cancelled by a T3 server restart (`events.jsonl:28`). The causes were all plans or environments, never code. QA accounts have "no readable inbox" (`overview.md:317`). The verify server "had live QStash" (`:212`). One plan expected an unmerged task's page (`:207`), one expected no write from an editor that autosaves (`:221`), and two broke the fixture rules (`:88,122`). r2-prep's README counts "Nine more on ai-connections (154 minutes)" as plans or environments, not code (`harness/r2-prep:.claude/harness/README.md:84`). The cause "plans guessed from intent-level `sees` lines" is **Unverified**: no file says how the plans were written.
- **Unverified OAuth risks swallowed.** Spec #230 has a section "Unverified facts the build must check", including whether consent can narrow scopes and "That the Claude prefill link still works" (`gh issue view 230`, body lines 377-380). None of them became a task, gate or spike. AC-07's criterion A simply assumes the prefill link works (`tasks/ac.mjs:252`). That the architect flagged them is **Unverified**: `layout.md` holds no risks.
- **1.1M-token misfire.** The AC-10 fix run "got its args as a string and built all 26 tasks (about 1.1M tokens)" (`overview.md:220`, `events.jsonl:145`). It was fixed in `03405f29f`.
- **Orchestrators under-reporting context.** **Unverified.** The files hold only self-reports: handoffs "at ~175k" to "~196k" context (`overview.md:314,416`) against `relayAt` 200k. Context is passed by flag, so a wrong number can't be caught (`next.mjs:10-11`).

## Rules proposed for reef-stack

| Rule | From |
|---|---|
| Size checking capacity (servers, verifier slots, memory) before builders. Never run more builders than verifies can drain. | Owner 1 |
| Auth and money features get a required abuse-case branch in grilling: revoke and reconnect, cross-tenant, rate limits, banned or removed users, and side effects on other tenants. | Owner 2 |
| Every unverified assumption is listed, then spiked or explicitly accepted as a gate before tickets. | Not-owner 2, owner 3 |
| Verify plans come from a project feature map (routes, accounts, fixtures, mail and queue capture), not from `sees` prose. | Not-owner 1. The audit found no wired map. |
| The baseline is green before launch: the full suite passes N times on the release base. | Owner 4 |
| Harness changes happen between releases, on their own branch and PR. | Owner 5 |

Not covered yet: owner 6 (make the release PR an explicit agent step that no handoff can park), the misfire (since fixed by a check), and context reporting (measure it, don't trust a self-report).

## Corrections and unverified

- `build-feature` has 11 steps (0 to 10), not 10, and **step 7 Release PR has no 🧑** (`SKILL.md:86`).
- The orchestrator's model is a convention, not config. Only the Puck log names Opus.
- "Issues hold the what and why; the ledger holds the how and status; one PR per release" is a paraphrase. No file says it in those words.
- The 6 files with DagangNow names are verified, but about 8 more assume the pnpm/Next/`.env.local`/Codex stack.
- Layer check: 15 violations, reported earlier and not re-run.
- `.git/hooks/pre-push` exists but never runs.
- A feature catalog with a route map does exist (`docs/FEATURES.md`), but nothing uses it for verification.
- About 1,000 is the count of unit test *files*. The suite has 8,653 test lines and about 10,600 to 11,150 tests at run time.
- First pass: 33.3% from the event log, but the report prints 29.4%. The 16 h is really 15.9 h.
- 12 non-code verdicts, not "12 verify rounds": 11 verify plus 1 review.
- Of the five security cases, 4 were caught by reviewers and 1 by the live verifier.
- **Unverified (from earlier session):** the 6.3 h queue, plans guessed from `sees`, the architect's flag, and orchestrators under-reporting context.
- r2-prep vs 7b0a3dc8d adds `launch-file.mjs`, `plan-lint.mjs` (plus `plan-check --offline`), `bank/` (the mistake bank), the string-args refusal in `run.js`, the `--note` limit in `brief.mjs`, plan-lint lines in `harness-orchestrate`, and README sections on these. It changes nothing above: the verdicts, ledger columns, other skills, biome, tsconfig, CI, hooks and settings are all unchanged.
