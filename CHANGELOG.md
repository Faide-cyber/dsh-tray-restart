# Changelog

All notable changes to this project are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project uses [Semantic Versioning](https://semver.org/).

## [1.0.1] - 2026-10-03

### Changed

- Made the already-installed path strictly zero-file-write with no temporary artifacts, and required both complete locale labels in state/idempotency checks.
- Added a modification-only no-op ASAR round-trip, per-entry integrity, allowed-semantic-region source diff, and Electron embedded-ASAR/fuse policy gates.
- Replaced the in-session rollback fiction with a two-stage, user-executed offline flow and an immediate pre-replacement hash recheck.
- Scoped rollback-manifest uniqueness to the current live hash, DSH version, and target path, so historical manifests from earlier update cycles do not block valid recovery.
- Reclassified `0.2.0-rc.2` as a structural observation baseline rather than an end-to-end authoring guarantee.
- Clarified that the patch adds no network requests while normal Harness model transport remains unchanged.

## [1.0.0] - 2026-10-03

### Added

- Copy-ready Chinese Harness prompt for adding restart to the official DSH tray.
- Structure-aware and idempotent patch contract.
- SHA-256-verified backup manifest and staged ASAR validation requirements.
- Manual verification, negative checks, update guidance, and hash-gated rollback documentation.
- English project overview, security policy, and contribution guide.

### Removed

- Cordis plugin distribution.
- Extra WinForms/PowerShell tray helper.
- Runtime flag, log, mutex, helper lifecycle, and second tray icon.

### Security

- The prompt refuses ambiguous anchors, unverified archives, changed baselines, and cross-version backup restoration.
