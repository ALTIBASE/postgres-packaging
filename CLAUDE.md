# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repo contains **packaging scripts and metadata** for Percona's PostgreSQL distribution (PPG) and related components. It produces RPM and Debian packages; it is not an application codebase. See `AGENTS.md` for contributor conventions (commit style, lint expectations, security notes).

## Architecture

Each component lives in its own directory (`common/h3/`, `common/pgbouncer/`, `common/pgbadger/`, `common/etcd/`, `common/pysyncobj/`, `common/ydiff/`, `pgadmin4/`, `proj95/`, `gdal311/`, `gdal385/`). Components under `common/` are server-independent; top-level dirs are PostgreSQL-version-specific extensions.

The standard builder pattern (e.g. `common/h3/h3_builder.sh`) is:

1. `source versions.sh "<component>"` — pulls in version constants and the URL block for that component (the `case "$1"` switch in `common/versions.sh`).
2. `source common-functions.sh` — shared helpers: `parse_arguments`, `check_workdir`, `get_system` (detects RHEL vs Debian, sets `OS`=`rpm`/`deb`, `OS_NAME`, `ARCH`), repo setup (`add_percona_yum_repo`/`add_percona_apt_repo`), `get_tar`.
3. Builder defines `get_sources`, `build_srpm`, `build_rpm` (and `build_source_deb`/`build_deb` where supported), then runs them at the bottom gated by flags.

The build flow is staged and each stage is **toggled by a CLI flag** parsed into globals (`SOURCE`, `SRPM`, `RPM`, `SDEB`, `DEB`, `INSTALL`, `WORKDIR`, `PG_MAJOR`, `REPO_COMP`, `NIGHTLY`). A stage set to `0` short-circuits and returns. Artifacts are copied to **both** `$WORKDIR` and `$CURDIR` (under `source_tarball/`, `srpm/`, `rpm/`, `deb/`) so later stages — or a later CI job on a different runner — can find them via `get_tar` and `find ... | sort | tail -n1`.

`common/install-deps.sh` (sourced only when `--install_deps=1`) centralizes per-distro, per-component build dependencies. It branches heavily on `$COMPONENT` and `$RHEL` version — when adding a component, add its deps here.

`common/versions.sh` is the **single source of truth** for versions, upstream source repos, and `*_RELEASE` numbers. Bump versions here, not in builders or specs. `PG_VERSION=@@PGVERSION@@` is a template placeholder substituted by CI.

`proj95/` and `gdal*/` use a different, simpler builder interface (`--get_src_rpm`, `--build_rpm`, `--branch`, `--repo`) — they rebuild existing PGDG/upstream SRPMs rather than building from a git clone.

Special-purpose scripts: `pg_tarballs/` (assembles PostgreSQL source tarballs), `pg_sbom/` (generates SBOMs, `--pg_version`/`--repo_type`), `pg_snyk_scan/` (Snyk vulnerability scans).

`.github/workflows/` are the canonical build orchestration. Each `workflow_dispatch` builds a matrix of components × PG major versions × distros. `ppg-common-packages.yml` builds the server-independent components; `ppg-postgres-packaging.yml` builds PostgreSQL itself (14–18); `ppg-packaging*.yml` build PPG packages/extras; `ppg-testing.yml` and `ppg-docker-testing.yml` validate.

## Building locally

There is no top-level build command. Builders need a separate build dir and typically root/containerized RPM or Debian build environments. Replicate the CI layout by copying the shared scripts + the component builder into a working dir, then invoking the builder:

```bash
mkdir -p /tmp/pkg-build /tmp/BUILD
cp common/common-functions.sh common/install-deps.sh common/versions.sh common/h3/h3_builder.sh /tmp/pkg-build/
cd /tmp/pkg-build
sudo -E bash ./h3_builder.sh --builddir=/tmp/BUILD --get_sources=1 --build_src_rpm=1 --build_rpm=1 --pg_major=18
```

The builders `source` the shared scripts by bare filename, so they must sit in the same directory at runtime (this is why CI copies them flat).

## Validation

- Shell changes: `bash -n path/to/script.sh` before committing.
- RPM changes: build the SRPM/RPM in the target container; run `rpmlint` when available.
- Debian changes: build source/binary package; run `lintian` when available.
- When touching `common/` shared files, validate at least one RPM and one Debian target.

## Conventions

- Do not commit generated artifacts: `source_tarball/`, `srpm/`, `rpm/`, `deb/`, or downloaded upstream trees.
- Keep version/URL/release values in `common/versions.sh`.
- Builders use `#!/usr/bin/env bash` with `set -ex`; globals UPPERCASE, functions lowercase (`get_sources`, `build_rpm`).
- Large commented-out blocks (e.g. `build_deb` in `h3_builder.sh`) are intentionally retained as templates for not-yet-enabled targets — leave them unless the task is to enable that target.
