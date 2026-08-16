# llm-wiki

## Automated wiki sync

`.github/workflows/update-wiki.yml` keeps this repo's GitHub Wiki in sync with code changes on merge to `main`.

**How it works:** a push to `main` that touches a watched path (currently `src/**`, `docs/**`, `README.md` — see the `paths:` filter in the workflow, adjust as the repo grows) computes a diff limited to those paths, hands that diff plus a checkout of the wiki to Claude Code, and lets it update only the wiki pages the diff actually affects. If anything changed, the workflow commits and pushes straight to the wiki repo (wikis don't support pull requests). No matching paths, or no resulting edits, means no commit.

**One-time setup required before this runs successfully:**

1. Create the wiki's first page manually (Repo → Wiki tab → "Create the first page") — GitHub doesn't initialize `<repo>.wiki.git` until a page exists.
2. Generate a Claude Code OAuth token locally (`claude setup-token`, needs a Pro/Max login) and add it as repo secret `CLAUDE_CODE_OAUTH_TOKEN`. No `ANTHROPIC_API_KEY` is used anywhere.
3. Generate a classic GitHub PAT with `repo` scope and add it as repo secret `WIKI_SYNC_TOKEN` — the default `GITHUB_TOKEN` can't push to the wiki repo, and wiki access isn't reliably supported on fine-grained PATs.

Full details and rationale are documented as comments at the top of the workflow file.
