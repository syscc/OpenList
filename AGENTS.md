# AGENTS.md

This repository is `syscc/OpenList`, a personal fork of `OpenListTeam/OpenList`.
It follows upstream OpenList while maintaining a personal beta image workflow.

## Local Work Limits

- During testing and troubleshooting, do not compile the source into local binaries.
- Do not run `go build`, `bash build.sh`, `docker build`, release packaging, or image builds locally.
- Local work should be limited to source inspection, YAML/text validation, Git/GitHub status checks, and source-mode runtime debugging when needed.
- Build validation is expected to happen in GitHub Actions.

## Remotes

- `origin` points to upstream `OpenListTeam/OpenList`; use it for reading upstream state only, do not push to it.
- `fork` points to `syscc/OpenList`; push only to this remote when updating this fork.
- `xrgzs` is reference-only; do not push to `xrgzs/OpenList` or create PRs there.

## Branch Model

- `main`: follows upstream, while keeping fork management files and workflows.
- `beta`: generated runtime branch, built from `main` plus open personal PRs that have not been merged upstream.
- `feat/*` and `fix/*`: PR branches for upstream OpenList. Base them on upstream `OpenListTeam/OpenList:main` and avoid fork-only README/workflow/management commits.

Expected state:

```text
main = upstream beta-validated base + fork management files
beta = main + open, non-draft upstream PRs whose head repo is syscc/OpenList
```

Do not keep unmerged feature code directly on `main`; keep it in PR branches and let `beta` aggregate it.

## Beta PR Rules

- `Sync beta branch` normally discovers PRs in `OpenListTeam/OpenList` where:
  - author is `syscc`
  - state is open
  - draft is false
  - head repository is `syscc/OpenList`
- `.github/beta-prs.txt` is an optional allow-list:
  - empty or absent: include every matching open PR
  - contains PR numbers: include only those PRs
- After upstream merges a PR, it is no longer open and will be dropped from the next generated `beta`.

## Workflows

- `Sync upstream main`
  - Checks the latest successful upstream beta Docker push run.
  - Moves `main` forward only to an upstream SHA that has successfully built beta Docker.
  - Triggers `Sync beta branch` after a successful update.

- `Sync beta branch`
  - Regenerates `beta` from `main`.
  - Merges matching unmerged PRs.
  - Writes `.github/beta-state.txt` with the selected base and PR state.
  - Pushes only when the generated `beta` tree changes.

- `Beta Release (Docker)`
  - Builds GHCR images only from `beta`.
  - Uses path filters so unrelated changes do not trigger image builds.
  - Publishes `ghcr.io/syscc/openlist` and `ghcr.io/syscc/alist`.

- `Beta Release builds`
  - Builds beta release binaries only from `beta`.
  - Also uses path filters.

- `Test Build`
  - Runs upstream PR build checks.

## Upstream Collaboration

### Issues

Before creating an issue, review the available issue templates in `.github`.

When drafting an issue:

- Use the most appropriate template.
- Follow the template structure.
- Fill in all required sections.
- Remove optional or not-applicable sections when the template says to.
- Do not invent reproduction steps, logs, screenshots, or expected behavior.

### Pull Requests

Before creating a pull request, read `.github/PULL_REQUEST_TEMPLATE.md`.

When drafting a pull request:

- Follow the template structure.
- Use the title format required by the template.
- Fill in or remove each section according to the template guidance.
- Include testing details, or explicitly explain why testing was not run.
- Do not invent testing results.
- Do not claim validation, verification, or review steps that were not actually performed.

### Git Commits

When creating commits, follow the repository `git-commit` skill rules:

- Use Conventional Commits title format: `type(scope): subject`.
- Allowed types: `feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, `build`, `ci`, `chore`, `revert`.
- Use a meaningful scope based on the main module, package, or feature.
- Write the subject in imperative mood and describe the actual change.
- Use a concise Markdown list in the commit body when a body is useful, with each item describing one key change.
- Do not invent changes that are not present in the diff.
- Do not describe behavior, refactors, fixes, or tests that are not reflected in the commit.

Include at most one `Co-authored-by` trailer that matches the AI assistant actually used to produce the change.

Examples:

- `Co-authored-by: Codex <267193182+codex@users.noreply.github.com>`
- `Co-authored-by: GitHub Copilot <copilot@github.com>`
- `Co-authored-by: Claude <81847+claude@users.noreply.github.com>`

If you are not one of the listed assistants, do not add a `Co-authored-by` trailer. Ask the human collaborator for the exact trailer instead.

## Checks

Run these first when doing a full repository check:

```bash
git status --short
git ls-remote --heads fork main beta
ruby -e 'require "yaml"; ARGV.each { |f| YAML.load_file(f); puts "ok #{f}" }' .github/workflows/*.yml
git diff --check refs/heads/main refs/heads/beta
gh workflow list --repo syscc/OpenList --all
gh run list --repo syscc/OpenList --limit 12
gh pr list --repo OpenListTeam/OpenList --author syscc --state open
```

Use these when inspecting beta generation:

```bash
git diff --stat refs/heads/main..refs/heads/beta
cat .github/beta-state.txt
```

## Operational Notes

- Do not open upstream PRs from `beta`.
- Do not base upstream PRs on `beta`.
- Do not create PRs from GitHub compare pages like `xrgzs/OpenList/compare/main...syscc:beta`; those are only compare views.
- Force-push `main` only when explicitly needed to rewrite fork management history, and only after confirming it will not overwrite user work.
- `beta` is generated and may be force-pushed by workflows or maintenance operations.
- Do not commit the local untracked `.spec-workflow/` directory unless explicitly requested.
