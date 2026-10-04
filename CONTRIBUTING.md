# Contributing

Thanks for your interest in improving Setup Git SemVer Release. This guide is for anyone changing this repository — human contributors and AI coding agents alike. `AGENTS.md` is a symlink to this file.

For what the action does and how to use it (inputs, outputs, configuration, examples), see [README.md](README.md).

## Reporting issues

- Search [existing issues](https://github.com/michalstutzmann/setup-git-semver-release/issues) before opening a new one.
- For bugs, include the workflow snippet using the action, the action version (e.g. `@v2.0.1`), the `version` input if set, and the relevant job log output.
- Problems with version calculation or tagging itself usually belong to [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release/issues); this repo only installs and runs that tool.

## Submitting changes

1. Fork the repository and create a branch from `main`.
2. Make your change, keeping it small and focused.
3. Run the [validation checks](#validation) relevant to the files you touched.
4. Update [README.md](README.md) if the change affects inputs, outputs, or documented behavior.
5. Open a pull request against `main` describing what changed and why.

## Repository layout

| Path                            | Purpose                                                                                                                                                         |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `action.yml`                    | Action metadata and the composite action implementation: downloads the `git-semver-release` script, adds it to `PATH`, runs it, and writes the `version` output. |
| `publish`                       | Bash helper that creates a GitHub release for the current release tag (see below). Used by this repo's own release workflow; not part of the action.            |
| `.github/workflows/publish.yml` | Runs `publish` on every tag push to create the matching GitHub release.                                                                                          |
| `README.md`                     | User-facing documentation and examples.                                                                                                                          |
| `CONTRIBUTING.md`               | This file. `AGENTS.md` symlinks to it.                                                                                                                           |

## Guidelines

- Keep changes small and focused.
- This repo wraps the behavior implemented in [`michalstutzmann/git-semver-release`](https://github.com/michalstutzmann/git-semver-release); do not document behavior here that this action does not actually expose.
- Keep `action.yml` inputs and outputs in sync with the Inputs and Outputs tables and the examples in [README.md](README.md).
- When changing the default installed `git-semver-release` version, also update the `version` input default in the README, the pinned version in `.github/workflows/publish.yml`, and any documentation that depends on the tool's behavior.
- Shell: Bash is fine (the existing scripts use it); prefer POSIX-compatible patterns where practical, quote variables, and preserve `set -euo pipefail` where it is already used.

## Validation

Before finishing a change, run the checks relevant to the files you touched:

- Shell changes: `bash -n publish`.
- `action.yml` changes: confirm it is valid YAML and that the README inputs, outputs, and examples still match it.

## Release process

Releases of this action are cut by maintainers by pushing a SemVer tag (e.g. `v2.0.1`). The `Publish` workflow then installs `git-semver-release` and runs `./publish`, which:

1. Requires `gh` to be installed and authenticated (`GH_TOKEN`).
2. Reads the version (`git-semver-release version`) and the release tag of `HEAD` (`git-semver-release release-tag`).
3. If `HEAD` is detached and points at a release tag, creates a GitHub release titled with the version and using the tag annotation as notes. An existing release for the tag is left untouched.

Exit codes: `1` — `gh` missing, `2` — `gh` not authenticated, `3` — version lookup failed, `4` — release-tag lookup failed, `5` — release creation failed.

After releasing, update the version pinned in the README examples (e.g. `@v2.0.1`) to the new tag.
