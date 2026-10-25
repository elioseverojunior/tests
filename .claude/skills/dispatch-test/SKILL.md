---
name: dispatch-test
description: Dispatch a docker-osx test workflow on GitHub Actions, watch it, and summarize per-macOS-version results. Use when the user wants to run the macOS image tests remotely.
disable-model-invocation: true
---

# Dispatch test

Run a test workflow for this repo and report the outcome. `$ARGUMENTS` may name
the workflow file (default `tests.yml`) and/or a branch (default: the current
branch).

1. Run `actionlint` on the target workflow; stop and report if it fails.
2. Make sure the branch is pushed (`git status -sb`). If there are unpushed
   commits or uncommitted workflow changes, ask before committing or pushing.
3. `workflow_dispatch` only works when the workflow file exists on the default
   branch. If `gh workflow view <file>` fails, tell the user the file must be
   merged to `main` first instead of attempting workarounds.
4. Dispatch: `gh workflow run <file> --ref <branch>`, then find the run with
   `gh run list --workflow <file> --branch <branch> --limit 1 --json databaseId`.
5. `gh run watch <id> --exit-status`. Runs are long (BaseSystem download plus
   boot); run the watch in the background rather than polling.
6. Summarize with `gh run view <id> --json jobs`: one line per matrix job
   (macOS shortname -> conclusion and duration). Count jobs that exist only via
   `include` entries — they run even when commented out of `shortname`.
7. For each failed job, show the tail of `gh run view <id> --log-failed`,
   focusing on the container-logs step (`docker logs` of the `macos` service:
   QEMU / Launch.sh output), and give a
   one-line likely cause (health check timed out, KVM missing, plist/CPU flags).
