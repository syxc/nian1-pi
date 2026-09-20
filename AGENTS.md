# PROJECT KNOWLEDGE BASE

**Generated:** 2026-09-20

## OVERVIEW

Project: **nian1-pi-preset** — a Pi preset (distributed by git, not npm) that
bundles personal Pi coding-agent resources (extensions / skills / prompts /
themes) plus 22 third-party Pi packages (17 npm + 5 git), so one `pi install`
loads everything.
Stack: Node.js ESM (`"type": "module"`), no build step, no framework, no
first-party code beyond resource placeholders. There is no test suite. The only
automation is a manually triggered GitHub Actions workflow that refreshes
dependencies and opens a pull request.

## STRUCTURE

*   `package.json` — the single source of truth: version, `pi` manifest, `dependencies`,
    `bundledDependencies`, `files`.
*   `extensions/`, `skills/`, `prompts/`, `themes/` — own resources (currently `.gitkeep` placeholders).
*   `.github/workflows/update-dependencies.yml` — manual `workflow_dispatch` job: runs
    `npm-check-updates`, verifies the Pi manifest paths resolve, and opens a PR. Nothing is
    published or tagged.
*   `.pi/npm/` — private local extensions workspace; **not** published (`files` excludes it).
*   `README.md` — user-facing, written in Chinese; update it when the install/automation flow changes.

## COMMANDS

| Action           | Command |
|------------------|---------|
| Install          | `npm ci --allow-git=root` |
| Build            | none (nothing is compiled) |
| Test             | none (no first-party code/tests) |
| Load locally     | `pi -e .` (session only) or `pi install .` |
| Inspect manifest | `pi config` |
| Refresh deps     | Actions → Update dependencies → Run workflow (opens a PR) |

`--allow-git=root` is required: npm 12 defaults `allow-git` to `none`, and this
preset has 5 root git dependencies (`@dietrichgebert/ponytail`, `@joelhooks/pi-until`,
`pi-autoresearch`, `pi-fff-non-ascii-guard`, `sol-pi`).

## CODING STANDARDS

*   **Language**: ESM only, `node:`-prefixed built-in imports.
*   **Style**: double quotes, 2-space indent, semicolons, trailing commas; arrow functions and `const`.
*   No linter or formatter is configured — match surrounding style.
*   **Comments** explain *why* (security, ordering, escaping), not *what*. All code, comments,
    identifiers, and commit messages in English.
*   **Commits**: Conventional Commits (`feat(scope): ...`, `fix(ci): ...`, `chore(deps): ...`).

## WHERE TO LOOK

*   **Manifest / dependencies**: `package.json`
*   **Automation docs**: `README.md` section "依赖更新"
*   **CI**: `.github/workflows/update-dependencies.yml`

## NOTES

*   **This preset is distributed by git, never published to npm.** Do not add `npm publish`,
    OIDC/Trusted-Publisher, version tagging, or release-note steps. Do not add `NPM_TOKEN`
    or other publishing secrets.
*   Dependency refresh is manual only: `workflow_dispatch` runs `npm-check-updates` on direct
    deps, applies the lockfile update, verifies the `pi` manifest paths resolve, and opens a PR
    against the base branch. Requires node >= 24 and npm >= 12 in CI (workflow upgrades npm),
    and `contents: write` + `pull-requests: write` permissions.
*   Don't hand-edit `package-lock.json` versions — run `npm install`.
*   Bundle a new Pi package by adding it to `dependencies` **and** `bundledDependencies`, then
    pointing `pi.extensions` / `pi.skills` at its `node_modules/...` path. Pi core packages
    (`@earendil-works/pi-*`, `typebox`) belong in `peerDependencies` as `"*"` and stay unbundled.
*   `.npmrc` sets `force=true` because bundled Pi packages routinely pin **stale** peer ranges on
    Pi core packages the runtime provides (e.g. several packages require `@earendil-works/pi-tui`
    `>=0.74.0 <0.80.0` while this preset tracks 0.85.x). `force` tolerates the conflict and still
    installs the peer tree. Do **not** switch to `legacy-peer-deps`: it drops the entire
    `@earendil-works/*` peer subtree from the lock.
*   Repo uses a `.gitlock` file as a commit lock: create it before committing, delete it after; if
    it already exists, wait 30s and re-check.
*   Global user rules live in `~/.claude/CLAUDE.md` (Chinese replies, English code/docs, `gh` CLI
    over web UI for GitHub operations).

## SELF-CHECK

Before claiming the preset works locally: `npm ci --allow-git=root` must install cleanly and
`pi -e .` must load all bundled resources without errors.
