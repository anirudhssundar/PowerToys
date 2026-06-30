# Efficiency Improver Memory

## Last Updated
2026-06-30 08:10 UTC (run 28429830976)

## Build/Test/Benchmark Commands
- **Build**: `tools\build\build-essentials.cmd` (first time), `tools\build\build.cmd` (incremental)
- **PowerShell**: `./tools/build/build.ps1 -Platform x64 -Configuration Release`
- **Tests**: VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe`. Avoid `dotnet test`.
- **Lint**: StyleCop.Analyzers (C#), clang-format (C++), XamlStyler (XAML)
- **Note**: Build NOT runnable in Linux CI. Sparse checkout has ~2% of files.

## Work In Progress
*(none)*

## Completed Work
- 2026-06-30 (run 28429830976): Task 4 — verified all PRs, no new human activity; Task 7 — Monthly summary updated
- 2026-06-29 (run 28361576460): Task 2 — analyzed auto-cherry-pick.ps1, Program.cs — no new findings; ALL sparse-checkout files now fully scanned; Task 4 — PRs unchanged; Task 7 — Monthly summary updated
- 2026-06-28 (run 28315942087): Task 4 — discovered PR #22 (O(n²) → O(n) PS1 array append, created 2026-06-26 by run 28225749722); no new human comments on any PR; Task 7 — Monthly summary updated
- 2026-06-26 (run 28225749722): Task 3 — PR #22 submitted (O(n²) → O(n) in diff_prs.ps1 and find-commit-by-title.ps1). Run failed with token budget exhausted after PR creation.
- 2026-06-25 (run 28155752616): Run failed with token budget exhausted before completing.
- 2026-06-24 (run 28084130360): Task 2 — analyzed remaining ImageResizer files, no new findings; Task 4 — PRs unchanged; Task 7 — Monthly summary updated
- 2026-06-23 (run 28011714250): Task 2 (F-8 upgraded to MEDIUM, deferred); Task 4 (PR check); Task 7 updated
- 2026-06-22 (run 27945993254): Task 4 (PR review), Task 2 (F-6/F-8 re-evaluation), Task 7 updated
- 2026-06-21 (run 27899109219): Task 6 — filed Issue #19 (BenchmarkDotNet); Task 7 updated
- 2026-06-20 (run 27865081378): Task 3 — PR #17 submitted (F-3); Task 4 — PR #14 closure comment
- 2026-06-19 (run 27817120053): Task 5 — Commented on #13; Task 2 — F-8 identified
- 2026-06-17 (run 27678861580): PR #16 submitted (F-1, F-2, F-4, F-5)
- 2026-06-15: PR #3 MERGED (telemetry-pr-check.js 9.12x speedup); PR #6 submitted

## Optimisation Backlog

### HIGH Priority
*(none pending)*

### MEDIUM Priority
- **Code-Level | ImageResizer F-8**: `sizeNameSanitized` = `sizeName.Replace('\\','_').Replace('/','_')` — same for all files in a batch. Pre-computing saves 2 allocs per file. DEFERRED: implement AFTER PR #16 merges (conflict avoidance — both touch ResizeBatch.cs).
- **Code-Level | ImageResizer F-6**: `GetEncoderPropertySet()` creates new `BitmapPropertySet` per JPEG; fixed per batch. Thread-safety of sharing needs investigation.
- **Code-Level | Launcher EnvironmentHelper/ResultsViewModel**: `.ToList()` copies. Source NOT in sparse checkout.
- **Code-Level | ColorPicker/AdvancedPaste UserSettings**: Thread.Sleep retry loops. Source NOT in sparse checkout.

### LOW Priority
- **Network & I/O | msstore-submissions.yml/package-submissions.yml**: 8 jq subprocess invocations; no wingetcreate caching. Protected-file paths.

## Efficiency Notes
- **Sparse checkout**: ~2% of tracked files locally accessible.
- **ALL sparse-checkout files fully analyzed** (ImageResizer Models, auto-cherry-pick.ps1, Program.cs) — no new findings beyond F-1 through F-8.
- **Protected-file restriction**: `.github/workflows/` and `.github/skills/` CANNOT be auto-pushed. Always create as issues for manual apply.
- **CI Infrastructure Issues**: `check-spelling` v0.0.26 has security advisory (fails all PRs); `dependency-review` fails (Dependency Graph not enabled). NOT caused by our code.
- **Token budget**: Keep monthly summary concise to avoid exhaustion.
- **ResizeOperation architecture**: One instance per file, not reused. F-6/F-8 require batch-level changes.

## Backlog Cursor
ALL sparse-checkout files analyzed. No new files to scan until sparse checkout is expanded or PRs merge.
Current focus:
- Task 4: Monitor PRs #6, #14, #16, #17, #22 for review/merge
- PR #14 needs closure (superseded by #16)
- F-8/F-6 blocked pending #16/#17 merge
- Issue #19 (BenchmarkDotNet) awaiting feedback

## Tasks Last Run
- 2026-06-30 (run 28429830976): Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-06-29 (run 28361576460): Task 2 (scan remaining files), Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-06-28 (run 28315942087): Task 4 (PR review), Task 7 (Monthly Summary)
- 2026-06-26 (run 28225749722): Task 3 (PR #22), Task 7 partial

## Monthly Summary Issue
- June 2026: issue #4 — updated in run 28429830976

## Previously Checked Off Items
*(none)*

## Open PRs (Efficiency Improver)
- #6: perf(dump-prs-since-commit): replace N git-show calls with single git log batch (open draft)
- #14: perf(ImageResizer): reduce per-file allocations (draft, SUPERSEDED BY #16 — suggest close)
- #16: perf(imageresizer): reduce per-file allocations + fix FileNameFormat cache (open, CI failures infrastructure-only)
- #17: perf(imageresizer): eliminate duplicate GetEncoderIdForDecoder call per file (draft)
- #22: perf(release-scripts): replace O(n²) array append with generic List in PS1 scripts (draft, protected-files flag)

## Open Issues (blocked/awaiting maintainer action)
- #8, #9, #10, #11: sparse-checkout fallback issues (same 3-line YAML change to telemetry-pr-check.yml) — needs manual apply
- #12: perf(telemetry-pr-check) — older sparse-checkout issue, superseded by #11
- #13: ImageResizer analysis — 7 findings; F-1/F-2/F-4/F-5 in #16; F-3 in #17; F-6/F-7 pending; F-8 analyzed
- #19: BenchmarkDotNet proposal — awaiting feedback
