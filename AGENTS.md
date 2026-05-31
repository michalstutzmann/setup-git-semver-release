# Agent Instructions

## Project Overview

This repository contains a composite GitHub Action that installs and runs
`michalstutzmann/git-semver-release`.

Key files:

- `action.yml`: GitHub Action metadata and composite action implementation.
- `publish`: helper script for creating a GitHub release from the current
  release tag.
- `README.md`: user-facing documentation and examples.

## Development Guidelines

- Keep changes small and focused.
- Prefer POSIX-compatible shell patterns where practical, but `bash` is allowed;
  the existing scripts already use Bash.
- Preserve `set -euo pipefail` behavior in shell snippets that already use it.
- Quote shell variables unless there is a specific reason not to.
- Keep `action.yml` inputs, outputs, and README documentation in sync.
- Do not change the default installed `git-semver-release` version without also
  updating related documentation.

## Validation

Before finishing changes, run the most relevant checks available for the files
you touched:

- For shell changes, run `bash -n publish` when applicable.
- For action metadata changes, inspect `action.yml` for valid YAML and verify the
  README examples still match the inputs and outputs.

## Release Notes

This repo wraps the release behavior implemented in
`michalstutzmann/git-semver-release`; avoid documenting behavior here that is not
actually exposed by this action.
