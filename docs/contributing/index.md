# Contributing

How to write or update an almanac page.

The almanac is markdown. Add a file under `docs/<section>/`, link it from the section's `index.md` (and from `mkdocs.yml` `nav` if it's a top-level entry), then `mkdocs serve` locally and check the rendered output before opening a PR.

## Conventions

- **One page, one topic.** If a page grows past ~800 lines, split it.
- **Lead with the why.** Open with one or two paragraphs explaining what the subsystem is for and the load-bearing constraints. Save the reference detail for later sections.
- **Link to code.** Reference files and classes by their actual path: `service/platform/AuditEmitterImpl.java`, not "the audit emitter class somewhere."
- **Mark current-vs-planned.** When a section describes a roadmap surface (e.g. payments today), say so explicitly — readers should never have to guess whether what's documented is shipped.
- **Use Mermaid for state and sequence.** Order lifecycle, member lifecycle, auth flows — all candidates for diagrams. See [Mermaid docs](https://mermaid.js.org/) for syntax.
- **Admonitions for emphasis.** Material supports `note`, `warning`, `info`, `tip`, `example`. Use sparingly — every page tagged "warning" makes none of them warnings.

## Architecture Decision Record (ADR) template

```markdown
# <Title — what was decided, not what was considered>

- **Status:** Accepted | Superseded by [<other Architecture Decision Record>](...) | Deprecated
- **Date:** YYYY-MM-DD
- **Deciders:** <names or roles>

## Context

What problem were we solving? What constraints applied? What did the codebase look like before?

## Decision

What did we decide? State it as a directive, not a discussion.

## Alternatives considered

- **Option A** — why rejected
- **Option B** — why rejected
- **Option C** — why rejected

## Consequences

- What gets easier?
- What gets harder?
- What new invariants does this introduce? (If any are ArchUnit-pinned, link the test.)
- What follow-up work does this create?
```

## Adding a new section

1. Create `docs/<section>/index.md` with the section overview + planned-pages list.
2. Add the section to `mkdocs.yml` under `nav:`.
3. Add a card to `docs/index.md` so the new section shows on the landing page.

## Style notes

- American English (default for the project).
- Backticks for `code`, `paths`, and `Permission codes`.
- Sentence case in headings (not Title Case).
- Avoid emoji unless illustrating a UI element.
