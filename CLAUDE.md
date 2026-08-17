# CLAUDE.md

This repo hosts a personal knowledge-base wiki maintained by Claude Code, following the "LLM Wiki" pattern: raw sources are ingested once, and the wiki is a persistent, compounding artifact that stays current rather than being re-derived from scratch on every question.

Aleksei reads the wiki on GitHub (the repo's built-in Wiki tab), side by side with a Claude Code session. Claude does the reading, summarizing, cross-referencing, and filing. Aleksei curates sources, directs the analysis, and asks the questions.

## Directory structure

- `raw/` — immutable source material (articles, notes, transcripts, journal entries). **Local-only: gitignored, never committed or pushed.** This repo is public, and sources are rarely ours to redistribute (copyrighted articles, vendor specs with no-redistribution clauses, etc.) — auditing each source's license before deciding whether to commit it doesn't scale, so the simple rule is nothing under `raw/` ever goes into git. The wiki (built from these sources) is the only shareable, pushed artifact. Once a file lands here it is never edited by Claude — if something changes, add a new file or a follow-up note rather than modifying the original. Images/attachments go in `raw/assets/`. Since `raw/` isn't tracked, treat it as ephemeral to *this repo's history* — Aleksei is responsible for his own backup of the source files themselves.
- `wiki/` — a local clone of this repo's GitHub Wiki (`git@github.com:rybalkin-an/llm-wiki.wiki.git`). This is where every wiki page actually lives; it's **not** tracked by this repo's git (see `.gitignore`) since it's its own separate git repository with its own remote. Browse the live result at the repo's Wiki tab on GitHub. If `wiki/` doesn't exist locally, clone it first: `git clone git@github.com:rybalkin-an/llm-wiki.wiki.git wiki`.
- `wiki/Home.md` — the wiki's landing page, doubling as the index: a catalog of every other page, grouped by category, each with a one-line summary. Read this first when answering a query, before opening individual pages.
- `wiki/Log.md` — append-only chronological log of ingests, queries, and lint passes.

Subfolders/categories within the wiki are not fixed — propose new ones as the wiki's actual shape becomes clear rather than forcing a rigid taxonomy up front. Starting categories: `People`, `Health`, `Projects & Goals`, `Concepts`, `Journal`, `Sources`.

**Relationship to `.github/workflows/*.yml`:** those workflows also push to this same GitHub Wiki, but for a different purpose — they sync code/doc changes from `src/**`/`docs/**` on push to `main`, automatically, without Aleksei in the loop. This personal-KB workflow is the manual/interactive counterpart, driven by sources Aleksei feeds in during a session. Both write to the same wiki repo; they shouldn't collide in practice since the CI path only touches pages related to code/doc diffs, but keep it in mind if a page unexpectedly changed.

## Workflows

### Ingest (adding a new source)

1. Read the new file(s) in `raw/`.
2. Discuss key takeaways with Aleksei before writing anything — don't file silently.
3. In the `wiki/` clone: write or update a source summary page under `Sources/`.
4. Update or create the entity/concept/journal pages elsewhere in `wiki/` that the source actually touches. A single source might touch several pages — that's expected.
5. Update `wiki/Home.md` with any new or changed pages.
6. Append an entry to `wiki/Log.md`.
7. Commit and push inside the `wiki/` clone (it has its own remote — do not try to commit wiki pages into the main repo).
8. Flag contradictions with existing wiki content explicitly instead of silently overwriting — ask which version is right.

Prefer ingesting one source at a time with Aleksei staying involved, unless told to batch-process with less supervision.

### Query (asking a question)

1. Read `wiki/Home.md` first to find candidate pages — don't scan the whole `wiki/` tree.
2. Read only the relevant pages to answer.
3. Cite which wiki pages the answer draws on.
4. If the answer is substantial (a comparison, an analysis, a synthesis worth keeping), ask whether to file it back into `wiki/` as a new page. If yes, follow the same Home/Log update and commit/push steps as an ingest.

### Lint (health check — run on request, not automatically)

Check for:
- Contradictions between pages
- Stale claims that newer sources have superseded
- Orphan pages with no inbound links
- Concepts mentioned repeatedly but lacking their own page
- Missing cross-references

Report findings and propose fixes; don't apply them without confirming first.

## Conventions

- Wiki pages are plain markdown. GitHub Wiki natively supports `[[Page Name]]` links for cross-references — use them.
- Add YAML frontmatter (tags, date) where it's actually useful for later filtering — optional, add as the need arises, not preemptively.
- Prefer smaller, incremental edits Aleksei can follow (review the diff, or refresh the GitHub Wiki page) over large rewrites of many pages at once.
- `wiki/` is a separate git repo from this one — commits there use their own history, independent of this repo's commits.

## Note

This file is a starting scaffold, not a finished spec — update it as we learn what actually works for this specific wiki. The directory layout and categories above are a reasonable default for a personal knowledge base, not a fixed requirement.
