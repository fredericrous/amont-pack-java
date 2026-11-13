---
canonical: greenharness/greengate:docs/plans/2026-11-13-greengates-last-amont-names.md
phases: [2]
status: active
---
# greengate's last amont names, and the packs renamed

The canonical plan lives in the greengate repository. This file is a pointer.

## This repository's slice

Phase 2 of the plan renames this pack: the pack comments and the README name
greengate (the commands `greengate add`, `greengate trust` and `greengate list`,
and the markers `greengate:pack:start` / `greengate:pack:end`). The pack rows
are unchanged. The repository moves to `greengate-pack-java` on GitHub, and the
tag `v1.1.0` carries the change; the `v1` tag is never moved.

## Code review

- amont-pack-java: none — markdown and pack comment text only
