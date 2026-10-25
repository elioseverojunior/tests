# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Purpose

Tests `sickcodes/docker-osx` images on GitHub-hosted `ubuntu-latest` runners.
The image runs as a job `services:` container with `--device /dev/kvm`; QEMU
forwards the guest's SSH (10022) to host port 50922. Docker-OSX publishes only
the `latest` tag: the macOS version is picked at runtime by `SHORTNAME`, and the
BaseSystem is downloaded on boot (hence the long health-check retry window).

## Workflows

- `.github/workflows/tests.yml` is a scratch copy for manual experiments
  (`workflow_dispatch` only). Changes that prove out are ported to `ci.yml`,
  which runs the full matrix on push to `main`.
- `workflow_dispatch` only works once the workflow file exists on the default
  branch; dispatch other branches with `gh workflow run <file> --ref <branch>`.
- Matrix gotcha: commenting a version out of `matrix.shortname` does NOT
  disable it while an `include` entry with that `shortname` remains — an
  `include` that cannot merge into an existing combination creates a new job.
  Remove or comment out the matching `include` entry too.
- Empty `CPU` / `CPUID_FLAGS` fall back to the defaults in Docker-OSX's
  `Launch.sh`; sonoma and newer need `Haswell-noTSX` and the sonoma plist.
- `latest` + `SHORTNAME` boots only the installer: no user, no sshd. The only
  pre-installed image is `auto` (Catalina, `user`/`alpine`); newer versions
  need a self-installed disk via `naked-auto`.
- The goal is to SSH into the guest and run commands (`sw_vers`, Homebrew).
  `nc -zv localhost 50922` passes even with no guest sshd (QEMU's hostfwd
  accepts on the host side), so readiness means a successful `ssh ... true`.
- `tests.yml` starts the image with `docker run` in a step, not `services:`,
  so runner disk is freed before the multi-GB `auto` pull.
- Validate workflow edits with `actionlint` (runs shellcheck on `run:` blocks).

## Conventions

- Pin `uses:` to the loosest published tag (`actions/checkout@v7`), not a SHA.
  Do NOT run `scripts/update-actions-pins.py`: it rewrites refs to SHA pins,
  which this repo no longer uses. When touching a workflow, convert existing
  SHA pins to tags.
- Write comments, step names and commit messages in English (the existing
  Portuguese ones are legacy; translate when touching them).
