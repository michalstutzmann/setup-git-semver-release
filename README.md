# Setup Git SemVer Release

GitHub Action wrapper around [`git-semver-release`](https://github.com/michalstutzmann/git-semver-release) — a single Bash script that derives Semantic Versioning releases from Git history.

The action downloads the `git-semver-release` script from the [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release/releases) GitHub releases, adds it to `PATH`, and runs it against your repository checkout.

## Quick start

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    fetch-tags: true
    ref: ${{ github.ref }}

- id: semver
  uses: michalstutzmann/setup-git-semver-release@v2.0.1

- run: echo "Version is ${{ steps.semver.outputs.version }}"
```

The `actions/checkout` settings matter:

- **`fetch-depth: 0`** is required — the tool derives the version from prior tags and full commit history (`git describe`, `git rev-list`), so the default shallow clone would break version calculation.
- **`fetch-tags: true`** ensures release tags are present so the tool can find the latest release. With `fetch-depth: 0` tags are already fetched, so this is belt-and-suspenders; keep it if you ever lower the fetch depth.
- **`ref: ${{ github.ref }}`** checks out the branch rather than a detached `HEAD`. This is required when `push: true`, because the tool runs `git push origin HEAD` — from a detached `HEAD` that has no branch to push to and fails. It is harmless for read-only `version` use.

## Inputs

| Name      | Default                                       | Description                                                                                                  |
| --------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `command` | `version`                                     | Subcommand to run: `version`, `major`, `minor`, `patch`, `conventional`, or `release-tag`.                   |
| `push`    | `false`                                       | Push the created tag to `origin`. Only applies to `major`, `minor`, `patch`, and `conventional`.             |
| `channel` | `''`                                          | Pre-release channel suffix to append to the tag (e.g. `alpha`, `beta`, `rc`).                                |
| `message` | `''`                                          | Annotation message for the created tag. Supports the `$version` placeholder.                                 |
| `version` | `v2.0.1`                                      | Release tag of [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release/releases) to install (e.g. `v2.0.1`), or `latest`. This selects which build of the tool to download — it is unrelated to the `version` command and the `version` output. |

## Outputs

| Name      | Description                                                                                                                                                                                                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version` | The calculated version, without the tag prefix. On a clean release-tagged commit this is a plain `1.2.3`; if the working tree is dirty on a release-tagged commit, the patch level is bumped and the dirty indicator (default `dirty`) is appended via `$dirty_indicator`; on commits after a tag it carries the pre-release suffix, e.g. `1.2.4-main.5.ab12cd3` (channel — the current branch by default — number of commits since the tag, and short SHA, per the default `pre_release_format`). |

## Configuration

`git-semver-release` reads an optional `.git-semver-release.properties` file from the repository root. Because the action runs the tool inside your checkout, this file is respected — it is the way to customize the tag prefix and the pre-release version format. Recognized keys:

| Key                  | Default                                                                                 | Description                                                                                                                                                  |
| -------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `tag_prefix`         | `v`                                                                                     | Prefix for created and matched tags (e.g. `v1.2.3`). Set to empty for unprefixed tags.                                                                        |
| `channel`            | _current branch_                                                                        | Default pre-release channel. When neither the `channel` input nor this key is set, the current Git branch name is used.                                        |
| `dirty_indicator`    | `dirty`                                                                                 | Token substituted for `$dirty_indicator` when the working tree has uncommitted changes.                                                                       |
| `pre_release_format` | `$channel$separator$commit_count$separator$commit_short_sha$separator$dirty_indicator` | Template for the pre-release suffix. Placeholders: `$channel`, `$branch`, `$commit_count`, `$commit_short_sha`, `$dirty_indicator`, and `$separator` (`.`).    |

Example `.git-semver-release.properties`:

```properties
tag_prefix=
channel=beta
pre_release_format=$channel$separator$commit_short_sha
```

## Examples

### Print the current version

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
    fetch-tags: true
    ref: ${{ github.ref }}
- id: semver
  uses: michalstutzmann/setup-git-semver-release@v2.0.1
- run: echo "${{ steps.semver.outputs.version }}"
```

### Release on push to `main` from Conventional Commits

```yaml
on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          fetch-tags: true
          ref: ${{ github.ref }}
      - uses: michalstutzmann/setup-git-semver-release@v2.0.1
        with:
          command: conventional
          push: 'true'
```

`permissions: contents: write` lets the workflow's `GITHUB_TOKEN` push tags back to the repo.

> **Note:** `conventional` exits non-zero — failing the step — when there are no releasable commits since the last release (no `feat:`, `fix:`, `perf:`, or breaking-change commits). On a branch where most pushes have nothing to release, add `continue-on-error: true` to the step or gate it on the commit history so the workflow does not fail on every no-op push.

### Bump a specific level with a custom annotation

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2.0.1
  with:
    command: minor
    message: 'Release $version'
    push: 'true'
```

### Pre-release channel

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2.0.1
  with:
    command: patch
    channel: rc
    push: 'true'
```

Produces a tag like `v1.2.4-rc`.

### Pin to a specific `git-semver-release` version

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2.0.1
  with:
    version: 'v2.0.1'
```

## Development

This section is for anyone changing this repository — human contributors and AI coding agents alike. `AGENTS.md` is a symlink to this file.

### Repository layout

| Path                            | Purpose                                                                                                                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `action.yml`                    | Action metadata and the composite action implementation: downloads the `git-semver-release` script, adds it to `PATH`, runs it, and writes the `version` output. |
| `publish`                       | Bash helper that creates a GitHub release for the current release tag (see below). Used by this repo's own release workflow; not part of the action.          |
| `.github/workflows/publish.yml` | Runs `publish` on every tag push to create the matching GitHub release.                                                                                        |

### Release process

Releases of this action are cut by pushing a SemVer tag (e.g. `v2.0.1`). The `Publish` workflow then installs `git-semver-release` and runs `./publish`, which:

1. Requires `gh` to be installed and authenticated (`GH_TOKEN`).
2. Reads the version (`git-semver-release version`) and the release tag of `HEAD` (`git-semver-release release-tag`).
3. If `HEAD` is detached and points at a release tag, creates a GitHub release titled with the version and using the tag annotation as notes. An existing release for the tag is left untouched.

Exit codes: `1` — `gh` missing, `2` — `gh` not authenticated, `3` — version lookup failed, `4` — release-tag lookup failed, `5` — release creation failed.

### Guidelines

- Keep changes small and focused.
- This repo wraps the behavior implemented in [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release); do not document behavior here that this action does not actually expose.
- Keep `action.yml` inputs and outputs in sync with the [Inputs](#inputs) and [Outputs](#outputs) tables and the examples above.
- When changing the default installed `git-semver-release` version, also update the `version` input default in the README, the pinned version in `.github/workflows/publish.yml`, and any documentation that depends on the tool's behavior.
- Shell: Bash is fine (the existing scripts use it); prefer POSIX-compatible patterns where practical, quote variables, and preserve `set -euo pipefail` where it is already used.

### Validation

Before finishing a change, run the checks relevant to the files you touched:

- Shell changes: `bash -n publish`.
- `action.yml` changes: confirm it is valid YAML and that the README inputs, outputs, and examples still match it.
