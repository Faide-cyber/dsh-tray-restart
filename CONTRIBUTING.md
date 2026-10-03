# Contributing

Thank you for improving `dsh-tray-restart`.

## Project boundary

This is a prompt-only project. Contributions must preserve:

- one official DSH tray icon;
- no Cordis plugin or helper process;
- no runtime package dependency;
- no patch-specific network download or installation step;
- read-only per-entry integrity and Electron runtime-policy gates for every branch, plus a no-op ASAR round-trip gate only when modification is needed;
- collision-safe backup-before-write and externally executable hash-gated rollback;
- structural, full-label idempotent detection that evaluates both locale tables instead of using fixed line numbers;
- source changes limited to the official tray-menu region and the same tray instance's restart-options region, touching only whichever region is missing or wrong.

A contribution that reintroduces a second tray, background script, service, registry entry, scheduled task, startup item, or profile bundle is out of scope.

## Documentation ownership

The complete install prompt in `README.md` is the single source of truth. Supporting documents may explain or test it, but should not carry a second divergent installation recipe. `README.en.md` intentionally points to the canonical prompt rather than duplicating it.

## Change checklist

For prompt changes, describe evidence for all applicable cases:

- [ ] fresh DSH archive that passes ASAR-authoring and Electron runtime-policy gates;
- [ ] already-patched archive with both complete locale labels, zero file writes/rebuilds, and no temporary artifacts;
- [ ] partially patched or incomplete-label archive;
- [ ] missing or duplicate anchors (safe refusal);
- [ ] modification branch: no-op ASAR round-trip and every entry's integrity hash/blocks;
- [ ] unrelated pre-existing source changes preserved by a line-level diff confined to the two allowed semantic regions, with only the actually missing/wrong region(s) changed;
- [ ] full ASAR content and semantic metadata differ only where the canonical prompt permits;
- [ ] current archive or backup changes after baseline (safe refusal);
- [ ] DSH update causes old backup hash mismatch (safe refusal, no similarity-based deletion);
- [ ] rollback command remains executable outside DSH and rechecks target hash immediately before replacement;
- [ ] rollback decision follows `docs/ROLLBACK.md`.

For documentation changes:

- keep links relative where possible;
- keep Windows paths clearly marked as examples;
- distinguish verified facts from compatibility expectations;
- update `CHANGELOG.md` for user-visible behavior changes;
- avoid claims that DSH officially supports this patch.

## Pull requests

A pull request should state:

1. the problem being solved;
2. the exact prompt or documentation behavior changed;
3. the DSH version(s) inspected;
4. the evidence used for safe apply and safe refusal paths;
5. whether rollback or update semantics changed.

Never include API keys, tokens, session logs, user profiles, or proprietary application binaries in an issue or pull request.
