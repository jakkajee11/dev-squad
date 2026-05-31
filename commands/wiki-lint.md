---
description: Run wiki quality checks — deterministic auto-fixes (index consistency, internal links, raw references, see-also pruning) plus heuristic findings reported for human review.
---

Load the `dev-wiki` skill at `${CLAUDE_PLUGIN_ROOT}/skills/dev-wiki/SKILL.md` and execute the Lint operation.

Steps:

1. If `knowledge/wiki/index.md` does not exist, tell the user "No wiki yet — run /wiki-ingest first." Stop.

2. **Deterministic auto-fix pass:**
   - **Index consistency** — compare `index.md` against actual files in `knowledge/wiki/` (excluding `index.md` and `log.md`).
     - File exists but not in index → add entry with summary from the article's lead paragraph (or `(no summary)`), Updated date from frontmatter (or file mtime).
     - Index entry points to nonexistent file → mark as `[MISSING]`. Do not delete.
   - **Internal links** — for every markdown link in `wiki/` article files (body + Sources frontmatter), excluding Raw field links and excluding `index.md`/`log.md`:
     - Target missing → search `wiki/` for a file of the same name.
       - Exactly one match → fix the path.
       - Zero or multiple → report.
   - **Raw references** — every link in a `raw:` frontmatter field must resolve to a file under `knowledge/raw/`:
     - Target missing → search `knowledge/raw/` for a file of the same name. Same fix-or-report logic.
   - **See Also pruning** — within each topic directory:
     - Add obviously missing cross-references between articles that cite each other or share `related_runs`.
     - Remove See Also links to deleted files.

3. **Heuristic report pass** — do NOT auto-fix; surface findings:
   - Factual contradictions across articles (same claim, different conclusions, no Conflicts annotation).
   - Outdated claims superseded by newer ingests.
   - Missing conflict annotations where sources disagree.
   - Orphan pages with no inbound links from other wiki articles.
   - Missing cross-topic references.
   - Concepts frequently mentioned across articles but lacking a dedicated page.
   - Archive pages whose cited source articles have been substantially updated since archival.

4. Append a single log entry to `knowledge/wiki/log.md`:
   ```
   ## [YYYY-MM-DD] lint | <N> issues found, <M> auto-fixed
   ```

5. Report to the user:
   - **Auto-fixed** — count + brief list (e.g. "fixed 3 broken internal links, added 2 missing index entries").
   - **Findings (no auto-fix)** — list each with article path, type of issue, and one-line description. Don't make the user dig.
   - **Suggested next steps** — e.g. "consider creating `patterns/idempotency-keys.md` — mentioned in 4 articles but no dedicated page".

Lint never deletes content. It marks, fixes paths, and reports.
