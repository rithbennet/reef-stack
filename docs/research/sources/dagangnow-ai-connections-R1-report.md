# AI connections, Release R1 report (2026-10-13)

Spec #230. Integration branch `feat/ai-connections`. Release check and final review r3 at 439115e61 (after AC-08b).

## What shipped (18 rows, all merged)

| Row | Ticket | What |
|---|---|---|
| AC-01 | #232 | One MCP core; every tool declares its AI permission |
| AC-02 | #233 | Better Auth upgraded to the latest 1.7 |
| AC-05, AC-05b | #236 | Tracer bullet: connect Claude to one store by signing in |
| AC-06 | #237 | Consent grid for Store design and Sales pages |
| AC-07 | #238 | Add an AI app buttons on Connect AI |
| AC-08 | #239 | Connected AI apps list and Disconnect |
| AC-09, AC-09b | #240 | Email the owner and managers when someone connects an AI app |
| AC-10 | #241 | Plain MCP address with the store picker |
| AC-11 | #242 | Edit permissions: narrow your own AI connection |
| AC-12 | #243 | Lifecycle: roles, Pro, 90 days, cap of 10 |
| AC-13 | #244 | History labels for changes made through an AI connection |
| AC-14 | #245 | Pasted tokens under Advanced, with the same permission grid |
| AC-15 | #246 | Platform admin one-press connection and Admin → AI access |
| AC-12b | (R1 review fix) | A banned member's connections are refused on every request, store and plain addresses |
| AC-15b | (R1 review fix) | A Free store shows the DagangNow support row |
| AC-08b | (R1 product fix) | The Connected AI apps row reads on a phone |

Held: AC-20 (gate 1, until hotfix #231 ships).

## Release gates

- Release check: PASS at cc38b54f0 (11147 pass, 66 skipped).
- Final review r1: FAIL, 3 majors and 1 minor. Two majors became AC-12b and AC-15b; the third (owner-requested harness commits on the branch) is gate 8, default "keep".
- Final review r2: PASS at cc38b54f0, one minor (below).
- Product review r1: FAIL at 7de62bd8b, 1 major (Connected AI apps row unreadable at 360-430 px) and 5 minors. The major became AC-08b (review PASS, verify PASS, merged d973d3b53).
- Release check: PASS again at 439115e61 (11148 pass, 66 skipped).
- Final review r3: PASS at 439115e61, two minors (already carried).
- Product review r2: PASS at 16fb26a1c, six minors (the five from r1, plus the Edit permissions tooltip opening by itself and briefly clipping off the right edge). R1 closed; feat/ai-connections pushed at 21a47e9e1.

## Metrics (metrics.mjs --tasks AC-)

Per merged task: 80.3 min wall-clock, 275.6k builder tokens, 1.3 fix rounds, 6 agent runs.
First-pass 29.4%. 12 non-code verdicts, 266.8 min lost. 0 escapes. 7 owner waits (604.4 min).

## Migration

0127 applied to the local QA DB (wt_ai_connections) only. Staging and production still need it when the release ships.

## Carried minors (not gated)

- `memberCallerProfile` (src/entities/ai-connection/server/repository.ts:55) uses a manual select/innerJoin; prefer db.query.profiles with the authUser relation (AC-12b review, R1 r2).
- AC-15b: the Pro comparison page test renders a mocked ConnectedAiAppsSection.
- An idle connection past 90 days can refresh once before its next request ends it (auth owner).
- A ban longer than 90 days, once lifted, ends the connection on the next request under the 90-day rule (AC-12b builder).
- AC-13 and AC-15 minors listed in the 19:55Z handoff.
- Product r1 minors: same-app rows from the store and plain addresses look identical (owner call: dedupe or label); the editor banner says "AI assistant" where History names the app; the ChatGPT How? steps name a Plugins menu ChatGPT no longer has; a signed-in shopper gets different refusals at the two addresses; the Admin → AI access intro still talks about tokens.
- AC-08b: its responsive test lacks BrandHubContext, so only Disconnect renders in it.

## Open gates

- 6 (default: leave the three pre-existing sign-in problems out of ai-connections).
- 8 (default: keep the owner's harness commits on the branch).
- 2-5 on their defaults. 1 holds AC-20.
