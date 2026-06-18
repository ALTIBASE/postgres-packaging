# Repository Guidelines

## Project Structure & Module Organization

This repository contains packaging scripts and metadata for Percona PostgreSQL-related components. Package-specific content lives in directories such as `common/pgbouncer/`, `common/pgbadger/`, `common/h3/`, `pgadmin4/`, `proj95/`, `gdal311/`, and `gdal385/`. Shared shell logic and version constants are in `common/common-functions.sh`, `common/install-deps.sh`, and `common/versions.sh`. RPM packaging uses `*.spec` files, Debian packaging uses files such as `control`, `rules`, `copyright`, and `compat`, and CI entry points are under `.github/workflows/`.

## Build, Test, and Development Commands

There is no single top-level build command. Package builders are run with a separate build directory and commonly require root or containerized package build environments.

Example local source/build flow, matching the CI layout:

```bash
mkdir -p /tmp/pkg-build /tmp/BUILD
cp common/common-functions.sh common/install-deps.sh common/versions.sh common/h3/h3_builder.sh /tmp/pkg-build/
cd /tmp/pkg-build
sudo -E bash ./h3_builder.sh --builddir=/tmp/BUILD --get_sources=1 --build_src_rpm=1 --build_rpm=1 --pg_major=18
```

Use `.github/workflows/ppg-packaging.yml`, `ppg-common-packages.yml`, and related workflows as the canonical CI build references.

## Coding Style & Naming Conventions

Shell scripts use Bash with `#!/usr/bin/env bash`; most builders enable `set -ex`. Follow the indentation style in the file you edit, use spaces in shell scripts, and keep tabs only where required by make-based Debian `rules` files. Use uppercase global variables such as `WORKDIR`, `PG_MAJOR`, and component-specific version variables, and lowercase function names such as `get_sources` or `build_rpm`. Name package builders consistently, for example `pgbouncer_builder.sh` or `builder.sh`. Keep RPM spec macros and Debian metadata style consistent with adjacent packages.

## Testing Guidelines

Validation is packaging-oriented. For changed shell scripts, run `bash -n path/to/script.sh` before opening a PR. For RPM changes, build the SRPM/RPM in the target container and run `rpmlint` when available. For Debian changes, build the source/binary package and run `lintian` when available. Prefer validating at least one RPM and one Debian target when touching shared files in `common/`.

## Commit & Pull Request Guidelines

Recent commits use short, imperative summaries, sometimes prefixed with Jira IDs, for example `PG-2429 Bump h3 version 4.5.0` or `update h3 spec file with upstream`. Keep the subject focused on the package and behavior changed. Pull requests should include the affected packages, target PostgreSQL major versions or distributions, validation commands/results, and linked Jira or issue references when applicable.

## Security & Configuration Tips

Do not commit generated build artifacts such as `source_tarball/`, `srpm/`, `rpm/`, or downloaded upstream source trees. Keep repository URLs and release values centralized in `common/versions.sh` unless a package has a documented exception.
