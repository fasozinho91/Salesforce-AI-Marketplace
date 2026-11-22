# Contributing

This repo is public: everything you commit is published. Read [CLAUDE.md](../CLAUDE.md) (the
sources and evidence rule), the [plan](plan.md) and the [build backlog](build-backlog.md) before
you start.

## Set up

1. Install the dependencies as the [README](../README.md#install) describes.
2. Clone this repo and add your clone as a local marketplace, so Claude Code reads the plugins in
   place:

   ```
   claude plugin marketplace add <path-to-your-clone>
   claude plugin install sfai-core@sfai-marketplace
   ```

   Edits take effect at the next session start or after `/reload-plugins`; you don't need to raise
   the version to see them locally.
3. When you are done testing, remove it with `claude plugin marketplace remove sfai-marketplace`,
   which also uninstalls its plugins.

## Pick work

Follow the "what's next" procedure in [CLAUDE.md](../CLAUDE.md#whats-next-procedure): take an
`Open` item whose dependencies are `Done`, and set it to `In progress` with your name in the first
commit of your branch.

## Branches and commits

- One branch per backlog item, cut from an up-to-date `main`, named `feature/<ID>-<short-name>`
  (for example `feature/B01-repo-scaffold`).
- Commit as you work.
- Before opening a PR, run a final review of the whole branch and keep its summary for the PR.
- Another architect reviews every PR ([plan section 12](plan.md#12-how-we-build-and-test-the-skills)).
- When the PR merges, set the item to `Done`.

## Building skills, hooks and scripts

- Every skill is written with `superpowers:writing-skills`: eval scenarios from the skill's "Done
  when" lines first, then the minimal skill
  ([plan section 12](plan.md#12-how-we-build-and-test-the-skills)).
- Skill frontmatter: `name`, a `description` that starts with "Use when..." and names triggers
  only, and `metadata.roles` with roles from the list in
  [plan section 10](plan.md#10-roles-and-the-catalog).
- Scripts and hooks are Node, never bash-only or PowerShell-only, and are built test-first
  ([plan section 3](plan.md#3-constraints)).
- Salesforce guidance needs a source or the `(Architect experience)` tag, as CLAUDE.md sets out.
- Set model and effort on every subagent dispatch ([plan section 3](plan.md#3-constraints)).

## Validate and test

- Run `claude plugin validate . --strict` from the repo root, and
  `claude plugin validate plugins/<plugin> --strict` for each plugin you changed. The root check
  does not open the skill files inside the plugins.
- Run the evals locally and attach the results file to the PR. The harness arrives with backlog
  item B03.

## Versions

- Each plugin carries a semantic version in its own `.claude-plugin/plugin.json`, starting at
  0.1.0. Set it there only, not in `marketplace.json`.
- Raise a plugin's version in any PR that changes it. Installed copies stay on the version they
  have until the string changes.
- Dependency versions live in [supported-versions.json](../supported-versions.json). Raising one
  requires the full eval suite to pass.

## Public-repo safety

- No client names, people's names from client work, email addresses, org IDs, instance URLs or
  credentials, in files, commit messages or PR text.
- Fixtures and examples use the prefix `acme_`. Evals run against scratch orgs or your own
  Developer Edition org, never a client org.
- Never commit record-level extracts of people data; aggregate in the query instead.
- Write in US English. No em dash characters in chat, commit messages or PR text.
