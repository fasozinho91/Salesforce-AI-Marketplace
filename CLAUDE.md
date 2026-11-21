# Salesforce-AI-Marketplace: project instructions

Start with [docs/plan.md](docs/plan.md) and [docs/build-backlog.md](docs/build-backlog.md).

## Sources and evidence (standing rule)

When building the plugins, everything we add must be backed by a credible source or by this
repo's architects' own experience. This covers skills, agents, commands, hooks, templates and
reference docs. Use sources whenever possible.

### Source order

1. **architect.salesforce.com**: preferred, and treated as best practice. Start from the
   [Well-Architected framework](https://architect.salesforce.com/docs/architect/well-architected/guide/framework.html).
2. **salesforceben.com**: the only other acceptable source. Use it when architect.salesforce.com
   does not cover the point.

No other sources. That includes other Salesforce sites (for example help.salesforce.com,
developer.salesforce.com, Trailhead), blogs, forums and AI output.

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
