# Salesforce-AI-Marketplace

Claude Code plugins that bring Salesforce delivery practice to AI agents, built on top of
[superpowers](https://github.com/obra/superpowers) and Salesforce's official plugins.

The plugins are skeletons at version 0.1.0: they install, but carry no skills yet. Start with the
[plan](docs/plan.md) and the [build backlog](docs/build-backlog.md).

## Plugins

| Plugin | Contents | Install |
|---|---|---|
| `sfai-core` | Bootstrap, project setup, naming standard, platform gotchas, automation patterns, verify command and Code Analyzer rules, branch lifecycle, review lens, hooks, lesson contribution | Required, everyone |
| `sfai-solutioning` | Ticket stages 1 to 6: intake, impact analysis, UI design gate, solution design, UAT test cases | Architects, BAs, leads |
| `sfai-build` | Ticket stages 7 to 13: Apex test-first, functional QA, safe deploy, logging | Developers, admins, QA |
| `sfai-project-updates` | Personal update, weekly round-up, release notes | PMs, leads, anyone asked for updates |
| `sfai-assessment` | Org health assessment | Architects, leads |

Every plugin depends only on `sfai-core`. Installing any of the other four installs `sfai-core`
with it.

## Install

Install the dependencies first. Each comes from its own marketplace; this marketplace does not
re-list them, so you never end up with two copies.

### 1. Required dependencies

In a Claude Code session:

```
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
/plugin marketplace add forcedotcom/sf-skills
/plugin install salesforce-development@salesforce
```

### 2. Recommended dependencies

```
/plugin install dx-org-lifecycle@salesforce
/plugin install integration@salesforce
```

`dx-org-lifecycle` covers Dev Hub and scratch-org allocation, sandboxes, and permission set
assignment for test personas.

### 3. This marketplace

```
/plugin marketplace add fasozinho91/Salesforce-AI-Marketplace
/plugin install sfai-core@sfai-marketplace
```

Then add the plugins for your role, for example:

```
/plugin install sfai-solutioning@sfai-marketplace
/plugin install sfai-build@sfai-marketplace
```

From a terminal, the same steps use `claude plugin marketplace add <source>` and
`claude plugin install <plugin>@<marketplace>`. Run `/reload-plugins` or start a new session
afterwards.

## Tested versions

[supported-versions.json](supported-versions.json) lists the dependency versions this marketplace
is tested against. The versions there today are the ones installed when the repo was scaffolded;
the evals that confirm them are still pending.

## Contributing

See [docs/contributing.md](docs/contributing.md). Agents working in this repo follow
[CLAUDE.md](CLAUDE.md).

Licensed under [MIT](LICENSE).
