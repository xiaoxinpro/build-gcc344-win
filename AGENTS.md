# AGENTS.md

## Project Overview

This repository builds a native Windows MinGW GCC 3.4.4 toolchain through GitHub Actions and publishes the packaged toolchain to a GitHub Release.

The main workflow is:

- `.github/workflows/build-gcc344.yml`

The build target is `i686-pc-mingw32`. The workflow uses an Ubuntu GitHub Actions runner, starts a Debian 8 container, builds the required cross toolchain components, then performs a Canadian Cross build to produce Windows-native GCC binaries.

## Toolchain Components

The workflow currently defines these versions:

- GCC: `3.4.4`
- Binutils: `2.16.1`
- w32api: `3.11`
- mingw-runtime: `3.14`
- Release tag: `gcc-3.4.4-mingw32`

## Build Flow

The workflow performs these major stages:

1. Prepare a writable `work/` directory inside the GitHub Actions workspace.
2. Run all build steps inside a `debian:8` Docker container.
3. Configure archived Debian Jessie package sources for legacy build dependencies.
4. Download and validate GCC, binutils, w32api, and mingw-runtime source archives.
5. Build Linux-hosted cross binutils for `i686-pc-mingw32`.
6. Install MinGW headers and runtime libraries into the cross prefix.
7. Build the cross GCC bootstrap compiler and `libgcc`.
8. Build Windows-native binutils.
9. Build Windows-native GCC.
10. Copy runtime headers and libraries into the final native prefix.
11. Package the result as `gcc-3.4.4-i686-pc-mingw32-native-windows.tar.xz`.
12. Upload the package as a workflow artifact and publish it to the configured GitHub Release.

## Important Paths

- `.github/workflows/build-gcc344.yml`: authoritative build and release automation.
- `README.md`: short public project description.
- `logs/`: local-only directory for workflow logs downloaded by the user.
- `.gitignore`: ignores `logs/` so downloaded workflow logs are not committed.

## Log Handling

The `logs/` directory is intended for downloaded GitHub Actions logs. Agents may read those logs when diagnosing workflow failures. After completing workflow changes, agents may delete files inside `logs/` because they are transient diagnostic artifacts.

Do not commit files from `logs/`.

## Encoding Rules

All files created or modified in this repository must be written as UTF-8. Do not rely on the operating system default encoding.

When editing from PowerShell, explicitly use UTF-8-aware commands or tools. For example:

```powershell
Get-Content -Raw -Encoding UTF8 path\to\file
Set-Content -Encoding UTF8 path\to\file -Value $content
```

When modifying files as an AI agent, prefer patch-based edits and verify the resulting content can be read as UTF-8.

## Language Conventions

This file must remain in English.

For other repository files, comments added by agents should be written in Chinese when practical. Existing English identifiers, command output, tool names, YAML keys, release text, and upstream package names should remain unchanged unless there is a specific reason to edit them.

## Workflow Notes

- Keep the workflow self-contained; GitHub Actions should be able to rebuild from source archives without relying on local files.
- Preserve the explicit legacy Debian archive configuration because Debian 8 is end-of-life.
- Preserve failure log upload behavior so build errors can be downloaded and inspected later.
- Keep release publishing tied to the generated `gcc-*.tar.xz` package.
- Be careful when changing old GCC/binutils configure flags; these projects predate many modern defaults and can fail with newer assumptions.

## Git Notes

Before committing, inspect `git status --short --branch` and avoid reverting user changes. If unrelated user edits are present, leave them untouched and commit only the files required for the current task.
