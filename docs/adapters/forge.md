---
title: Forge adapter
summary: The code-forge host, repo, token location, and host quirks; the executable half is scripts/forge.
updated: 2026-09-02
status: living
---

# Forge adapter

All PR actions go through [`scripts/forge`](../../scripts/forge) (run
`scripts/forge help` for the interface). Core kit files invoke those verbs and never
call the forge API directly — so moving hosts means swapping the script and this file.

This project's forge is **GitHub**. The included `scripts/forge` implements the
portable verb set on top of the [`gh`](https://cli.github.com/) CLI. Auth is handled
by `gh`'s keyring locally; CI passes a `FORGE_TOKEN` (see below).

## Configuration

Set in the gitignored `.secrets/forge.env` (see
[`.secrets/forge.env.example`](../../.secrets/forge.env.example)):

- `FORGE_REPO` — `owner/repo` on GitHub (e.g. `snappautomotive/basset-landing`).
- `FORGE_TOKEN` — optional locally. When set, `scripts/forge` exports it as `GH_TOKEN`
  and `gh` uses it instead of its keyring auth. In CI this is set to
  `${{ secrets.GITHUB_TOKEN }}` (see `.github/workflows/auto-merge.yml`). Never commit
  or print it.

Local prerequisites:

- `gh` installed and authenticated (`gh auth login`).
- The authenticated account needs `repo` scope (to open/edit PRs, manage labels).
- `jq` on `$PATH`.

## Verbs (the portable interface)

`pr-create` · `pr-edit` · `prs` · `status` · `comments` · `reply` · `comment` ·
`upload` · `label` · `unlabel` · `seed-labels`

`seed-labels` idempotently creates the six workflow labels the kit's review gate uses
(`in-qa-review`, `in-human-review`, `allows-auto-merge`, `auto-merge`, `requires-adr`,
`contains-adr`). Run it once per repo at setup.

## Host quirks (GitHub)

Facts core files rely on; revalidate these when porting to another host:

- **Rendering**: GitHub renders GFM. Hard-wrapping bodies is safe (paragraphs collapse
  correctly), unlike Forgejo. Blank lines still separate blocks; lists must not have
  blank lines between items in the same list.
- **Bodies with backticks or `$(...)`** go through `@<file>` or stdin `-`, never inline
  — an inline literal body is mangled by the caller's shell before `forge` runs.
- **Review comments anchor to (SHA, path, position)**: bring branches current by
  **merging** `main` in, never rebase + force-push — a force-push moves the SHA and
  every inline comment becomes "outdated" (still visible, but the anchor is stale).
- **Attachments**: GitHub has **no public API for issue attachments**. `forge upload`
  uploads the file as a **secret gist** (unlisted but URL-accessible) and returns its
  raw content URL for embedding in markdown. Anyone with the URL can view it; do not
  upload files containing credentials or private data.
- **Branch protection**: the up-to-date-with-main rule is named
  `require_last_push_approval` / `require_branches_to_be_up_to_date`; required checks
  are attached by CI job name (`check`, `quality-gate` in this project).
- **Auto-merge** (native): `gh pr merge --auto --squash` enables GitHub's auto-merge,
  which merges the PR when required checks pass. The `.github/workflows/auto-merge.yml`
  workflow enables it once the `auto-merge` label lands and the deny-list check passes.
- **Rate limits**: `gh` handles pagination and the 5000 req/h primary limit; the
  wrapper uses `--paginate` where needed.
- Every comment the wrapper posts carries an `[agent] ` prefix so an agent's message is
  never mistaken for the human whose token authors it.

## CI token

The auto-merge workflow uses the default `GITHUB_TOKEN` provided to the workflow run;
no separate token secret is required for the merge itself. For workflows that need to
trigger other workflows (not this project yet), a PAT would be added as a repository
secret.
