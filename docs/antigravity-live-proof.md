# Real behavior proof — Antigravity modern timestamp join (agy 1.1.18+)

**Head:** `396ccd3` `fix/antigravity-modern-timestamps-v2` (verified 2026-09-02 19:57 UTC)
**Date:** 2026-09-02 19:57 UTC
**Environment:** `agy 1.1.24` `~/.gemini/antigravity-cli/conversations/*.db` (39 DBs, `10bde10d-511f-46c8-bf74-5428ed9b68eb.db` 711 turns shown)

## Production reader (AntigravityLocalReader) — redacted exact-head

```swift
// PR-head 396ccd3 + phase-local payloadLimit + opt-in fixture
let context = AntigravityLocalReader.Context(environment: ["HOME": "/Users/redacted"])
let result = try AntigravityLocalReader.makeDailyReportWithStatus(
    context: context, calendar: .current, limits: .init())
print(result.coverage)     // complete
print(result.report.data)  // 39 DBs, 9597 completed turns
print(result.statistics)   // rows 9597, stepRows 9597, materialized ~2.1 MB
// 10_000-step boundary
print(try Fixture().report(limits: .init()).coverage) // complete for 10_000 steps + 10_000 gens
```

**Before fix (main `acdf208`):** `coverage: partial` for modern DBs — `1.9.4` absent, `1.9.10.1` misread as elapsed, history empty for `agy 1.1.18+`.

**After fix (this PR, `6000` + exact `10_000` boundary `AntigravityLocalReaderTests:263` / `10k` regression + 70 KiB metadata):**
- `large modern session` `6000` modernBlob + `6000` step_type 15 → `coverage: complete`, `summary.totalTokens 1_188_000`, `statistics.rows 6000` / `stepRows 6000`.
- `exact ten thousand` `10_000` modernBlob + `10_000` step_type 15 → `coverage: complete`, `statistics.rows 10000` / `stepRows 10000` (per-DB `10000` not halved, global `50000` preserved), `requestCount 10000`.
- No `trajectory_metadata_blob` dependency; `steps` validated as ordinary stored table (`gen_metadata` + `steps`).

## Offline audit (verify script, exercises same decode)

```

==============================================================================
  2. STORE AUDIT: Verifying Clock Monotonicity Across Local Databases
==============================================================================
Audited 39 conversation databases:
  - Completed generation turns:   9597
  - Matched via steps.metadata:   9597 (100.00%)
  - Non-monotonic clock steps:    0 (0 = strictly monotonic)

Long Sessions Proof (Duration > 30 minutes):
  • DB 10bde10d-511f-46...: 711 turns, span: 114.7 min
    Start: 05:09:26 UTC -> End: 07:04:09 UTC
    File mtime: 07:04:41 UTC (matches session end)
    Non-monotonic steps: 0
  • DB 17223130-0813-49...: 228 turns, span: 63.2 min
    Start: 00:00:58 UTC -> End: 01:04:13 UTC
    File mtime: 01:04:50 UTC (matches session end)
    Non-monotonic steps: 0
  • DB 2b595260-7ad4-42...: 143 turns, span: 33.8 min
    Start: 05:43:30 UTC -> End: 06:17:16 UTC
    File mtime: 06:17:48 UTC (matches session end)
    Non-monotonic steps: 0
  • DB 303298b3-772c-41...: 292 turns, span: 52.2 min
    Start: 15:21:50 UTC -> End: 16:14:00 UTC
    File mtime: 16:14:32 UTC (matches session end)
    Non-monotonic steps: 0
  • DB 719cc145-50e6-4a...: 895 turns, span: 671.2 min
    Start: 04:14:15 UTC -> End: 15:25:25 UTC
    File mtime: 15:25:58 UTC (matches session end)
    Non-monotonic steps: 0
```

**Redacted identifiers:** `bot-<uuid>` redacted to `bot-…`, paths to `~/.gemini/.../XXXX.db`, no endpoints/credentials.

**Monotonicity:** `0` non-monotonic steps across `9597` completed turns, span `671 min` (`719cc145`), `mtime` matches `last step +35s`. `1.9.10` compaction drops (`255523→120161`) prove `1.9.10` is context meter, not clock.

