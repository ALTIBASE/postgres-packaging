# Repository Guidelines

## Project Structure & Module Organization

This repository contains packaging scripts and metadata for PostgreSQL-related components. Top-level component directories such as `pgadmin4/`, `proj95/`, `gdal311/`, `gdal385/`, `pg_sbom/`, `pg_snyk_scan/`, and `pg_tarballs/` hold builder scripts and package-specific assets. Shared package definitions live under `common/`, with subdirectories such as `common/pgbouncer/`, `common/pgbadger/`, `common/etcd/`, `common/ydiff/`, and `common/h3/`. RPM packaging uses `*.spec` files, while Debian packaging uses files such as `control`, `rules`, `copyright`, `changelog`, and `debian/source/format`. GitHub Actions workflows are in `.github/workflows/`.

## Build, Test, and Development Commands

Most work is driven by Bash builder scripts. Run commands from the repository root unless a script documents otherwise.

- `bash common/install-deps.sh <component> <pg_major>` installs distro dependencies; it usually requires root inside a suitable RPM or Debian container.
- `bash common/pgbouncer/pgbouncer_builder.sh --help` shows builder options for a common package.
- `bash pg_tarballs/pg_tarballs_builder.sh --version=18.4 --build_dependencies=1` builds PostgreSQL tarball artifacts and optional dependency packages.
- `bash pg_sbom/pg_generate_sbom.sh` generates SBOM output when required by release workflows.

Prefer disposable containers or temporary build directories such as `/tmp/BUILD`; shared helpers reject using the repository itself as a build directory.

## Coding Style & Naming Conventions

Shell scripts use Bash, `#!/usr/bin/env bash` or `#!/bin/bash`, and explicit option parsing with long flags like `--build_rpm=1`. Keep scripts POSIX-compatible only where existing code already does so. Use two-space or four-space indentation consistently within the file being edited, quote variable expansions where practical, and preserve existing package naming patterns such as `percona-<component>.spec`, `<component>_builder.sh`, and Debian `control`/`rules` files.

## Testing Guidelines

There is no standalone unit test suite in this repository. Validate changes by running the affected builder with `--help` or a minimal dry build when available, then test the relevant RPM or Debian path in a matching container. For workflow-only changes, inspect the affected `.github/workflows/*.yml` matrix and confirm shell snippets with `bash -n` where possible.

## Commit & Pull Request Guidelines

Recent history uses short imperative or ticket-prefixed subjects, for example `PG-2429 Bump h3 version 4.5.0` or `update h3 release number for Q2'26 custom images updates`. Keep commits focused on one package or workflow. Pull requests should describe the package, target PostgreSQL version, affected distros, commands run, and any expected artifact or repository changes. Link the tracking issue when one exists.

## Security & Configuration Tips

Do not commit downloaded tarballs, generated build trees, credentials, or repository tokens. Treat package-manager changes carefully: repository components such as `testing`, `experimental`, and release channels affect published artifacts.
