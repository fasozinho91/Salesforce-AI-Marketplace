# Build backlog

The ordered work list for [plan.md](plan.md). Priorities are a suggestion for the architects to
debate; change them here.

## How to pick the next item

1. List items with status `Open` whose dependencies are all `Done`, highest priority first
   (P0 before P1, then by ID).
2. Show the top few with their plan section and ask the person which to take.
3. Set the item to `In progress` with the person's name in the same PR as the first change, so
   nobody else picks it.
4. Build it with the method in [plan.md section 12](plan.md#12-how-we-build-and-test-the-skills):
   eval scenarios from the skill's "Done when" lines first, then the skill.
5. When the PR merges, set it to `Done`.

Status values: `Open`, `In progress`, `Done`, `Dropped`. Size: S (under a day), M (a few days),
L (a week or more).

## Phase 0: foundation (P0)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B01 | Repo scaffold: `marketplace.json` with the four plugin skeletons, `supported-versions.json`, README with install steps (dependencies first, from their own marketplaces), repo `CLAUDE.md` with the "what's next" procedure, `contributing.md` | 3, 4.1, 4.3, 13 | | S | Open | |
| B02 | CI: secret scan, client-name/org-ID denylist check (denylist kept outside the repo), SKILL.md frontmatter lint, catalog generator and freshness check | 10, 12 | B01 | M | Open | |
| B03 | Eval harness: fixture SFDX project (prefix `acme_`) and a local runner for scenario evals with superpowers at the supported version, producing a results file to attach to PRs | 12 | B01 | M | Open | |
| B04 | Profile template `templates/project-profile.md` (fields, defaults, "Workflow adjustments" block) | 5 | B01 | S | Open | |
| B05 | C2 `setting-up-project-profile` | 5.4, C2 | B03, B04 | M | Open | |
| B06 | C1 `using-sfai` bootstrap and session-start hook: dependency check with install commands first, then profile check, then adjustments; one eval per row of the section 6 table | 3, 4.2, 6, C1 | B03, B04 | M | Open | |
| B07 | C7 hooks: output hygiene (em dash, attribution), production deploy and data guard, `.gitignore` guard, finish gate | C7 | B04 | M | Open | |
| B08 | C3 `naming-salesforce-components` | C3, App. A | B04 | S | Open | |
| B09 | C4 `salesforce-platform-gotchas` | C4, App. D | B01 | S | Open | |
| B10 | C5 `verifying-salesforce-work` (verify script) and C6 Code Analyzer convention rules | 7.4, 7.5, C5, C6 | B03, B04 | L | Open | |

## Phase 1, track A: delivery workflow (P1)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B11 | Ticket document templates | App. I | B04 | S | Open | |
| B12 | D1 `taking-in-a-ticket` | 7.1, D1 | B06, B11 | M | Open | |
| B13 | D2 `analyzing-impact` | D2, App. C | B12 | M | Open | |
| B14 | D5 `writing-uat-test-cases`, including the approval record | 7.4, D5, App. B | B12 | M | Open | |
| B15 | D3 `designing-ui-first` | D3 | B12 | S | Open | |
| B16 | D4 `writing-solution-designs` | 7.3, D4 | B12, B08 | M | Open | |
| B17 | C8 `managing-ticket-branches` (start, finish, review summary) | 7.6, C8 | B06, B07, B10 | M | Open | |
| B18 | D6 `testing-apex-first` | 7.4, D6 | B10 | M | Open | |
| B19 | D8 `deploying-safely`, including the two-part diff check script (ticket component list from git; org copy compared with the branch base) | 7.7, D8 | B07, B10 | M | Open | |
| B20 | D9 `reviewing-salesforce-changes` | D9 | B08 | S | Open | |
| B21 | D11 `salesforce-automation-patterns` | D11, App. E | B01 | S | Open | |

## Phase 1, track B: reporting (P1)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B22 | R1 `writing-personal-updates` | 8, R1, App. G | B04 | M | Open | |
| B23 | R2 `compiling-weekly-roundups` | 8, R2, App. G | B22 | M | Open | |

## Phase 1, track C: UI testing spike (P1)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B24 | Throwaway spike: click through a fixture screen flow in Lightning with Playwright MCP, browser signed in through `sf org open --url-only`; record what works and what needs special handling | D7, 14 | B03 | S | Open | |

## Phase 2 (P2)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B25 | D7 `running-functional-qa` | 7.5, D7 | B10, B14, B24 | L | Open | |
| B26 | A1 assessment: baseline retrieve (custom-only filter) and production collector | 9, A1, App. H | B03, B04 | L | Open | |
| B27 | A1 assessment: findings, automation map, remediation waves, open questions, cleanup manifests | 9, A1, App. H | B26 | L | Open | |
| B28 | A1 assessment: client deliverable generator with its checks, engagement templates | 9, A1, App. H | B27 | M | Open | |
| B29 | D10 `logging-with-nebula` | D10, App. F | B08 | S | Open | |
| B30 | C9 `contributing-a-lesson` | C9 | B02, B09 | S | Open | |

## Later (P3)

| ID | Item | Plan | Depends on | Size | Status | Owner |
|---|---|---|---|---|---|---|
| B31 | Deeper tracker integration: pull tickets and post UAT hand-off when a tracker connector is available | D1, D5 | B12, B14 | M | Open | |
