# llm-wiki

## Personal knowledge base

This repo is also a personal knowledge-base wiki, following the "LLM Wiki" pattern: `raw/` holds immutable source material (tracked in this repo), the actual wiki pages live in this repo's GitHub Wiki (browse them under the **Wiki** tab), and `CLAUDE.md` is the schema/workflow file that tells Claude Code how to ingest sources, answer queries, and keep the wiki healthy. See `CLAUDE.md` for the full workflow.

`wiki/` in the working tree is a local clone of the GitHub Wiki (`git clone git@github.com:rybalkin-an/llm-wiki.wiki.git wiki`) — it's gitignored here since it's its own separate git repo with its own history and remote.

This shares the same GitHub Wiki as the automated sync below, just via a different path: that automation pushes code/doc-driven updates on every merge to `main`; this personal-KB workflow pushes source-driven updates during an interactive Claude Code session. Both write to the same wiki repo but shouldn't collide in normal use.

### Example workflow

**Ingest** — you drop a source in `raw/` and ask Claude to process it:

```
> I've added raw/2026-08-sleep-article.md — can you ingest it?
```

Claude reads the file, talks through the key points with you first, then:
- writes `wiki/Sources/2026-08-Sleep-Article.md` summarizing it, with a link back to the raw file
- updates `wiki/Health.md` (creating it if it doesn't exist) with the new claims, e.g. adding a `## Sleep` section referencing `[[2026-08-Sleep-Article]]`
- adds a one-line entry under **Health** and **Sources** in `wiki/Home.md`
- appends to `wiki/Log.md`:
  ```
  ## [2026-08-16] ingest | 2026-08 Sleep Article
  Added Sources/2026-08-Sleep-Article.md, updated Health.md (new Sleep section).
  ```
- commits and pushes inside the `wiki/` clone

**Query** — you ask a question that spans existing pages:

```
> What have I read about sleep and how does it connect to my stress notes?
```

Claude reads `wiki/Home.md`, finds `Health.md` and `Journal/2026-07-Stress.md` are both relevant, reads just those two, and answers with citations like `(see [[Health]], [[2026-07-Stress]])`. If the answer itself is worth keeping — say you asked for a synthesis of everything you know about sleep — Claude offers to file it back as a new page (e.g. `wiki/Concepts/Sleep.md`), same as an ingest.

**Lint** — periodically, or when something feels off:

```
> Can you health-check the wiki?
```

Claude scans for things like a `Sources/2026-08-Sleep-Article.md` that nothing links to (orphan page), or a claim in `Health.md` that a newer source contradicts, and reports them for you to confirm before it fixes anything.

## Automated wiki sync

Two workflows keep this repo's GitHub Wiki in sync with code changes on merge to `main`:

- `.github/workflows/wiki-sync-detect.yml` — triggers on push to `main`, gated by a `paths:` filter (currently `src/**`, `docs/**`, `README.md`, adjust as the repo grows). Its only job is to exist and succeed.
- `.github/workflows/update-wiki.yml` — triggers on that workflow's completion (`workflow_run`), and does the actual work.

**Why two files:** `anthropics/claude-code-action@v1` refuses to run directly on a `push` event (it only accepts issue/PR/dispatch/schedule/`workflow_run` events). Splitting the path-filtering (`push`) from the Claude step (`workflow_run`) works around that.

**How it works:** once `wiki-sync-detect.yml` succeeds, `update-wiki.yml` checks out the triggering commit, computes a diff limited to the watched paths, hands that diff plus a checkout of the wiki to Claude Code, and lets it update only the wiki pages the diff actually affects. If anything changed, it commits and pushes straight to the wiki repo (wikis don't support pull requests). No matching paths, or no resulting edits, means no commit.

**One-time setup required before this runs successfully:**

1. Create the wiki's first page manually (Repo → Wiki tab → "Create the first page") — GitHub doesn't initialize `<repo>.wiki.git` until a page exists.
2. Install the Claude GitHub App on this repo: [github.com/apps/claude](https://github.com/apps/claude) → Install (or Configure) → select this repository. `claude-code-action@v1` exchanges an OIDC token for a GitHub App installation token on every run and fails with "Claude Code is not installed on this repository" without it — this is separate from the OAuth token below.
3. Generate a Claude Code OAuth token locally (`claude setup-token`, needs a Pro/Max login) and add it as repo secret `CLAUDE_CODE_OAUTH_TOKEN`. No `ANTHROPIC_API_KEY` is used anywhere.
4. Generate a classic GitHub PAT with `repo` scope and add it as repo secret `WIKI_SYNC_TOKEN` — the default `GITHUB_TOKEN` can't push to the wiki repo, and wiki access isn't reliably supported on fine-grained PATs.

Full details and rationale are documented as comments at the top of `update-wiki.yml` and `wiki-sync-detect.yml`.

Note: this repo has no `src/` or `docs/` folders yet — add them and revisit the `paths:` filters in both workflow files once real code/doc structure exists.
