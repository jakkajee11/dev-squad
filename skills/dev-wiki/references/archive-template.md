# Archive Page Template

Use this format for archive pages — synthesized answers saved from Query operations when the user explicitly asks to archive. Archive pages live alongside articles in `knowledge/wiki/<topic>/` but follow different rules:

- They are **point-in-time snapshots** of a synthesized answer.
- They are **never cascade-updated** by future ingests.
- They have **no Raw field** (content derives from wiki articles, not raw sources).
- They are **never merged** — every archive is a new file.

## File naming

- `knowledge/wiki/<topic>/<query-topic-slug>.md`
- Slug reflects the question or topic the answer addresses, not a single concept.
- Examples: `auth-flow-comparison.md`, `2026-q2-tech-debt-recap.md`, `recurring-review-issues.md`.

## Format

```markdown
---
title: <Archive title — what question this answers>
topic: <Matches parent directory under knowledge/wiki/>
kind: archive
sources: [<Article Title>](<article>.md); [<Article Title>](../<other-topic>/<article>.md)
archived: <YYYY-MM-DD — date the user asked to archive>
updated: <YYYY-MM-DD — same as archived; archives don't cascade>
tags: <Optional comma-separated tags; omit if none>
---

# <Archive title>

> Archived synthesis from <YYYY-MM-DD>. Cited articles may have evolved since.

<Lead paragraph: restate the question and give the bottom-line answer.>

## <Section heading>

<Synthesized body content. Cite each claim back to the wiki article it came
from using inline links. Path style: relative to the archive page's location.>

## <Another section>

<More synthesis.>

## Sources

- [<Article Title>](<article>.md) — what this article contributed to the answer
- [<Article Title>](../<other-topic>/<article>.md) — what this article contributed to the answer
```

## Field rules

- **kind: archive** — required marker. Lint and Index use this to apply archive-specific rules (no cascade, `[Archived]` prefix in index).
- **sources** — markdown links to wiki articles, semicolon-separated. Path is relative to the archive page's location (same-directory: `article.md`; cross-topic: `../<other-topic>/article.md`).
- **No raw field** — archive content is downstream of wiki articles, not raw sources.
- **archived** — date the archive was created. Never changes.
- **updated** — set equal to `archived` and do not modify. Archives are immutable snapshots; for a fresh synthesis create a new archive page.

## Citation path conversion

When the user asks to archive an answer generated in conversation:
- In-conversation citations use **project-root-relative paths**: `knowledge/wiki/topic/article.md`.
- Archive page citations must use **file-relative paths** from the archive page's location.
  - Same topic directory: `article.md`
  - Different topic: `../<other-topic>/article.md`

Rewrite every link during the archive write.

## Example

```markdown
---
title: Recurring Review Issues in 2026-Q2
topic: lessons
kind: archive
sources: [Constant-Time Secret Comparison](constant-time-secret-comparison.md); [Migration Backfill Before Read-Flip](migration-backfill-before-read-flip.md); [API Key Authentication](../subsystems/api-key-authentication.md)
archived: 2026-06-30
updated: 2026-06-30
tags: retrospective, code-review, lessons
---

# Recurring Review Issues in 2026-Q2

> Archived synthesis from 2026-06-30. Cited articles may have evolved since.

This archive answers "which review-issue patterns repeated across multiple loops in Q2 2026, and what rules did the squad adopt to prevent them?" Short answer: three patterns dominated — secret-handling foot-guns, migration ordering mistakes, and FE/BE wire-shape mismatches.

## Secret handling

The most-flagged pattern was `==` comparison on tokens / API keys / HMAC digests. Captured as a hard rule in [Constant-Time Secret Comparison](constant-time-secret-comparison.md)...

## Migration ordering

Adding a NOT NULL column without an explicit backfill stage hit production twice. The expand/contract sequence is now mandatory — see [Migration Backfill Before Read-Flip](migration-backfill-before-read-flip.md)...

## Sources

- [Constant-Time Secret Comparison](constant-time-secret-comparison.md) — captured the secret-comparison rule and its detection signature
- [Migration Backfill Before Read-Flip](migration-backfill-before-read-flip.md) — captured the expand/contract migration sequence
- [API Key Authentication](../subsystems/api-key-authentication.md) — the subsystem that first surfaced the secret-comparison issue
```

## Notes

- Lead with the blockquote banner noting the archive date — sets reader expectations.
- The `Sources` section at the bottom is required, even though sources are in frontmatter.
- Inline citations during the body are recommended when a specific claim is contested or when the reader needs to drill into one source.
- If a follow-up question changes the answer materially, create a **new archive page** with a fresh date. Do not edit the old archive.
- In Obsidian, archive pages with `kind: archive` can be filtered out of the graph view via a Dataview query if they start cluttering the visualization.
