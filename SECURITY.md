# Security Policy

## Scope

This repository contains documentation and an agent prompt, not executable software. Its security boundary is the local modification the prompt asks DeepSeek Harness to perform on the Windows desktop application's `resources/app.asar`.

## Security properties

The canonical prompt requires:

- no patch-specific network downloads, requests, or global package installation; normal Harness model transport is unchanged;
- no access to API keys, tokens, or session content;
- read-only validation of every entry's integrity hash/blocks, plus a no-op ASAR round trip only after the state check selects the modification branch;
- inspection of Electron embedded-ASAR integrity, fuse, and signature policy, with refusal when a modified archive cannot be proven acceptable;
- a collision-safe full backup whose SHA-256 matches the source;
- local `lib/main.js` changes confined to the official tray-menu and same-instance restart-options regions, touching only whichever region is missing or wrong, with no unrelated formatting change;
- staged comparison of all archive content and semantic metadata, with only the target content and required offset/integrity updates allowed to differ;
- both-locale full-label, placement, call, and callback idempotency; an already-installed archive receives zero file writes and creates no temporary artifacts;
- live-target and backup hash checks immediately before applying;
- version-bound rollback executed manually outside DSH, with a second target-hash check immediately before atomic replacement.

## Supported versions

DSH `0.2.0-rc.2` is a structural observation baseline: its official tray anchors, localization resources, and ASAR integrity metadata were inspected. This is not an end-to-end authoring guarantee. Every version must independently pass the canonical prompt's read-only integrity, unique-anchor, and Electron runtime-policy gates; a modification branch must additionally pass its ASAR-writer/no-op-round-trip gate. A safe refusal is expected behavior, not a vulnerability.

## Reporting a security issue

Use GitHub's private security advisory feature for the repository when available. Include:

- DSH version and platform;
- whether the issue affects inspection, backup, staged rebuild, apply, or rollback;
- the relevant prompt step;
- redacted evidence sufficient to reproduce the problem.

Do not include application binaries, credentials, private session data, full user profiles, or unrelated system logs.

## Out of scope

- vulnerabilities in DeepSeek Harness or Electron themselves;
- problems caused by bypassing the prompt's stop conditions;
- restoring a backup whose manifest/hash gate does not match;
- third-party scripts or plugin forks not shipped by this repository.
