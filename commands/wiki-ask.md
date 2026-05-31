---
description: Ask the project wiki a question. Reads knowledge/wiki/index.md, locates relevant articles, and answers with citations. Optionally archive the answer back into the wiki.
argument-hint: <question>
---

Load the `dev-wiki` skill at `${CLAUDE_PLUGIN_ROOT}/skills/dev-wiki/SKILL.md` and execute the Query operation.

Steps:

1. If `$ARGUMENTS` is empty, ask the user with AskUserQuestion for the question.
2. If `knowledge/wiki/index.md` does not exist, tell the user "No wiki yet — run /wiki-ingest first." Stop.
3. Read `knowledge/wiki/index.md` to locate articles relevant to the question. Match by title, summary, and topic.
4. Read the relevant articles (no more than is needed to answer — don't grep the whole wiki).
5. Synthesize the answer. Prefer wiki content over your training knowledge. If the wiki contradicts your priors, the wiki wins for this project.
6. Cite every concrete claim with a markdown link to the article it came from, using project-root-relative paths: `[Article Title](knowledge/wiki/topic/article.md)`.
7. If the wiki has no relevant content, say so — do not invent. Suggest the user run `/wiki-ingest` to add the missing knowledge.
8. After the answer, offer two follow-ups inline:
   - "Archive this answer as a wiki page?" — on yes, write a new archive page per `references/archive-template.md`, update the index with `[Archived]` prefix, append a `query | Archived: <title>` line to `log.md`, and report the path.
   - "Ingest a related source?" — on yes, dispatch `/wiki-ingest` with the source the user names.

Do NOT write files unless the user accepts an archive or ingest. A plain query is read-only.

When generating archive paths, convert in-conversation citations from project-root-relative (`knowledge/wiki/topic/article.md`) to file-relative (`article.md` for same-directory, `../<other-topic>/article.md` for cross-topic) before writing.
