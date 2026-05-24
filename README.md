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
  uses: michalstutzmann/setup-git-semver-release@v2

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
| `version` | `v2.0.0`                                      | Release tag of [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release/releases) to install (e.g. `v2.0.0`), or `latest`. This selects which build of the tool to download — it is unrelated to the `version` command and the `version` output. |

## Outputs

| Name      | Description                                                                                                                                                                                                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version` | The calculated version, without the tag prefix. On a release-tagged commit this is a plain `1.2.3`; on commits after a tag it carries the pre-release suffix, e.g. `1.2.4-alpha.5.ab12cd3` (channel, number of commits since the tag, and short SHA, per the default `pre_release_format`). |

## Configuration

`git-semver-release` reads an optional `.git-semver-release.properties` file from the repository root. Because the action runs the tool inside your checkout, this file is respected — it is the way to customize the tag prefix and the pre-release version format. Recognized keys:

| Key                  | Default                                                                                 | Description                                                                                                                                                  |
| -------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `tag_prefix`         | `v`                                                                                     | Prefix for created and matched tags (e.g. `v1.2.3`). Set to empty for unprefixed tags.                                                                        |
| `channel`            | `alpha`                                                                                 | Default pre-release channel used when the `channel` input is not given.                                                                                       |
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
  uses: michalstutzmann/setup-git-semver-release@v2
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
      - uses: michalstutzmann/setup-git-semver-release@v2
        with:
          command: conventional
          push: 'true'
```

`permissions: contents: write` lets the workflow's `GITHUB_TOKEN` push tags back to the repo.

> **Note:** `conventional` exits non-zero — failing the step — when there are no releasable commits since the last release (no `feat:`, `fix:`, `perf:`, or breaking-change commits). On a branch where most pushes have nothing to release, add `continue-on-error: true` to the step or gate it on the commit history so the workflow does not fail on every no-op push.

### Bump a specific level with a custom annotation

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2
  with:
    command: minor
    message: 'Release $version'
    push: 'true'
```

### Pre-release channel

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2
  with:
    command: patch
    channel: rc
    push: 'true'
```

Produces a tag like `v1.2.4-rc`.

### Pin to a specific `git-semver-release` version

```yaml
- uses: michalstutzmann/setup-git-semver-release@v2
  with:
    version: 'v2.0.0'
```
