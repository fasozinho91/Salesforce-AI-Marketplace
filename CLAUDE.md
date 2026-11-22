# Salesforce-AI-Marketplace: project instructions

Start with [docs/plan.md](docs/plan.md) and [docs/build-backlog.md](docs/build-backlog.md).

## Sources and evidence (standing rule)

When building the plugins, everything we add must be backed by a credible source or by this
repo's architects' own experience. This covers skills, agents, commands, hooks, templates and
reference docs. Use sources whenever possible.

### Source order

1. **architect.salesforce.com**: preferred, and treated as best practice. Start from the
   [Well-Architected framework](https://architect.salesforce.com/docs/architect/well-architected/guide/framework.html).
2. **help.salesforce.com**: official Salesforce Help.
3. **developer.salesforce.com**: official Salesforce developer documentation.
4. **salesforceben.com**: use it when none of the above covers the point.

No other sources. That includes other Salesforce sites (for example Trailhead), blogs, forums
and AI output.

### Conflicts between sources

The order above sets where to look first and which source to prefer, not how to settle a
disagreement. If two sources conflict, even when one of them is architect.salesforce.com, do
not pick one yourself. Bring it to the user for an architect decision, with both claims and
both deep links.

### Experience-based guidance

When neither source covers a point, the guidance may still go in if it comes from this repo's
architects. Mark it inline, right after the statement, with the tag `(Architect experience)`.

### Citation format

- Each file that makes claims ends with a `## Sources` section.
- Each entry is a deep link to the specific page, never the site root, with the page title as
  the link text and the claim it backs:
  `- [Page title](https://architect.salesforce.com/...): what it backs`
- Before citing, open the page and confirm it says what the claim says. Never cite a page you
  have not read.
- Files that make no claims (manifests, config) do not need a Sources section.

### Unsourced claims

If a claim cannot be sourced or attributed to architect experience, flag it to the user. Do not
keep it silently, and do not invent a citation.

## What's next (procedure)

When someone asks "what's next?" in this repo, follow
[plan.md section 13](docs/plan.md#13-build-order-and-whats-next):

1. Read [docs/build-backlog.md](docs/build-backlog.md).
2. List items with status `Open` whose dependencies are all `Done`, highest priority first (P0
   before P1, then by ID).
3. Show the top few with their plan section and ask which to take (or confirm the top one).
4. Mark it `In progress` with the person's name in the same PR as the first change, so two
   architects don't pick the same item.
5. Build it with the method in
   [plan.md section 12](docs/plan.md#12-how-we-build-and-test-the-skills).
6. When the PR merges, mark it `Done`.

How to contribute (branches, versions, evals, public-repo safety) is in
[docs/contributing.md](docs/contributing.md).
