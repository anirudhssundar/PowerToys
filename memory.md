# Efficiency Improver Memory

## Last Updated
2026-06-22 10:25 UTC (run 27945993254)

## Build/Test/Benchmark Commands
- **Build**: `tools\build\build-essentials.cmd` (first time), `tools\build\build.cmd` (incremental)
- **PowerShell**: `./tools/build/build.ps1 -Platform x64 -Configuration Release`
- **Tests**: VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe`. Avoid `dotnet test`.
- **Lint**: StyleCop.Analyzers (C#), clang-format (C++), XamlStyler (XAML)
- **Build requires**: Visual Studio 2022 17.4+ on Windows 10 1803+
- **Note**: Build NOT runnable in Linux CI environment. Sparse checkout has ~2% of files.

## Work In Progress
*(none)*

## Completed Work
- 2026-06-22 (run 27945993254): Task 4 — reviewed all open PRs; no human comments, no new CI issues; Task 2 — re-evaluated F-6/F-8, both are LOW impact; Task 7 — Monthly summary updated (fixed #aw_bench18 → #19)
- 2026-06-21 (run 27899109219): Task 6 — filed Issue #19 (BenchmarkDotNet proposal); Task 7 — Monthly summary updated
- 2026-06-20 (run 27865081378): Task 3 — PR #17 submitted (F-3 eliminate duplicate GetEncoderIdForDecoder); Task 4 — commented on PR #14 recommending closure
- 2026-06-19 (run 27817120053): Task 5 — Commented on issue #13 (status update on 7 findings); Task 2 — Identified F-8 (sizeNameSanitized pre-computation); Task 7 — Monthly summary updated
- 2026-06-18 (run 27748850464): Task 4 — Commented on PR #16 (CI failures infrastructure-only); Task 7 — Monthly summary updated with PR #16 and PR #14 supersession
- 2026-06-17 (fix run — run 27678861580): PR #16 submitted — ImageResizer efficiency fixes (F-1, F-2, F-4, F-5)
  - F-1: FileNameFormat stale cache invalidation (correctness bug + efficiency)
  - F-2: Shared IFileSystem across parallel ResizeOperation instances
  - F-4: SearchValues<char> single-pass path sanitization (9 allocs -> 0-2 per file)
  - F-5: HashSet<string>(OrdinalIgnoreCase) replaces array + ToUpperInvariant()
  - Branch: efficiency/imageresizer-perf-cache-alloc
- 2026-06-15 (run 27566903334): PR #3 MERGED — telemetry-pr-check.js regex optimization
  - Benchmark: 9.12x speedup (89% fewer .test() calls), Node 22, correctness verified
- 2026-06-15 (run 27576993655): PR #6 submitted — dump-prs-since-commit.ps1 batch git-show
  - Estimated 10-30x speedup on typical PowerToys release milestones (200-400 commits)
- 2026-06-16 (runs 27581376978, 27609572469, 27634407962, 27635261800): 5 attempts at sparse-checkout PR for telemetry-pr-check.yml
  - ALL FAILED: .github/workflows/ path is protected — push blocked by protected-file restriction
  - Issues #8, #9, #10, #11 created as fallbacks (all same 3-line change)
  - v4 bundle at efficiency/sparse-checkout-telemetry-ci-v4 — needs manual apply by maintainer

## Optimisation Backlog

### HIGH Priority
*(none pending)*

### MEDIUM Priority
- **Code-Level | ImageResizer F-6**: `GetEncoderPropertySet()` creates new `BitmapPropertySet` per JPEG file; `JpegQualityLevel` is fixed per batch. Caching would require shared state at ResizeBatch level or Settings level. 2 fewer allocs per JPEG file.
  - NOTE: Per-instance caching in ResizeOperation constructor doesn't help (constructor called once per file). Batch-level sharing needed.
- **Code-Level | Launcher EnvironmentHelper**: `foreach (string varName in newEnvironment.Keys.ToList())` — ToList() on Keys collection. Source NOT in sparse checkout.
- **Code-Level | Launcher ResultsViewModel**: `.Where(...).ToList()` materialisation. Source NOT in sparse checkout.
- **Code-Level | ColorPicker/AdvancedPaste UserSettings**: Thread.Sleep retry loops waste CPU during I/O wait. Source NOT in sparse checkout.

### LOW Priority
- **Code-Level | ImageResizer F-8**: `sizeNameSanitized` recomputed per file. Note: pre-computing in ResizeOperation constructor does NOT save anything (constructor called once per file, same as GetDestinationPath). Would need batch-level pre-computation to save across files. 2 string allocs per file saved.
- **Network & I/O | msstore-submissions.yml**: fetches ALL releases without pagination; 8 jq subprocess invocations where 1 would suffice. Runs only on release events.
- **Network & I/O | package-submissions.yml**: wingetcreate.exe downloaded fresh on every release without caching. Runs only on release events.
- **Code-Level | Launcher UserSelectedRecord**: `Records.Keys.ToList()` copy. Source NOT in sparse checkout.

## Efficiency Notes
- **Sparse checkout**: This fork has only ~2% of tracked files. Most PowerToys source (C#/C++) not locally accessible.
- Available workflow files: telemetry-pr-check.yml, auto-label-issues.yml, dependency-review.yml, spelling2.yml, efficiency-improver.lock.yml, msstore-submissions.yml, package-submissions.yml, manual-batch-issue-deduplication.yml, automatic-issue-deduplication.yml
- All workflow files now analyzed for efficiency opportunities.
- Benchmark tool: `node` (v22 available in Linux CI). Can benchmark JS files.
- Build validation impossible in Linux CI — document in PR test status.

## Efficiency Notes — Protected-File Restriction
- `.github/workflows/` files CANNOT be auto-pushed by this automation
- The protected-file restriction blocks ALL push attempts for workflow YAMLs
- DO NOT attempt further auto-PRs for workflow YAML files — always create as issues for manual apply
- Non-workflow changes (scripts, PowerShell) CAN be auto-pushed (PR #3 merged, PR #6 open)

## Efficiency Notes — CI Infrastructure Issues (as of 2026-06-18)
- `check-spelling` action v0.0.26 has a security advisory (credential leak) causing ALL PR checks to fail — this is NOT caused by our code changes
- `dependency-review` fails because Dependency Graph is not enabled for this repo — this is NOT caused by our code changes
- Both CI failures on PR #16 are infrastructure-only

## Efficiency Notes — ResizeOperation Architecture
- ResizeOperation is created ONCE per file (not reused across files in a batch)
- Therefore, caching computed values in the ResizeOperation constructor only helps if GetDestinationPath() is called multiple times per instance (it isn't — called once per file)
- F-6/F-8 cross-batch savings require architectural changes (passing cached values from ResizeBatch, or Settings-level caching)
- BitmapPropertySet is a WinRT type; thread-safety and ownership semantics need investigation before caching

## Backlog Cursor
All workflow files analyzed. Sparse-checkout optimization for telemetry-pr-check.yml BLOCKED (protected-file restriction — needs manual apply). Current focus:
- Task 4: Monitor PRs #6, #16, #17 for review/merge
- PR #14 should be closed (superseded by PR #16) — maintainer action required
- F-6 and F-8 are LOW impact with current architecture; defer unless architectural change is planned
- Issue #19 (BenchmarkDotNet) proposes measurement infrastructure — awaiting maintainer feedback

## Tasks Last Run
- 2026-06-22 (run 27945993254): Task 4 (PR review — no action), Task 2 (F-6/F-8 re-evaluation), Task 7 (Monthly Summary updated)
- 2026-06-21 (run 27899109219): Task 6 (Issue #19 — BenchmarkDotNet), Task 7 (Monthly Summary updated)
- 2026-06-20 (run 27865081378): Task 3 (PR #17 F-3 fix), Task 4 (PR #14 closure comment)
- 2026-06-19 (run 27817120053): Task 5 (comment on issue #13), Task 2 (identified F-8), Task 7 (Monthly Summary updated)

## Monthly Summary Issue
- June 2026: issue #4 — updated in run 27945993254

## Previously Checked Off Items
*(none)*

## Open PRs (Efficiency Improver)
- #6: perf(dump-prs-since-commit): replace N git-show calls with single git log batch (open, no CI failures)
- #14: perf(ImageResizer): reduce per-file allocations in ResizeOperation (draft, SUPERSEDED BY #16 — suggest close)
- #16: perf(imageresizer): reduce per-file allocations + fix FileNameFormat cache (open, awaiting maintainer build+test; CI failures are infrastructure-only)
- #17: perf(imageresizer): eliminate duplicate GetEncoderIdForDecoder call per file (draft, no CI issues)

## Open Issues (blocked/awaiting maintainer action)
- #8, #9, #10, #11: all sparse-checkout fallback issues (same 3-line change) — BLOCKED by protected-file restriction, needs manual apply by maintainer; close #8-#10 once #11 is applied
- #12: perf(telemetry-pr-check) — older sparse-checkout issue, superseded by #11
- #13: ImageResizer analysis issue — 7 findings; F-1, F-2, F-4, F-5 addressed in PR #16; F-3 in PR #17; F-6 and F-7 pending; F-8 analyzed
- #19: BenchmarkDotNet micro-benchmarks proposal — awaiting maintainer feedback
