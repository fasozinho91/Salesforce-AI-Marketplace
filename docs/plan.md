# Salesforce-AI-Marketplace: Plan

**Status:** Draft for review · **Date:** 2026-10-13 · **Owner:** Felipe Sozinho

This document describes what the marketplace will contain and how its parts fit together. It is
written for the architects who will build it. The ordered work list lives in
[build-backlog.md](build-backlog.md).

## Contents

1. [Why this exists](#1-why-this-exists)
2. [Goals, non-goals, anti-goals](#2-goals-non-goals-anti-goals)
3. [Constraints](#3-constraints)
4. [Architecture](#4-architecture)
5. [The project profile](#5-the-project-profile)
6. [How we adjust superpowers](#6-how-we-adjust-superpowers)
7. [The ticket delivery workflow](#7-the-ticket-delivery-workflow)
8. [Project updates](#8-project-updates)
9. [Org health assessment](#9-org-health-assessment)
10. [Roles and the catalog](#10-roles-and-the-catalog)
11. [Skill catalog](#11-skill-catalog)
12. [How we build and test the skills](#12-how-we-build-and-test-the-skills)
13. [Build order and "what's next"](#13-build-order-and-whats-next)
14. [Decisions, calls and items to verify](#14-decisions-calls-and-items-to-verify)
- Appendices: [A Naming standard](#appendix-a-naming-standard) ·
  [B UAT scenario format](#appendix-b-uat-scenario-format) ·
  [C Impact analysis checklist](#appendix-c-impact-analysis-checklist) ·
  [D Verified platform gotchas](#appendix-d-verified-platform-gotchas) ·
  [E Automation patterns](#appendix-e-automation-patterns) ·
  [F Logging standard](#appendix-f-logging-standard) ·
  [G Project update formats](#appendix-g-project-update-formats) ·
  [H Org assessment artifacts](#appendix-h-org-assessment-artifacts) ·
  [I Ticket document set](#appendix-i-ticket-document-set)

---

## 1. Why this exists

Every Salesforce project accumulates its own instructions for AI agents: naming rules, test
conventions, deployment rules, how to write UAT steps. They live in each project's repo, they
drift apart, and every new team member starts from zero. Two kinds of public tooling exist, and
neither covers the middle:

- **superpowers** (`obra/superpowers`) gives agents a disciplined software process: design before
  building, written plans, task-by-task execution with reviews, root-cause debugging, and evidence
  before claiming anything is done. It assumes code with a local test command, commits after every
  task, and knows nothing about Salesforce.
- **Salesforce's own plugins** (`forcedotcom/sf-skills`: salesforce-development, dx-org-lifecycle,
  integration) generate and deploy metadata, run Apex tests and run Code Analyzer. They are
  single-purpose tools with no opinion on how a ticket moves from requirements to UAT.

This marketplace is the missing middle: Salesforce delivery practice that uses superpowers for
process and Salesforce's plugins for generation, configured for each project by a profile.

**Who it is for:** anyone working on a Salesforce project with Claude Code, whether a
consultant or a member of the client's own team. Skills are tagged by role so people find what
applies to them.

**What success looks like:**

- A new team member installs from the public marketplace, runs setup once in a project, and from
  then on the agent follows that project's rules without being reminded.
- A ticket goes from requirements to a deployed, UAT-ready change. The person approves the UAT test
  cases before anything is built, makes every business decision, and receives honest evidence of
  what was and was not tested.
- A PM gets a weekly round-up draft assembled from the team's personal updates.
- An architect runs a production org health assessment and gets client-ready deliverables,
  separate from the internal evidence behind them.

## 2. Goals, non-goals, anti-goals

### Goals

1. **Configure once per project.** A profile captures each project's conventions, and a setup skill
   fills it in by inspecting the repo and asking the rest.
2. **Ticket delivery workflow:** intake, impact analysis, UI design, solution design, UAT test
   cases, implementation plan, build, deploy, verification, review, finish.
3. **Testing that fits Salesforce:** UAT test cases approved first, Apex test-first, declarative
   changes verified against the approved cases, and functional QA done by the agent wherever it
   can, including clicking through the UI.
4. **Project updates:** personal updates, weekly round-ups and release notes.
5. **Org health assessment:** read-only production assessment with client deliverables.
6. **Role-based catalog:** every skill lists the roles it serves.
7. **Build on, don't replace:** superpowers and the Salesforce plugins stay the engines; this
   marketplace adds Salesforce practice and project configuration on top.

### Non-goals

- Replacing Salesforce's metadata generators (fields, objects, flows, Apex, permission sets). We
  add conventions on top of their output.
- Forking or editing superpowers. We adjust it through instructions, the profile and hooks.
- A ticket tracker or project management tool.
- Client-specific content of any kind.
- Harnesses other than Claude Code in version 1.

### Anti-goals (failures even if everything technically works)

- Any client name, org ID, URL, username, record data or personal data in the public repo.
- An agent making a business or functional decision without asking.
- An agent validating logic against UAT test cases nobody approved.
- An agent claiming "done" or "tested" without evidence, or marking a case Pass that it did not run.
- Skills that only work on one operating system.
- Dumping long question lists on people instead of asking in small batches.
- Duplicating what superpowers or the Salesforce plugins already do, creating two sources of truth.

## 3. Constraints

- **Public GitHub repository.** Everything committed is public. (Bitbucket Cloud also supports
  public repos and would work through a full git URL; GitHub is chosen because people can install
  with the short `owner/repo` form.)
- **Tested dependency versions.** superpowers and the Salesforce plugins are installed from their
  own marketplaces, not re-listed in ours: many people already have them, and a second copy would
  load duplicate skills and session-start hooks. This repo documents the versions the evals passed
  against. Checking them is the first thing the bootstrap does in every session: if a dependency is
  missing it prints the exact install commands; if an installed version is outside the tested range,
  or two copies are installed, it warns.
  - Required: superpowers, salesforce-development.
  - Recommended: dx-org-lifecycle (scratch orgs, permission set assignment for test personas),
    integration.
- **Cross-platform.** Scripts and hooks are written in Node (Claude Code already requires Node). No
  bash-only or PowerShell-only scripts. Known Windows trap: `sf` is a `.cmd` shim, so
  `sf data query -q "..."` breaks on `&`, quotes and parentheses; scripts write SOQL to a temp file
  and use `--file`.
- **Output hygiene defaults:** no em dash characters in chat, commit messages or PR/MR text; US
  English spelling. Markdown files may contain em dashes unless the project bans them.
- **Personal data:** aggregate in the query; never leave record-level extracts of people data on
  disk; when a raw extract is unavoidable, delete it in the same session.
- **Skill authoring method:** every skill is written with `superpowers:writing-skills` (watch an
  agent fail without the skill first, then write the skill). Descriptions start with "Use when..."
  and describe triggers only. Mechanical rules become scripts, Code Analyzer rules or hooks, not
  prose.
- **Model and effort:** every subagent dispatch sets model and effort explicitly. Team default,
  overridable in the profile:

  | Work | Tier (today) | Effort |
  |---|---|---|
  | Search, file lookups, mechanical edits, summarizing output | Fastest (Haiku) | medium |
  | Implementing a task from a clear written plan | Standard (Sonnet) | high |
  | Review, debugging, design or architecture decisions | Most capable (Opus) | extra-high |

  Never max effort unless the person asks.

## 4. Architecture

```
 Layer 0  dependencies (installed from their own marketplaces, tested versions documented)
          +--------------------+   +------------------------------------------+
          | superpowers        |   | Salesforce plugins                       |
          | process skills     |   | salesforce-development, dx-org-lifecycle,|
          |                    |   | integration                              |
          +---------^----------+   +-------------------^----------------------+
                    | adjusts                          | adds conventions to
 Layer 1  this marketplace
          +------------------------------------------------------------------+
          | sfai-core             bootstrap, setup, naming, gotchas,         |
          |                       patterns, verify, branches, review, hooks  |
          | sfai-solutioning      intake to approved design and test cases   |
          | sfai-build            Apex test-first, QA, deploy, logging       |
          | sfai-project-updates  personal update, round-up, release notes   |
          | sfai-assessment       org health assessment                      |
          | templates             profile, ticket document templates         |
          +-----------------------------------^------------------------------+
                                              | reads
 Layer 2  each project
          +------------------------------------------------------------------+
          | project profile (filled in by the setup skill, one per project)  |
          +------------------------------------------------------------------+
```

**Nodes:** superpowers; Salesforce plugins; the five marketplace plugins; the project profile.
**Edges:**

- `sfai-core` → superpowers: adjusts named steps (where documents go, branching, question
  handling, test command, finish gate). See [section 6](#6-how-we-adjust-superpowers).
- `sfai-build` → Salesforce plugins: calls their generators, deploy and test skills, then
  applies naming, descriptions and review rules to the result.
- `sfai-solutioning` → `sfai-build`: hands over through files only (approved test cases, solution
  design). Neither plugin requires the other to be installed.
- `sfai-solutioning`, `sfai-build`, `sfai-project-updates`, `sfai-assessment` → `sfai-core`: all
  read the profile and use the core verify command and hooks.
- Every Layer 1 skill → project profile: reads the project's values. No skill hard-codes a
  project's conventions.

### 4.1 Plugins

| Plugin | Contents | Install |
|---|---|---|
| `sfai-core` | Bootstrap, project setup, naming standard, platform gotchas, automation patterns, verify command and Code Analyzer rules, branch lifecycle, review lens, hooks, lesson contribution | Required, everyone |
| `sfai-solutioning` | Ticket stages 1 to 6: intake, impact analysis, UI design gate, solution design, UAT test cases | Architects, BAs, leads |
| `sfai-build` | Ticket stages 7 to 13: Apex test-first, functional QA, safe deploy, logging | Developers, admins, QA |
| `sfai-project-updates` | Personal update, weekly round-up, release notes | PMs, leads, anyone asked for updates |
| `sfai-assessment` | Org health assessment | Architects, leads |

**Why five plugins.** Every installed skill adds its description to every session. The token cost
is small; the real cost is relevance: more skills means more chances the agent picks the wrong one,
and people see skills that aren't theirs.

- **One plugin** would give a developer project-update and assessment skills, and give a client team
  skills it never uses.
- **One plugin per role** doesn't work because most skills serve several roles (UAT test cases
  serve BAs, QA, developers and admins); skills would be duplicated or placed arbitrarily. Roles
  are tags on skills instead ([section 10](#10-roles-and-the-catalog)).
- **By workflow** lets people install what they do. The ticket workflow splits at the approval gate
  (stage 6) because that is where work usually changes hands: architects and BAs take a ticket to
  an approved design and approved test cases, developers and admins build it. Someone who does the
  whole ticket installs both.
- **Core** holds what every other plugin depends on (profile, verify command, hooks) and anything
  used on both sides of the gate: naming, gotchas, automation patterns, and the review lens, which
  core's finish gate requires.
- Every plugin depends only on core, never on another plugin.

### 4.2 How adjustments reach superpowers

superpowers states that user instructions override its skills, and its brainstorming and
writing-plans skills explicitly defer to the user's preferred document locations. We use three
channels together:

1. **Bootstrap.** `sfai-core` injects a short `using-sfai` block at session start (the same
   mechanism superpowers uses). It lists the adjustments in [section 6](#6-how-we-adjust-superpowers)
   and points to the profile. Target: under 200 words.
2. **Profile.** The setup skill writes a "Workflow adjustments" block into the project profile,
   which the project's `CLAUDE.md` imports, so the adjustments also exist as project instructions.
3. **Hooks.** Anything mechanical is enforced by a hook rather than trusted to instructions
   (see [skill C7](#c7-hooks)).

Each adjustment has an eval proving superpowers honors it ([section 12](#12-how-we-build-and-test-the-skills)).

### 4.3 Repository layout

```
Salesforce-AI-Marketplace/
  .claude-plugin/marketplace.json     our five plugins
  supported-versions.json             superpowers and Salesforce plugin versions the evals passed against
  plugins/
    sfai-core/
      .claude-plugin/plugin.json
      skills/<skill-name>/SKILL.md (+ supporting files)
      hooks/                         Node hook scripts + hooks.json
      scripts/                       Node scripts (verify, setup helpers)
      code-analyzer/                 custom rules
    sfai-solutioning/ ...
    sfai-build/ ...
    sfai-project-updates/ ...
    sfai-assessment/ ...
  templates/
    project-profile.md               the profile template
    ticket/                          ticket document templates (appendix I)
  evals/
    fixture-project/                 sample SFDX project using the prefix acme_
    scenarios/<skill-name>/          pressure and application scenarios
  docs/
    plan.md                          this document
    build-backlog.md                 ordered work list
    catalog.md                       generated: skills by role
    contributing.md
  CLAUDE.md                          how agents work in this repo, incl. "what's next"
  README.md                          install and quick start
  LICENSE
```

## 5. The project profile

The profile is the only place project-specific values live. Layer 1 owns its template and the
setup skill; Layer 2 is one filled-in copy per project.

### 5.1 Where it lives

The setup skill asks, per project, which of these the team wants:

| Location | Shared with the team? | When to use |
|---|---|---|
| Tracked in the repo, imported by the repo's `CLAUDE.md` | Yes | The team agrees to keep AI configuration in the repo |
| A folder above the repo's worktrees, with its own `CLAUDE.md` | No (outside git) | The repo should not carry AI files, or several worktrees share one config |
| `CLAUDE.local.md` | No (personal) | One person trying it out |

### 5.2 Format

One Markdown file, `sf-project-profile.md`: a YAML front block for the values skills read, followed
by prose sections for rules that need explanation (project-specific gotchas, the "Workflow
adjustments" block). The `CLAUDE.md` that loads it imports it with `@sf-project-profile.md`.

### 5.3 Fields

"Default" is what setup proposes when the project has no convention of its own.

| Group | Field | Default | How setup finds it |
|---|---|---|---|
| Identity | Project name | (ask) | `sfdx-project.json`, folder name |
| Identity | Component prefix: the client's or vendor's prefix, e.g. `acme_` | (always ask; never derive from the project name) | Suggest from the most common prefix on custom components in the repo, then confirm |
| Identity | Label prefix, e.g. `ACME -` | Derived from the component prefix, then confirmed | Existing layout and flow labels |
| Identity | Systems for flow naming, e.g. `Sales`, `Billing`, `Standard`, `Custom` | `Standard`, `Custom` | Installed packages, existing flow labels |
| Conventions | Naming standard | [Appendix A](#appendix-a-naming-standard) | Compare existing components |
| Conventions | Flow policy | One flow per business process; combine only to resolve a real conflict | Existing flows |
| Conventions | Descriptions mandatory | Yes | |
| Conventions | Em dash ban scope | Chat, commits, PR/MR | Ask whether metadata text and docs are also banned |
| Conventions | Spelling | US English | |
| Tracker | Ticket key pattern, e.g. `ABC-123` | (ask) | Branch names, commit messages |
| Tracker | Tracker tool and whether the agent can reach it | (ask) | Connected tools |
| Docs | Ticket docs tracked in git? | (ask) | |
| Docs | Ticket docs location | Tracked: `docs/<TICKET>-<summary>/` inside the repo, committed on the ticket branch. Untracked: a folder outside the repo and outside every worktree (setup refuses a location inside one, since removing the worktree would delete the docs) | Existing docs folders |
| Docs | Ticket document set | [Appendix I](#appendix-i-ticket-document-set) | |
| Git | Remote exists | Detected | `git remote -v` |
| Git | Base branch rules: single base, or base per branch type (e.g. `feature/*` from one branch, `bugfix/*` from another) | Single base: the default branch | Branch list, existing aliases |
| Git | Branch naming pattern | `<TICKET>-<short-summary>` | Existing branches |
| Git | Worktree location | Sibling folders next to the main checkout | Existing worktrees |
| Git | Who owns the tracked `.gitignore` | Not us: never edit it | (ask) |
| Git | Commit message rules | Ticket keys spelled out in full; attribution trailer allowed | Recent commits |
| Git | Finish style | Agent commits; optional "staged finish" (soft reset to staged changes for human commit) | (ask) |
| Git | Integration style | PR/MR to the remote; or local merge with `--no-ff` when there is no remote | Remote and history |
| Orgs | Working org alias (build and test) | (ask) | `sf org list` |
| Orgs | Working org shared with other people? | (ask) | Org type (scratch org vs sandbox) |
| Orgs | UAT org alias | (ask) | |
| Orgs | Production alias (read-only use) | (ask) | |
| Orgs | Deploy policy | [Section 7.7](#77-deploy-policy-defaults) | |
| Orgs | Deploy method | Explicit `--source-dir` paths per component; per-ticket manifest optional | Existing manifests |
| Testing | Apex test data factories | Discovered | Search for `*TestDataFactory*`, `*Setup*Test*` classes |
| Testing | Coverage target | 85% per class | |
| Testing | Test persona users | None | (ask) |
| Testing | UAT format | Scenario-driven ([Appendix B](#appendix-b-uat-scenario-format)) | Existing test docs |
| Testing | Who approves test cases and design (stage 6) | The person doing the work | (ask: could be a BA, lead or client) |
| Testing | UAT testers can run SOQL | No | (ask) |
| Testing | UAT delivery | Text in chat for pasting into the tracker, plus the test-cases file | (ask) |
| Logging | Framework | None; Nebula Logger if installed | Installed packages, `Logger.` calls |
| Logging | Tags, scenarios, flow error prefix | Empty; `Error Logged` | Existing code |
| Project updates | Updates folder (weekly files, notes, personal updates) | `weekly-roundups/` in the project repo, kept on the default branch, never in ticket branches | Existing folders |
| Project updates | Round-up template document | (ask, only when round-ups are used) | |
| Project updates | Release notes folder | `release-notes/` in the project repo, on the default branch | Existing folders |
| Agents | Model/effort mapping override | Team default ([section 3](#3-constraints)) | |
| Gotchas | Project-specific gotchas | Empty | Added as they are found |

### 5.4 The setup skill

See [skill C2](#c2-setting-up-project-profile). In short: infer first, propose values, ask only
what it could not find, ask in small batches in dependency order, write the profile, then show the
person the result. If a session starts in a Salesforce project (has `sfdx-project.json`) with no
profile, the bootstrap offers to run setup. Setup can be re-run at any time to update a section.

## 6. How we adjust superpowers

| superpowers step | Default behavior | Our adjustment | Channel |
|---|---|---|---|
| Brainstorming: where the design goes | `docs/superpowers/specs/<date>-<topic>-design.md`, committed to whatever branch is checked out | For tickets, the design is `<TICKET>_solution-design.md` in the profile's ticket docs location. Never committed to a base branch | Bootstrap + profile |
| Brainstorming: intake | Open conversation to discover intent | For a ticket, `taking-in-a-ticket` runs first; brainstorming covers what the ticket leaves unclear | Bootstrap |
| Writing plans: location | `docs/superpowers/plans/` | `<TICKET>_implementation-plan.md` in the ticket docs location | Bootstrap + profile |
| Writing plans: test command | `npm test`, `pytest`, etc. | The core verify command ([skill C5](#c5-verifying-salesforce-work)) | Bootstrap |
| Writing plans: global constraints | From the spec | Also the profile's conventions (naming, descriptions, flow policy) | Bootstrap |
| Worktrees | Built-in tool creates `.claude/worktrees/<name>` from the remote's default branch; git fallback edits and commits `.gitignore` | Fetch, then branch from the remote copy of the profile's base for this branch type; create the worktree in the profile's location; enter it with the built-in tool by path; never edit a tracked `.gitignore` the team doesn't own (use `.git/info/exclude`) | `managing-ticket-branches` + hook |
| Test-driven development | Failing test first for all production code | By kind of work ([section 7.4](#74-testing-by-kind-of-work)). Approved UAT cases come before any of it | Bootstrap + solutioning and build skills |
| Executing plans: "rulings" | Agent settles any ambiguity itself and keeps going | Technical: decide and record. Business or functional: stop and ask ([section 7.3](#73-questions-and-decisions)) | Bootstrap |
| Executing plans: when to stop | Destructive ops, security, merges/pushes/publishing, hopeless plan | Also: any deploy outside the working org, any data change, any production access beyond read-only | Bootstrap + hook |
| Verification before completion | Evidence from a test command | Evidence per level of done ([section 7.5](#75-definition-of-done)) | Bootstrap + C5 |
| Code review | Generic reviewer | Reviewer receives the Salesforce review lens ([skill D9](#d9-reviewing-salesforce-changes)) as global constraints | Bootstrap |
| Review package | Script inside superpowers' build skills; only exists when work went through a plan; deleted with the workspace after a clean final review | Final review always runs before finish, PR or MR, plan or no plan; a review summary is kept with the ticket docs and in the PR/MR description | `managing-ticket-branches` + hook |
| Finishing a branch | Runs `npm test`, offers merge locally / push and PR / keep; plain `git merge` | Verify command instead of `npm test`; base branch and integration style from the profile; refuses to finish without the final review summary | `managing-ticket-branches` |
| Output | Templates contain em dashes; Claude adds attribution | No em dash in chat, commits, PR/MR; attribution per profile | Bootstrap + hook |
| Asking questions | One question per message | Keep the full list in the ticket's requirements file; ask in batches sized by dependency | Bootstrap |

## 7. The ticket delivery workflow

### 7.1 Stages

```
 1 Intake ............ taking-in-a-ticket: ticket branch and worktree first, then
                       ticket folder, frozen + latest requirements
 2 Impact analysis ... analyzing-impact: read-only, repo then production
 3 UI design gate .... designing-ui-first: only when new or changed custom UI
 4 Solution design ... writing-solution-designs: components, logic, order, risks, decisions
 5 UAT test cases .... writing-uat-test-cases: scenarios written before any build
 6 APPROVAL GATE ..... the profile's approver approves the UAT test cases and the
                       design; the approval (name, role, date) is recorded in the
                       test-cases file, so it can happen outside the session
                       (client decisions stay tagged [Client Decision Needed])
 7 Implementation plan superpowers:writing-plans, saved to the ticket folder
 8 Build ............. superpowers:subagent-driven-development or executing-plans,
                       using testing-apex-first and Salesforce generators
 9 Deploy ............ deploying-safely: diff check, working org only
10 Verify ............ verifying-salesforce-work + running-functional-qa
11 Final review ...... reviewing-salesforce-changes; review summary kept
12 Finish ............ managing-ticket-branches: commit/PR/MR per profile
13 UAT hand-off ...... client-facing test cases (no QA column)
```

Requirements can change at any stage. When they do, `<TICKET>_latest-requirements.md` is updated
in the same session, affected test cases go back to the person for re-approval, and the design and
plan are updated to match.

### 7.2 Ticket documents

The house default document set is in [Appendix I](#appendix-i-ticket-document-set). The profile
can rename, relocate or drop files, and decides whether they are tracked in git. Metadata source
never goes in the ticket folder; it stays under `force-app/`.

### 7.3 Questions and decisions

- **Technical questions** (which component type, how to structure a flow, test approach): the agent
  decides, records the decision and its reason in the solution design ("Decisions made" section) or
  the execution ledger, and continues.
- **Business or functional questions** (what the user sees, what happens to existing data, which
  records qualify): the agent stops and asks the person doing the work. The person can:
  - answer it;
  - tell the agent to decide (recorded as "decided by <name>, agent recommendation accepted");
  - mark it as a client decision.
- **Client decisions** start with the literal tag `[Client Decision Needed]` at the start of the
  sentence, carry a recommendation, and the documents are written as if the recommendation is
  accepted so the design stays readable. When answered: remove the tag, record answer, date and
  decider in the change log of `<TICKET>_latest-requirements.md`, and update the design if the
  answer differs. While a client decision is open, the agent builds what does not depend on it and
  holds back what does.
- **Asking style:** all open questions live in `<TICKET>_latest-requirements.md`. The agent asks in
  small batches, choosing the batch size by dependency (questions whose answers change other
  questions go first), and says how many remain.

### 7.4 Testing by kind of work

The approved UAT test cases always come first: validating logic against unapproved cases is wasted
work.

| Kind of work | "Test first" means | Verified by |
|---|---|---|
| Apex | New class or method: test plus a stub with the signatures (bodies return null or throw); deploy both to the working org; watch the test fail on its assertion; implement; deploy; run until green. Changed behavior in an existing method: the new test fails against the current code, so no stub is needed. Stubs only ever add code nothing calls yet, so they are safe in a shared org. Existing test data factories first; bulk tests with 200+ records | `sf apex run test` results and coverage |
| LWC | Jest, exactly as superpowers' test-driven development describes | `npm test` (Jest) |
| Declarative (fields, validation rules, flows, permission sets, layouts) | The approved UAT scenarios are the failing tests | Deploy, then agent-run checks: anonymous Apex inside a savepoint that rolls back, queries, Flow Tests where the flow type supports them, and UI click-through |
| Everything | Static checks | Code Analyzer, including our custom convention rules |

The core verify command runs all of the applicable checks for the changed components and writes one
report. superpowers' "run the tests" steps call it.

### 7.5 Definition of done

| Level | What | Who | Evidence |
|---|---|---|---|
| 1 | Code checks pass: Apex tests, Jest, Code Analyzer | Agent | Verify command report |
| 2 | Deployed to the working org | Agent | Deploy result plus a re-query of the org (deploys are transactional: a failed deploy rolls back components the CLI listed as created) |
| 3 | Functional QA: every approved case run where the agent can, including UI click-through | Agent | QA column per case: Pass, Fail or Not run, with screenshots or query output. "Not run" cases listed for a person |
| 4 | UAT by the client | Client | UAT column, filled by the client |

The agent's work is done at level 3 plus the UAT hand-off version of the test cases. A case
verified only by reading metadata is "Not run", never "Pass".

### 7.6 Branches, commits and finishing

- **Starting, at intake (stage 1), before any document is written:** fetch, then create the ticket
  branch from the remote copy of the base the profile names for this branch type (or the single
  shared branch). Create the worktree in the profile's location and enter it with Claude Code's
  built-in worktree tool by path. Tracked ticket docs are then written and committed on the ticket
  branch; untracked ticket docs go to the profile's folder outside every worktree. Stages 1 to 6
  write documents only; nothing under `force-app/` changes until stage 8.
- **During work:** agents commit normally; superpowers' reviews depend on commits. Commit messages
  follow the profile (ticket keys in full).
- **Before finishing:** the final whole-branch review must have run, plan or no plan. Its summary
  (what changed, decisions the agent made, QA results, deferred minor findings) is saved as
  `<TICKET>_review-summary.md` and reused as the PR/MR description.
- **Finishing:** per the profile's integration style: PR/MR to the remote, or local merge with
  `--no-ff`. If the profile asks for a staged finish, the agent soft-resets to the branch's base so
  the work becomes one staged set, and the person writes the commit.
- **Never:** edit a tracked `.gitignore` the team does not own (personal ignores go in
  `.git/info/exclude`, which covers every worktree and is never committed); push to protected
  branches; force-push without an explicit request.

### 7.7 Deploy policy (defaults)

| Action | Default |
|---|---|
| Deploy to the working org | Allowed after the two-part diff check in [D8](#d8-deploying-safely) shows only this ticket's changes. If it shows anything else, stop and flag it |
| Records created by functional QA in the working org | Allowed. Named with a `QA-<TICKET>` marker where the object has a name field, listed in the QA evidence, and deleted afterward unless the person keeps them for UAT. Creating users or changing setup data still asks |
| Deploy to any other sandbox | Ask first |
| Deploy to production | Never without explicit approval in the session; the hook forces a confirmation |
| Any other data change in any org | Ask first; aggregate-only queries otherwise |
| Production during impact analysis or assessment | Read-only queries and retrieves |
| Full profile files | Never deployed whole; send a sparse payload with only the changed permissions |
| Field-level security | On permission sets, never profiles |

## 8. Project updates

Three skills, one style. Details in [Appendix G](#appendix-g-project-update-formats).

- **Personal update** (Everyone): a team member's week, for a PM who asked for it. Built from the
  person's own commits (author filter), ticket documents, notes and tracker activity. Written in the
  same shape the round-up uses, with risks and anything that needs confirmation.
- **Weekly round-up** (PM, Lead): one running file per week in the project repo, added to as work
  happens rather than on Friday. Combines notes, commits, ticket docs and personal updates into the
  client-facing body, keeps working sections (log, needs confirmation, still open) below it, and is
  published by copying last week's document so the template carries over.
- **Release notes** (PM, Lead, BA): what a release delivers, for the people receiving it. Built from
  the tickets in the release (latest requirements, solution designs, deployment guidance, test
  cases) and the merged commits. Business-facing by default, with an admin section for actions such
  as permission set assignments and post-deployment steps.

Rules for all three: client-facing wording that describes outcomes, not file paths or repo layout; no
personal data; a source recorded for any line that isn't obvious; anything uncertain goes under
"Needs confirmation" instead of into a bullet.

## 9. Org health assessment

A read-only assessment of a **production** org. Details in
[Appendix H](#appendix-h-org-assessment-artifacts).

1. **Baseline:** if the project has no metadata baseline or repo, retrieve one from production first
   (custom components only: exclude installed-namespace components, keep local fields on managed
   objects).
2. **Collect:** read-only, aggregate-only queries against production.
3. **Analyze:** structured findings, an automation map per object, a remediation plan in waves with
   gate conditions, open questions with owners.
4. **Deliver:** an internal evidence set and a separate client set (exec readout and a backlog ready
   for tracker import), with a script that blocks client files containing local paths, data
   inventories or personal data.
5. **Engage:** readout run-sheet and decision briefs grouped into sessions.

Cleanup manifests may be prepared and validated with a dry run, always with a rollback, and are
never run by the assessment.

## 10. Roles and the catalog

| Role | Who |
|---|---|
| PM | Plans, status, client communication, project updates |
| Architect | Solution and technical architecture, impact analysis, assessments |
| Lead | Technical or delivery lead: reviews, merges, deploy approvals |
| Developer | Apex, LWC, integrations |
| Admin | Declarative builder: fields, flows, permission sets, layouts |
| BA | Requirements, decisions, UAT test cases |
| QA | Functional testing and evidence |
| Everyone | Applies to every role |

Every skill declares its roles as a list in its frontmatter (`metadata.roles: [Architect, Lead]`).
`docs/catalog.md` is generated from the frontmatter: one section per role listing its skills, and
a skill appears under every role it lists. CI fails if the catalog is stale or a role is not in the
list above.

## 11. Skill catalog

| ID | Skill | Plugin | Roles | Builds on |
|---|---|---|---|---|
| C1 | `using-sfai` | core | Everyone | superpowers:using-superpowers |
| C2 | `setting-up-project-profile` | core | Everyone | |
| C3 | `naming-salesforce-components` | core | Admin, Developer, Architect | |
| C4 | `salesforce-platform-gotchas` | core | Developer, Admin, Architect | |
| C5 | `verifying-salesforce-work` | core | Developer, Admin, QA, Lead | Code Analyzer, salesforce-development test/deploy skills |
| C6 | Convention rules for Code Analyzer | core | (used by C5) | salesforce-development:dx-code-analyzer-custom-rule-create |
| C7 | Hooks | core | (always on) | |
| C8 | `managing-ticket-branches` | core | Developer, Admin, Lead | superpowers:using-git-worktrees, finishing-a-development-branch |
| C9 | `contributing-a-lesson` | core | Everyone | superpowers:writing-skills |
| D1 | `taking-in-a-ticket` | solutioning | BA, Lead, Architect, Developer, Admin | superpowers:brainstorming |
| D2 | `analyzing-impact` | solutioning | Architect, Lead, BA | |
| D3 | `designing-ui-first` | solutioning | BA, Architect, Developer | Claude Design |
| D4 | `writing-solution-designs` | solutioning | Architect, Lead | superpowers:brainstorming |
| D5 | `writing-uat-test-cases` | solutioning | BA, QA, Developer, Admin | |
| D6 | `testing-apex-first` | build | Developer | superpowers:test-driven-development, salesforce-development Apex test skills |
| D7 | `running-functional-qa` | build | QA, Developer, Admin | browser automation |
| D8 | `deploying-safely` | build | Developer, Admin, Lead | salesforce-development deploy and validate skills |
| D9 | `reviewing-salesforce-changes` | core | Lead, Architect, Developer | superpowers:requesting-code-review |
| D10 | `logging-with-nebula` | build | Developer | |
| D11 | `salesforce-automation-patterns` | core | Architect, Admin, Developer | |
| R1 | `writing-personal-updates` | project-updates | Everyone | |
| R2 | `compiling-weekly-roundups` | project-updates | PM, Lead | Google Workspace connector when available |
| R3 | `writing-release-notes` | project-updates | PM, Lead, BA | |
| A1 | `assessing-org-health` | assessment | Architect, Lead, PM | |

IDs are stable labels, not plugin codes: the D skills are the ticket workflow wherever they are
packaged, so backlog references stay valid if a skill moves between plugins.

Each entry below gives what the skill must do and how a builder knows it is done. The "Done when"
lines are the acceptance tests for the eval scenarios in [section 12](#12-how-we-build-and-test-the-skills).

### C1 `using-sfai`

Injected at session start by a hook. Runs its checks in this order:

1. **Dependencies.** Checks superpowers and salesforce-development are installed and within the
   versions in `supported-versions.json`. If one is missing, prints the install commands and stops
   there:

   ```
   /plugin marketplace add obra/superpowers-marketplace
   /plugin install superpowers@superpowers-marketplace
   /plugin marketplace add forcedotcom/sf-skills
   /plugin install salesforce-development@salesforce
   ```

   Warns (without stopping) on a version outside the tested range, a duplicate copy, or a missing
   recommended plugin.
2. **Profile.** In a Salesforce project with no profile: offers to run setup. With a profile:
   reminds the agent to read it.
3. **Adjustments.** Lists the adjustments in [section 6](#6-how-we-adjust-superpowers) in under
   200 words.
- States the question-batching rule and the technical vs business decision rule.
- When the ticket workflow reaches a stage whose skill isn't installed (for example a developer
  without `sfai-solutioning`), names the plugin that provides it and how to install it.
- **Done when:** with superpowers uninstalled, the session opens with the install commands; with
  superpowers installed, an agent asked to build a ticket change saves its design and plan in the
  profile's location, uses the verify command as its test command, and stops on a business
  question.

### C2 `setting-up-project-profile`

- Inspects before asking: `sfdx-project.json`, prefixes on existing custom components, branches and
  remote, existing worktrees, installed packages (Nebula Logger, others), test factory classes,
  recent commit message style, existing docs folders.
- Proposes values with the evidence for each; asks only what it could not establish; always asks the
  component prefix; asks in small batches in dependency order (location of the profile first).
- Writes `sf-project-profile.md` from `templates/project-profile.md`, including the "Workflow
  adjustments" block, and adds the `@` import to the chosen `CLAUDE.md`.
- Can be re-run for one section.
- **Done when:** on the fixture project, setup infers at least the prefix suggestion, branches,
  factories and logging framework, asks for the rest in batches of at most three, and produces a
  profile that loads in a new session.

### C3 `naming-salesforce-components`

- Applies [Appendix A](#appendix-a-naming-standard) with the profile's prefix, label prefix and
  systems list.
- Used whenever a new component is named, including names passed to Salesforce's generators.
- Requires a description on every component that supports one, within the 255-character limit
  where it applies.
- If a convention cannot be followed, says so in the solution design and asks.
- **Done when:** given ten component requests (object, field, layout, record page, app, each flow
  type, permission set, permission set group, custom permission), the agent produces names and
  labels that match the appendix for the fixture prefix.

### C4 `salesforce-platform-gotchas`

- Reference skill holding [Appendix D](#appendix-d-verified-platform-gotchas), searchable by
  symptom and error text.
- Only verified items. Each entry: symptom, cause, what to do.
- **Done when:** given an error message from the list, the agent finds the entry and applies the fix.

### C5 `verifying-salesforce-work`

- A Node script, `verify`, that takes the changed components (from git diff against the base) and
  runs what applies: Code Analyzer with our rules, Apex tests for affected classes (named tests),
  Jest for affected LWC, and an org re-query for deployed components. Writes one report and exits
  non-zero on any failure.
- The skill text explains the four levels of done and what evidence each needs.
- **Done when:** a superpowers plan run on the fixture project uses `verify` as its test command,
  and a deliberately broken validation rule description (over 255 characters) fails it.

### C6 Convention rules for Code Analyzer

Regex and XPath rules built with Salesforce's custom-rule skill:

- new custom components carry the profile prefix;
- description present on components that support one;
- description length at most 255 characters on validation rules, permission sets and permission set
  groups;
- no `fieldPermissions` added to profiles;
- no em dash in metadata text when the profile bans it.

**Done when:** each rule has a passing and a failing fixture.

### C7 Hooks

All Node, all configurable from the profile.

| Event | Check | Action |
|---|---|---|
| Session start | Dependency check first (C1 step 1), then inject `using-sfai` and detect a Salesforce project without a profile | Context; install commands when a dependency is missing |
| Before a shell command | `git commit`, `gh pr create`, `glab mr create` or similar with an em dash, or with attribution when the profile forbids it | Deny with the reason |
| Before a shell command | `sf project deploy` or data-changing `sf data` command targeting the production alias | Ask for confirmation |
| Before a shell command | `sf project deploy` without a recorded diff check for this session | Ask |
| Before a file edit | Edit to a tracked `.gitignore` when the profile says the team doesn't own it | Deny, suggest `.git/info/exclude` |
| Before a shell command | Finish, PR or MR creation without `<TICKET>_review-summary.md` | Deny with the reason |

**Done when:** each check has a test that triggers it and one that passes through.

### C8 `managing-ticket-branches`

- Start (called by D1 at intake, before any document is written): fetch, compute the base from the
  profile, create branch and worktree, enter it, run the baseline verify.
- Finish: confirm the final review ran and the summary exists, run verify, then integrate per the
  profile (PR/MR, local `--no-ff` merge, or staged finish).
- **Done when:** on the fixture project with two branch rules configured, a `feature` and a
  `bugfix` ticket each branch from the right remote base, and finishing refuses without a review
  summary.

### C9 `contributing-a-lesson`

- Turns a verified lesson from project work into a generic gotcha entry or skill improvement.
- Removes client names, org details and data; opens a PR to the marketplace repo.
- **Done when:** given a lesson written with client details, the PR it prepares contains none.

### D1 `taking-in-a-ticket`

- Starts the ticket branch and worktree through C8, then creates the ticket folder and documents
  per the profile.
- Writes the requirements verbatim to `<TICKET>_initial-requirements.md` (never edited afterward)
  and copies them to `<TICKET>_latest-requirements.md`.
- Pulls the ticket from the tracker when reachable, otherwise asks for it to be pasted.
- Sorts open questions into technical and business, and spots whether new UI is involved (routes to
  D3).
- Hands off to D2.
- **Done when:** given a pasted ticket, the folder and both files exist, nothing is edited in
  `force-app/`, and business questions are listed rather than answered.

### D2 `analyzing-impact`

- Read and query only: no deploys, data changes or `force-app/` edits. Retrieves and query output go
  to the scratchpad.
- Covers the six areas of [Appendix C](#appendix-c-impact-analysis-checklist), plus the picklist
  checklist when a picklist changes.
- Searches the repo first, then queries production read-only for what the repo cannot show.
- Gives every finding a verdict (No impact / Needs change / Needs decision).
- Writes `<TICKET>_impact-analysis.md`.
- **Done when:** on the fixture project, a picklist change finds the validation rule, flow and
  report filter that reference the value, and every area has a line even when it is "No impact".

### D3 `designing-ui-first`

- **Triggers** on new or substantially changed custom UI: LWC, Experience Cloud pages, screen flows
  with real screens, new app or home pages. Adding a field to an existing page does not count.
- Stops before the solution design and asks the person to design the UI in Claude Design, then to
  bring back the design (link or export).
- The solution design maps the approved design to Lightning base components and SLDS.
- Falls back to a text sketch of the screens in the solution design when Claude Design is not
  available.
- **Done when:** a ticket for a new screen flow triggers the prompt, and a ticket adding one field
  to a record page does not.

### D4 `writing-solution-designs`

Writes `<TICKET>_solution-design.md`, before any metadata changes, with:

- a component-by-component plan;
- formulas and logic written out;
- order and dependencies, including data that must be true before deploy;
- risks and conflicts with existing automation;
- "Decisions made" (technical) and `[Client Decision Needed]` items with recommendations;
- the naming and flow policy from the profile.

**Done when:** a reviewer can build from it without asking; every business question is either
answered, delegated or tagged.

### D5 `writing-uat-test-cases`

- Uses the profile's UAT format. The default is the scenario format in
  [Appendix B](#appendix-b-uat-scenario-format).
- Written before the build and approved by the profile's approver before any logic is validated.
  The approval (name, role, date) is recorded at the top of the test-cases file; build skills check
  for it, so approval can happen outside the session. Changed cases need a new approval.
- Uses field labels, not API names. Checks go through the UI or list views unless the profile says
  testers can run SOQL.
- Confirms every step is physically possible for the tester.
- Keeps QA and UAT status per scenario; the hand-off version drops QA.
- **Done when:** given a developer's technical draft, the output has no class or API names, opens
  with a business reason of at most 255 characters, uses no SOQL (default profile), and has 3 to 7
  scenarios.

### D6 `testing-apex-first`

- The stub-first loop from [section 7.4](#74-testing-by-kind-of-work); profile factories first;
  bulk tests; meaningful assertions; coverage target from the profile.
- Delegates generation details to Salesforce's Apex test skills.
- **Done when:** the eval shows a deployed failing test before any implementation body exists.

### D7 `running-functional-qa`

- Runs each approved case that can be automated:
  - **Data-level:** anonymous Apex inside a savepoint with rollback, plus queries.
  - **UI-level:** browser automation. Playwright MCP is the default; Claude in Chrome is an
    alternative. The browser signs in with the URL from `sf org open --url-only`. That URL carries a
    session token: it is never written to files, logs or chat.
- Sandboxes only.
- Screenshots are saved as evidence.
- Runs as test personas when the profile lists them (creating users counts as a data change).
- Records it creates follow the QA data rule in [section 7.7](#77-deploy-policy-defaults).
- Fills the QA column honestly and lists "Not run" cases for a person.
- **Done when:** on a scratch or developer org, a screen flow fixture is clicked through end to end
  and the QA column carries screenshot evidence.

### D8 `deploying-safely`

- Two-part diff check before every deploy, otherwise stop and flag:
  1. **Local:** a core script builds the ticket's component list from git: everything changed
     since the branch base, committed or not. It compares that list with the components the deploy
     command names. Anything in the deploy set that isn't on the ticket list stops the deploy and
     is named, and so does any uncommitted change that isn't part of the ticket. Deploys always name
     components one by one (`--metadata CustomField:Account.acme_Tier__c` or the single file path),
     never a whole folder: deploying an object's folder also sends every other field in it. When the
     profile uses a per-ticket manifest, the script also checks the manifest matches the ticket list
     exactly.
  2. **Org:** for each component that already exists in the target org, retrieve the org's copy
     and compare it with the version at the branch base. If the org copy differs, someone changed
     it in the org since, and deploying would overwrite their work.
- Deploys explicit components.
- Maintains `<TICKET>_package.xml` as it goes when the profile uses manifests (explicit members,
  never `*`, deletions in a destructive manifest).
- Re-queries the org after any deploy, and always after a failed one.
- Sends sparse profile payloads only.
- Writes `<TICKET>_deployment-guidance.md` with every pre- and post-deployment step, order, commands
  and rollback.
- Production per [section 7.7](#77-deploy-policy-defaults).
- **Done when:** with an unrelated change in the working tree, the deploy is stopped and the extra
  file is named; and with a component changed directly in the org after the branch was cut, the
  deploy is stopped and the component is named.

### D9 `reviewing-salesforce-changes`

The checklist handed to superpowers' reviewers as global constraints:

- the profile's naming and descriptions;
- FLS on permission sets only;
- bulkification and selective SOQL;
- sharing and CRUD/FLS enforcement in Apex;
- the logging framework used where the profile requires it;
- flow policy respected;
- toggle pattern used correctly ([Appendix E](#appendix-e-automation-patterns));
- tests assert behavior;
- no hard-coded IDs;
- no em dash where banned.

**Done when:** a seeded diff with five convention violations gets all five flagged.

### D10 `logging-with-nebula`

- Applies [Appendix F](#appendix-f-logging-standard) with the profile's tags, scenarios and flow
  error prefix.
- Only active when the profile names Nebula Logger.
- **Done when:** a generated Apex action wraps DML in try/catch, logs at ERROR with the scenario and
  tags, saves once, and returns the profile's error prefix to the calling flow.

### D11 `salesforce-automation-patterns`

- Reference for [Appendix E](#appendix-e-automation-patterns): the automation toggle, combining
  flows only on a real conflict, and trigger order.
- **Done when:** asked to block manual edits to a field a before-save flow writes, the agent uses
  the toggle pattern, not an `ISCHANGED`-only rule.

### R1 `writing-personal-updates`

- Collects the person's own week (commits by author, ticket docs touched, notes, tracker activity).
- Writes the update in the round-up's "work" shape with risks and items needing confirmation.
- **Done when:** given a week of fixture commits and notes, it produces an update a PM can paste
  into the round-up without rewording.

### R2 `compiling-weekly-roundups`

- Maintains the running weekly file in the profile's location.
- Merges personal updates.
- Produces the client body and the working sections ([Appendix G](#appendix-g-project-update-formats)).
- Publishes by copying last week's document when a document connector is available, otherwise
  outputs text to paste.
- Records the edits the PM made to the draft so the next draft improves.
- **Done when:** given three personal updates and last week's file, it produces a client body with
  no file paths or personal data, every update represented, and a "Needs confirmation" section for
  anything unsourced.

### R3 `writing-release-notes`

- Asks which release: a range of commits or tags, a date range, or a list of ticket keys.
- Includes only tickets actually in the release (merged, and deployed to the release's target), and
  names any ticket it excluded and why.
- Writes the format in [Appendix G](#appendix-g-project-update-formats):
  - what's new and changed for users, grouped by business area;
  - fixes;
  - actions needed by users, and by admins (taken from each ticket's deployment guidance);
  - known issues (open client decisions, deferred findings).
- User sections use field labels and business wording, never API names; the admin section may name
  components.
- Saves to the profile's release notes folder and also outputs text to paste.
- **Done when:** given three fixture tickets, one not yet merged, it writes one entry per included
  ticket, excludes and names the unmerged one, uses no API names in the user sections, and lists the
  admin actions from each deployment guidance.

### A1 `assessing-org-health`

One skill with supporting files for each phase ([section 9](#9-org-health-assessment),
[Appendix H](#appendix-h-org-assessment-artifacts)):

- **Baseline retrieve:** production, custom-only filter.
- **Collector:** Node, read-only, aggregate-only, production alias from the profile.
- **Analysis:** findings, automation map, remediation waves, open questions.
- **Deliverable generator:** enforces client-facing rules and fails on a violation.
- **Engagement templates.**

**Done when:** against a developer org seeded with known issues (overlapping before-save flows, a
validation rule with a global bypass, unused fields), the findings list contains each seeded issue
with evidence, and the client deliverables pass the generator's checks.

## 12. How we build and test the skills

- **Method:** each skill follows `superpowers:writing-skills`.
  1. Write the eval scenarios from the skill's "Done when" lines.
  2. Run them without the skill and record what the agent does.
  3. Write the minimal skill.
  4. Re-run until they pass.
  5. Close loopholes.
- **Scripts and hooks:** built with `superpowers:writing-plans` and test-first development (Node test
  runner).
- **Fixtures:** `evals/fixture-project/` is a small SFDX project using the prefix `acme_`. It has a
  few objects, flows, validation rules, permission sets, an Apex class with a test factory, and one
  screen flow. Org-dependent evals run against scratch orgs or a free Developer Edition org. Never a
  client org.
- **Running evals:** `claude plugin eval` where it fits; otherwise a Node runner that dispatches
  scenarios to fresh subagents. Every adjustment in [section 6](#6-how-we-adjust-superpowers) has
  an eval with superpowers installed at the version in `supported-versions.json`.
- **Where evals run:** locally, by the PR author, with their own Claude Code and their own Developer
  Edition org (Dev Hub enabled for scratch orgs). The PR attaches the results file. CI runs only the
  deterministic checks: lint, script and hook unit tests, denylist, catalog.
- **Upgrading dependencies:** raising a version in `supported-versions.json` requires the full eval
  suite to pass.
- **Public-repo safety (CI):**
  - secret scanning;
  - a client-name and org-ID denylist check, where the denylist itself is kept out of the repo
    (local file or CI secret), since a committed denylist would publish the names;
  - frontmatter lint (name, "Use when" description, roles from the allowed list, word budgets);
  - catalog freshness.
- **Review:** every PR is reviewed by another architect.
- **Versioning:** semantic versions per plugin.

## 13. Build order and "what's next"

The ordered list with status is [build-backlog.md](build-backlog.md). Priorities are a suggestion
for the architects to debate.

**How an agent answers "what's next?" in this repo** (this procedure also goes in the repo's
`CLAUDE.md`):

1. Read `docs/build-backlog.md`.
2. List items with status `Open` whose dependencies are all `Done`, highest priority first.
3. Show the top few with their plan section and ask which to take (or confirm the top one).
4. Mark it `In progress` with the person's name in the same PR as the first change, so two
   architects don't pick the same item.
5. Build it with the method in [section 12](#12-how-we-build-and-test-the-skills).
6. When the PR merges, mark it `Done`.

**Suggested phases:**

- **Phase 0, foundation:** repo scaffold and CI, profile template and setup, bootstrap, hooks,
  naming, gotchas, verify command.
- **Phase 1, three tracks in parallel once the foundation exists:**
  - Track A: ticket workflow (solutioning, then build).
  - Track B: project updates.
  - Track C: UI-testing spike.
- **Phase 2:** functional QA, assessment, logging, lesson contribution, catalog polish.

## 14. Decisions, calls and items to verify

### Decided by Felipe

- Team means anyone on the project: the consulting team or the client's own people. The plan names
  no client.
- Agents commit during work. A final review and its summary must exist before any finish, PR or MR.
- The layer decides where superpowers writes designs and plans, via the profile.
- Branch rules (base per branch type or single branch) live in the profile; the base is refreshed
  from the remote before branching.
- Technical questions: the agent decides and documents. Business or functional questions: ask the
  person. Client decisions use `[Client Decision Needed]`.
- UAT test cases are approved before any build or logic validation.
- Four levels of done, with the client's UAT as level 4.
- Agents test everything they can, including UI click-through and screen flows.
- No em dash in chat, commits or PR/MR; superpowers' own files stay unchanged.
- Deploy policy lives in the profile.
- The client profile template and setup skill are part of Layer 1.
- Scope includes org health assessment (production only, baseline retrieved if missing), weekly
  round-up and personal update.
- Skills carry a list of roles.
- Prefix is a setup field (client or vendor prefix), never derived from the project name.
- Defaults when a project has none: flows per business process; scenario-driven UAT; testers are
  not assumed to run SOQL, checks go through the UI and list views.
- New UI is designed in Claude Design first.
- Public GitHub repository.
- Dependencies are checked first in every session, with install instructions printed when missing.
- The deploy diff check has two parts: local (only this ticket's components) and org (no one
  changed the component in the org since the branch was cut).
- Round-up files default to the project repo. The standard client sections are work by theme and
  risks.
- Five plugins: `sfai-core`, `sfai-solutioning`, `sfai-build`, `sfai-project-updates`,
  `sfai-assessment`. The ticket workflow splits at the approval gate; automation patterns and the
  review lens live in core.
- Project updates include release notes.

### Calls I made (check these)

- The plugin name `sfai-solutioning` (the alternative was `sfai-analysis-design`; `sfai-planning`
  was rejected because it collides with superpowers' implementation plan and with project
  planning).
- Release notes format and its default sections (Appendix G).
- Dependencies are installed from their own marketplaces with tested versions documented, rather
  than re-listed and pinned in ours (avoids duplicate skills and hooks).
- The ticket branch and worktree are created at intake; untracked ticket docs live outside every
  worktree.
- Records created by functional QA in the working org are allowed, marked and cleaned up.
- The stage 6 approver is a profile setting, default the person doing the work, recorded in the
  test-cases file.
- Evals run locally by the PR author; CI runs deterministic checks only.
- Required versus recommended Salesforce plugins (C1).
- Adjustments delivered three ways: bootstrap, profile block, hooks.
- The profile is a Markdown file with a YAML front block, imported by `CLAUDE.md`. Its location is
  chosen per project.
- The ticket document set in Appendix I, adding `implementation-plan`, `impact-analysis` and
  `review-summary` to the six-file set.
- The layout naming rule in Appendix A resolves two contradicting sections in the source standard.
- Process-scoped record-triggered flow naming in Appendix A is new: the source standard only named
  combined flows.
- Coverage target default 85%.
- Playwright MCP as the default browser tool.
- Claude Design trigger list (D3).
- Version 1 targets Claude Code only.
- Personal updates and round-ups are in Markdown; publishing to a document tool is optional.
- Skill names use the verb-first style superpowers recommends.

### Items for builders to verify

- The most reliable way for a session-start hook to detect which plugins and versions are
  installed and enabled.
- Whether Claude Code plugins can declare a dependency on another plugin. If not, `sfai-core`
  is installed first by documentation and each skill checks for it.
- That SessionStart-injected text and imported profile instructions take precedence over
  superpowers' defaults in practice (covered by the section 6 evals).
- The SKILL.md frontmatter field for roles (`metadata.roles`) is accepted by Claude Code's skill
  loader.
- CLI support for running Flow Tests.
- The current Claude Code setting that controls commit and PR attribution, so the profile can set
  it per project.
- Entering a sibling-folder worktree with the built-in worktree tool by path.
- What Claude Design exports (link, image, code) and the best hand-back format for D3.
- Whether Lightning's shadow DOM needs special handling in Playwright for screen flows (the Phase 1
  spike).

### Open, not blocking the build

- License (the repo is public).
- Whether to fund an API key so CI can run the evals instead of PR authors.
- Whether the repo later moves from Felipe's personal GitHub account to an organization.
- Who besides Felipe can approve PRs.

---

## Appendix A: Naming standard

`<prefix>` is the profile's component prefix (e.g. `acme`), `<PREFIX>` its label form (e.g. `ACME`).
API names are the snake case of the label with the prefix. Any component that supports a
description must have one.

| Component | Label | API name | Example |
|---|---|---|---|
| Custom object | `<Label>` | `<prefix>_<Label_In_Snake_Case>` | `TV Channel` → `acme_TV_Channel` |
| Field (any object, ours or not) | `<Label>` | `<prefix>_<Label_In_Snake_Case>` | `Channel Number` → `acme_Channel_Number` |
| Page layout | `<PREFIX> - <Object> Layout` | `<prefix>_<Object>_Layout` | `ACME - Channel Number Layout` |
| Compact layout | `<PREFIX> - <Object> Compact Layout` | `<prefix>_<Object>_Compact_Layout` | |
| Lightning record page | `<PREFIX> - <Object> Record Page` | `<prefix>_<Object>_Record_Page` | `acme_Channel_Number_Record_Page` |
| App | `<Label>` | `<prefix>_<Label_In_Snake_Case>` | `acme_TV_Channel_Inventory` |
| Subflow | `<PREFIX> - <system> - Sub - <Verb> <Subject>` | `<prefix>_<system>_Sub_<Verb>_<Subject>` | `ACME - Sales - Sub - Create Milestone` |
| Screen flow | `<PREFIX> - <system> - Screen - <Verb> <Subject>` | `<prefix>_<system>_Screen_<Verb>_<Subject>` | `ACME - Sales - Screen - Cancel Opportunity` |

**Record-triggered flows.** The default is one flow per business process. Combine only to resolve
an actual conflict: two automations in the same phase on the same object writing the same field.
Then merge the conflicting logic into the object's combined flow for that flavor, and leave the
other process flows alone. Prefer an explicit trigger order or one merged decision over
restructuring an object.

| Flavor | Process-scoped flow (default) | Combined flow (only on conflict, one per object per flavor) |
|---|---|---|
| Sync upsert | `<PREFIX> - <system> - <Object> - <Process> - Sync` | `<PREFIX> - <system> - On <Object> Upserted - Sync` |
| Fast field update (before save) | `<PREFIX> - <system> - <Object> - <Process> - FFU` | `<PREFIX> - <system> - On <Object> Upserted - FFU` |
| Async create | `<PREFIX> - <system> - <Object> - <Process> - Async Create` | `<PREFIX> - <system> - On <Object> Created - Async` |
| Async update | `<PREFIX> - <system> - <Object> - <Process> - Async Update` | `<PREFIX> - <system> - On <Object> Updated - Async` |
| Delete | `<PREFIX> - <system> - <Object> - <Process> - Delete` | `<PREFIX> - <system> - On <Object> Deleted - Async` |

API names follow the label: `acme_Sales_Opportunity_Stage_Dates_FFU`.
`<system>` comes from the profile (package or app area, `Standard` for standard objects, `Custom`
for custom objects that don't extend a package).

**Permission sets: six segments**, separated by ` - `:

1. `<PREFIX>`
2. Vendor of the package targeted: `Salesforce` for standard objects, `<PREFIX>` for our own custom
   objects, otherwise the vendor
3. Package: `Standard`, `Custom`, or the package name
4. Department or team, or `Any`
5. Object(s): one object, a group name (`Core Objects`, `All`), or `None`
6. Operation (`Read`, `Create/Read/Edit`, `View All`, `Modify All`) or purpose (`MFA Exempt`)

Example: `ACME - Salesforce - Standard - Any - None - MFA Exempt`. Descriptions are at most 255
characters.

**Permission set groups: three segments**, for a persona within one product:
`<PREFIX> - <Product> - <Function>`, e.g. `ACME - Standard - Pipeline Management`. If the persona
spans products, or the name is an org-chart title, the group is wrong.

**Custom permissions: four segments**, for a feature flag:
`<PREFIX> - <Vendor> - <Package> - <Feature>`, e.g.
`ACME - Salesforce - Standard - Edit Closed Opportunities`.

Field-level security goes on permission sets, assigned through permission set groups. Never on
profiles.

## Appendix B: UAT scenario format

Written for a business tester who knows the process, not Apex, class names or API names.

**Opening:** a **Business Reason/Impact** in plain text, at most 255 characters, no formatting. It
says what changes for the business and who stops doing manual work; not how it was built and not
what the tester will do. Count the characters.

**Before you start** (only what applies): permission sets or groups to assign, any bypass that must
be off, which app and page to start from. If unavoidable technical terms appear, define each once
in a short terms table.

**Shape:**

```
Scenario N - <what is being exercised>

Setup: <only if test data has preconditions worth calling out>

1. Terse action step, in the users' own process wording
2. ...
3. Wait about N minutes (when an asynchronous process is involved)

Expected Results
- a. Observable outcome on screen, quoting the exact error message where there is one
- b. ...

4. Follow-up step that corrects the input or flips the setting back

Expected Results
- a. ...
```

**Rules:**

- 3 to 7 scenarios unless the change is genuinely broad.
- No rationale: steps and expected results only; the scenario title carries the meaning.
- Field labels, never API names; look labels up in the metadata rather than guessing. Settings are
  given by their Setup path and field label.
- Expected results describe what is on the screen, never what the code did.
- Terse steps, no explanatory clauses.
- Group by configuration state: one scenario walks the negative path and then the positive path for
  one switch position.
- Prefer extra steps inside an existing scenario over a new scenario. No standalone "run this after
  every scenario" scenario.
- Cut anything that tests the platform rather than the change.
- Keep exactly one "neighboring rule still fires" regression scenario.
- Checks use the UI or list views. Queries only when the profile says testers run SOQL; then each
  query is written out in full at the step that uses it, never in an appendix.
- Every step must be physically possible: if a switch hides the inputs, the testable outcome is that
  the process still completes cleanly without them.
- Don't re-test behavior that another ticket already took through UAT.
- When a setting is flipped to something risky, the last step turns it back, in bold.
- Fix gaps in the developer's draft against the actual build and note the resolution; don't pass
  gaps to the tester.

**Status tracking** (in the test-cases file, not the hand-off): per scenario, QA status (Pass, Fail,
Not run), QA evidence (screenshot or output reference), UAT status (blank for the client). A
scenario verified only by reading metadata is "Not run". The hand-off version drops QA status and
evidence.

## Appendix C: Impact analysis checklist

Goal: know what breaks, what behaves differently, and what needs a decision, before building.

**Rules:** read and query only. Repo first, then production for what the repo cannot show (a
custom-only baseline will be missing standard value sets, standard record types, many reports, list
views, sharing and matching rules, and anything owned by a package). Never conclude "no reference"
from the repo alone for those types. Production is the source of truth for data volumes. Aggregate
in queries.

**Six areas**, each with a line in the deliverable even when it is "No impact":

1. **The component itself**, from the org: type, length, required, unique, history tracking, help
   text, relationships.
2. **Data:** affected records by value in one aggregate query; what happens to existing records;
   whether a migration step is needed.
3. **Metadata referencing it:**
   - automation: flows, validation rules, formulas, workflow, Apex including string literals,
     assignment, auto-response and escalation rules;
   - sharing, duplicate and matching rules;
   - UI: layouts, record page visibility and Dynamic Forms, Path, compact layouts, quick actions;
   - analytics: report filters, groupings and columns, dashboards, list views, subscriptions. A
     filter on a removed value silently returns nothing.
4. **Integrations and data loads:** who writes it through the API; tell people from systems with
   field history or created/modified-by grouped by profile.
5. **Users:** what changes in the UI they use; records left holding old values.
6. **Deployment approach:** order, data prerequisites, permission impact, rollback.

**Picklist changes, additionally:**

- the full value list, active and inactive;
- local values or a global/standard value set, and every field sharing it;
- restricted or unrestricted, default and sorting;
- controlling and dependent mappings;
- record type assignments;
- deactivation leaves data unchanged, so the business decides whether to map old values;
- restricted fields reject unknown API values, while unrestricted fields quietly accept them (look
  for values in a `GROUP BY` that are not in the describe);
- Lead conversion mapping.

**Verdicts:** No impact / Needs change (say what) / Needs decision (question plus recommendation;
client decisions tagged `[Client Decision Needed]`).

**Deliverable** `<TICKET>_impact-analysis.md`:

- summary;
- scope and method, including what could not be checked and why;
- findings by area, each with its source and verdict;
- data broken down by the dimension that drives the decision;
- proposed solution per finding, with order and rollback;
- open questions with recommendations.

## Appendix D: Verified platform gotchas

| Symptom | Cause | What to do |
|---|---|---|
| Deploy fails with an unhelpful message on a description | Validation rule, permission set and permission set group descriptions cap at 255 characters | Keep them short; long explanations go in the solution design |
| Validation rule with `ISCHANGED(Field)` fires on every save | Validation rules run after before-save flows, so a field the flow writes is always "changed" | Use the automation toggle (Appendix E). Comparing the stored value to the expected value fires on historical records |
| Cannot "mirror" FLS for a required standard field | Fields like `Opportunity.StageName` have no FLS rows | Say so and use the nearest custom equivalent |
| Users can read a field you never granted | View All Data grants read on every new field | Check before reporting a defect |
| Need a parent field on a Dynamic Forms page | Dynamic Forms accepts `Record.Parent.Field__c` as a field item and in visibility rules; the parent field instance must not have `uiBehavior` | Use the parent field instead of a mirror formula |
| Parent field missing from a classic layout | Classic layouts cannot show parent fields; with Dynamic Forms the layout may only feed mobile | Check whether record pages use Dynamic Forms first |
| `sf project deploy start --source-dir` says "No local changes to deploy" | The folder is outside `packageDirectories` | Use `--metadata-dir` with a `package.xml` |
| Anonymous Apex fails to compile oddly | `bulk` is reserved; `=[` inside a string literal is parsed as inline SOQL | Rename the variable; avoid `=[` in strings |
| Sandbox permissions overwritten after a profile deploy | Repo profiles are baselined from production | Deploy a sparse payload with only the changed permissions |
| Components listed as Created are missing after a failed deploy | Deploys are transactional | Re-query the org after any deploy whose overall status is Failed |
| `sf data query -q "..."` breaks on Windows | `sf` is a `.cmd` shim; `&`, quotes and parentheses break | Write SOQL to a file; `sf data query --file` |
| `sf` misses metadata | The CLI resolves the project from the current folder | Run `sf` from the worktree you are working in |
| Manifest retrieve of reports returns nothing | `Report`, `Dashboard`, `ListView` don't support `*`; plain `-m Report` returns nothing | Query reports and folders; build explicit `Folder/Report` members; list views come with the object |
| Some reports never retrieve | Private and some managed folders are skipped | Describe them via `sf api request rest .../analytics/reports/<id>/describe` |
| `FORBIDDEN` on a report as System Administrator | Report type belongs to a package or feature the user can't use | Record as not inspected; use `Report.LastRunDate` |
| Tempted to pull the access token into a script | Credential leak risk | Use `sf api request rest` (if local policy blocks it, fall back to `COUNT()` queries) |
| Can't find report subscriptions in SOQL | Not an object | `CronTrigger` with `CronJobDetail.JobType = 'A'`, then the analytics notifications API per owner |
| SOQL alias fails | `last` is reserved | Use another alias |
| Inactive picklist values missing | Describe omits them | Compare describe with a `GROUP BY` on the data |
| Dependency API says a field is unused but flows use it | `MetadataComponentDependency` can be incomplete | Always grep as well |
| Report looks unused but feeds snapshots | `Report.LastRunDate` is not a usage signal for snapshot sources | Check analytic snapshot definitions first |

## Appendix E: Automation patterns

**Automation toggle.** When a validation rule must tell "an automation changed this" from "a person
changed this", add a checkbox `<prefix>_Automation_Toggle__c` and have the automation flip it in
the same transaction:

```
AND( ISCHANGED( TheField ), NOT( ISCHANGED( <prefix>_Automation_Toggle__c ) ) )
```

- Flip it (`NOT({!$Record.<prefix>_Automation_Toggle__c})`); never set it to a constant.
- It is shared on purpose: flipping it stands down every rule watching it, so whoever writes the
  automation must know which rules they bypass.
- Keep it off layouts and pages, read-only in permission sets, history tracking off.
- If the automation that flips it is deactivated, deactivate the rule too, or every guarded edit is
  blocked.

**Combining flows.** See Appendix A: combine only to resolve an evidenced conflict, scoped to the
conflicting writes, preferring trigger order or one merged decision.

**Test before believing.** A rule that reads correctly can still fail in practice because an
unrelated flow writes the field it guards. Before stating a platform limitation, deploy a throwaway
version and see.

## Appendix F: Logging standard

For projects using Nebula Logger. The profile supplies tags, scenarios and the flow error prefix.

- **Levels:**
  - ERROR: an action could not complete and the record needs reprocessing.
  - WARN: an acceptable scenario that may need follow-up.
  - INFO: information only.
  - DEBUG: runtime values that speed up troubleshooting.
  - FINE, FINER, FINEST: as the project defines them.
- **Scenario:** one per business process (`Logger.setScenario`); the profile lists them.
- **Tags:** per integration or feature area; entries may carry several.
- **Save methods:**
  - `EVENT_BUS` is the default.
  - `QUEUEABLE` to defer work.
  - `REST` to avoid mixed DML.
  - `SYNCHRONOUS_DML` only with care, since it rolls back on exception.
- **Correlation:** `Logger.getTransactionId()` and `Logger.setParentLogTransactionId()` link
  asynchronous work to its origin.
- **Masking:** `LogEntryDataMaskRule__mdt` for personal data and tokens.
- **Coding:** try/catch around DML and callouts; log at ERROR in the catch; return a generic message
  to the UI; call `Logger.saveLog()` once near the end, or after major phases or in `finally`.
- **Flows:** when an Apex action logs an error, its message back to the flow starts with the
  profile's prefix (default `Error Logged`).
- Reference: the Nebula Logger documentation (`jongpie/NebulaLogger`).

## Appendix G: Project update formats

**Weekly file:** one per week, named for the week-ending Friday (`YYYY-MM-DD.md`), in the profile's
updates folder (default `weekly-roundups/` in the project repo, kept on the default branch,
never in ticket branches).

**Notes:** free-form notes the person gives the agent, either pasted into the session or kept as
dated lines in a `notes.md` in the updates folder.

Front block:

```
Week ending: <date>
Doc: <document title for this week>
Status: draft | issued
```

**Client body:**

```
<Workstream or team>
  <Person>
    <Theme or ticket>
      - outcome bullets
Risks
  - "None", or one bullet per live risk
```

**Working sections** (below the client body, not copied into the document):

- Log, by day, with sources.
- Needs confirmation.
- Still open going into next week.
- Edits the PM made to the draft, so the next draft improves.

A project can add its own sections to either part through the round-up template document.

**Publishing:** copy last week's document, retitle it with the new week-ending date, replace the
body. Never create the document from scratch: the template lives in the document.

**Personal update:** the "Person → themes → bullets" block for one person, plus their risks,
"Needs confirmation" and what's still open. Built from the person's commits, ticket documents,
notes and tracker activity, with sources noted.

**Release notes:** one file per release, `<release-name-or-date>.md`, in the profile's release
notes folder (default `release-notes/` in the project repo).

```
<Release name> - <date>
Target: <org or environment>

What's new
  <Business area>
    - <outcome in plain words> (<TICKET>)
Fixes
  - <what now works> (<TICKET>)
Actions needed
  Users
    - <new steps or changed behavior they must know about>
  Admins
    - <permission set assignments, post-deployment steps, jobs to restart>
Known issues
  - <limitation or open client decision> (<TICKET>)
```

Every entry carries its ticket key. User sections follow the same wording rules as UAT test cases
(field labels, screen wording, no API names). A project can change the sections through its own
template.

## Appendix H: Org assessment artifacts

**Collect** (read-only, aggregate-only):

- Apex classes and triggers, code coverage aggregate, async job failures, scheduled jobs.
- Flows (definitions and versions), validation rules, workflow rules, duplicate rules.
- Connected apps, named credentials, remote site settings.
- Permission sets and assignment counts, profiles with user counts, record types, tabs.
- Reports and dashboards with usage, list view counts.
- Security Health Check score and risks, org limits.
- Login and setup-audit activity as aggregates only.
- Field population counts for key objects.

Raw extracts that hold people data are deleted once aggregated.

**Analyze:**

- `findings.jsonl`: one finding per line with `id`, `title`, `area`, `severity`, `confidence`,
  `business_impact`, `trigger_condition`, `evidence` (commands or file references), `remediation`,
  `effort`, `risk_of_fix`, `blocked_by`.
- **Automation map:**
  - execution order per object, and where it is undefined;
  - per-object write-path tables;
  - contradictions: two automations, one input, different outcomes;
  - overlaps: one rule in two layers;
  - automation that is built but unused;
  - failure surface: fault paths, governor headroom, hard-coded recipients;
  - what is healthy.
- **Remediation plan:** waves (stop the bleeding, consolidate, structural), each with the
  conditions that must be true before the next wave, and a sequencing summary.
- **Open questions:** blocking and non-blocking, each with an owner and a recommendation, plus
  "facts not verified and what would settle each".
- **Cleanup manifests:** destructive plus rollback per batch, validated with a dry run, never run.

**Deliver:**

- Internal exec summary: what the org is, top risks, scorecard, what is healthy and worth
  protecting, scope limits.
- Client readout (page or document): the same, without internal figures or pressure framing.
- Backlog CSV ready for tracker import with columns: ID, Type, Parent ID, Summary, Wave, Priority,
  Status, Estimate (hrs), Depends On, Blocks, Owner/decision-maker, Findings, Detail,
  Acceptance/How to verify, Risk and cautions, Import (y/n), Client notes.
- The generator fails on client files containing local file or repo paths, data inventories (record
  volumes and revenue used as inventory; counts that are the finding itself are allowed) or
  personal data.

**Engage:** readout run-sheet (purpose, attendees, pre-read, agenda, the line that must land for
each risk, the ask, expected pushback with answers, what to capture, follow-up) and decision briefs
grouped into sessions (each with the question, options, recommendation, consequences).

## Appendix I: Ticket document set

House default. The profile can rename, relocate or drop files. Every file starts with the ticket
key: `<TICKET>_<name>.<ext>`.

| File | Written by | Changes? | Purpose |
|---|---|---|---|
| `<TICKET>_initial-requirements.md` | Intake | Never | The ticket exactly as handed over |
| `<TICKET>_latest-requirements.md` | Whoever works the ticket | Yes | Working truth: answers, scope changes, open questions, change log (date, change, decider) |
| `<TICKET>_impact-analysis.md` | Impact analysis | Yes | Appendix C deliverable |
| `<TICKET>_solution-design.md` | Design | Yes | How the requirements are met; decisions; `[Client Decision Needed]` items |
| `<TICKET>_test-cases.md` | Design, then build | Yes | UAT scenarios with QA and UAT status; approval record (who, when) |
| `<TICKET>_implementation-plan.md` | superpowers:writing-plans | Yes | Task-by-task build plan |
| `<TICKET>_package.xml` | Build | Yes | Deploy manifest, explicit members (when the profile uses manifests) |
| `<TICKET>_deployment-guidance.md` | Build | Yes | Pre- and post-deployment steps, order, commands, rollback |
| `<TICKET>_review-summary.md` | Final review | Yes | What changed, decisions made, QA results, deferred findings; reused as the PR/MR description |
