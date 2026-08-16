# llm-wiki

## Automated wiki sync

Two workflows keep this repo's GitHub Wiki in sync with code changes on merge to `main`:

- `.github/workflows/wiki-sync-detect.yml` — triggers on push to `main`, gated by a `paths:` filter (currently `src/**`, `docs/**`, `README.md`, adjust as the repo grows). Its only job is to exist and succeed.
- `.github/workflows/update-wiki.yml` — triggers on that workflow's completion (`workflow_run`), and does the actual work.

**Why two files:** `anthropics/claude-code-action@v1` refuses to run directly on a `push` event (it only accepts issue/PR/dispatch/schedule/`workflow_run` events). Splitting the path-filtering (`push`) from the Claude step (`workflow_run`) works around that.

**How it works:** once `wiki-sync-detect.yml` succeeds, `update-wiki.yml` checks out the triggering commit, computes a diff limited to the watched paths, hands that diff plus a checkout of the wiki to Claude Code, and lets it update only the wiki pages the diff actually affects. If anything changed, it commits and pushes straight to the wiki repo (wikis don't support pull requests). No matching paths, or no resulting edits, means no commit.

**One-time setup required before this runs successfully:**

1. Create the wiki's first page manually (Repo → Wiki tab → "Create the first page") — GitHub doesn't initialize `<repo>.wiki.git` until a page exists.
2. Generate a Claude Code OAuth token locally (`claude setup-token`, needs a Pro/Max login) and add it as repo secret `CLAUDE_CODE_OAUTH_TOKEN`. No `ANTHROPIC_API_KEY` is used anywhere.
3. Generate a classic GitHub PAT with `repo` scope and add it as repo secret `WIKI_SYNC_TOKEN` — the default `GITHUB_TOKEN` can't push to the wiki repo, and wiki access isn't reliably supported on fine-grained PATs.

Full details and rationale are documented as comments at the top of `update-wiki.yml` and `wiki-sync-detect.yml`.
