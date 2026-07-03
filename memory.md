# Efficiency Improver Memory

## Last Updated
2026-07-03 07:49 UTC (run 28646434076)

## Build/Test/Benchmark Commands
- **Build**: `tools\build\build-essentials.cmd` (first time), `tools\build\build.cmd` (incremental)
- **PowerShell**: `./tools/build/build.ps1 -Platform x64 -Configuration Release`
- **Tests**: VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe`. Avoid `dotnet test`.
- **Lint**: StyleCop.Analyzers (C#), clang-format (C++), XamlStyler (XAML)
- **Note**: Build NOT runnable in Linux CI. Sparse checkout has ~2% of files.

## Work In Progress
*(none)*

## Completed Work
- 2026-07-03 (run 28646434076): Task 2 — analyzed StringMatcher.cs (SM-1 through SM-4); Task 7 — monthly summary updated; Task 4 — no new PR activity
- 2026-07-02 (run 28574372542): Task 4 — no new human activity on PRs; Task 2 — scanned tools/ via GitHub API (LOW findings only); Task 7 — monthly summary updated
- 2026-07-01 (run 28504751794): Task 4 — verified all PRs, no new human activity; Task 7 — closed June #4, created July 2026 monthly #24
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
- **Code-Level | StringMatcher SM-1**: O(L) repeated allocs in outer FuzzyMatch loop. In inner `FuzzyMatch` called L times: `query.Trim()`, `ToUpper()`, `Split()` all repeated. Move to outer method. Issue: #aw_sm_issue (pending creation, ~#25)
- **Code-Level | StringMatcher SM-2**: `CalculateClosestSpaceIndex` uses LINQ `.OrderBy().Where().FirstOrDefault()` — O(k log k). `spaceIndices` is already sorted ascending; use reverse linear scan O(k), zero-alloc. Issue: same as SM-1.
- **Code-Level | ImageResizer F-8**: `sizeNameSanitized` = `sizeName.Replace('\\','_').Replace('/','_')` — same for all files in a batch. Pre-computing saves 2 allocs per file. DEFERRED: implement AFTER PR #16 merges (conflict avoidance — both touch ResizeBatch.cs).
- **Code-Level | ImageResizer F-6**: `GetEncoderPropertySet()` creates new `BitmapPropertySet` per JPEG; fixed per batch. Thread-safety of sharing needs investigation.
- **Code-Level | Launcher EnvironmentHelper/ResultsViewModel**: `.ToList()` copies. Source NOT in sparse checkout.
- **Code-Level | ColorPicker/AdvancedPaste UserSettings**: Thread.Sleep retry loops. Source NOT in sparse checkout.

### LOW Priority
- **Code-Level | StringMatcher SM-3**: `AllPreviousCharsMatched` missing early exit — loop continues after mismatch. Add `return false;` on first mismatch.
- **Code-Level | StringMatcher SM-4**: `query.Count(c => !char.IsWhiteSpace(c))` in `CalculateSearchScore` — LINQ alloc. Replace with indexed loop.
- **Network & I/O | msstore-submissions.yml/package-submissions.yml**: 8 jq subprocess invocations; no wingetcreate caching. Protected-file paths.
- **Code-Level | tools/mcp/github-artifacts/server.js**: `fetchAllComments()` uses `comments.concat(pageComments)` inside loop — O(n) copy per page. Dev tool, low frequency, negligible impact.
- **Code-Level | tools/Test-AutoLabelProduct.ps1**: nested `Where-Object` label filter is O(n²) but with tiny n (max 100 issues × max ~10 labels). No action needed.

## Efficiency Notes
- **Sparse checkout**: ~2% of tracked files locally accessible.
- **ALL sparse-checkout files fully analyzed** (ImageResizer Models, auto-cherry-pick.ps1, Program.cs) — no new findings beyond F-1 through F-8.
- **tools/ directory analyzed via GitHub API** (2026-07-02): build-common.ps1 and server.js — only LOW priority findings.
- **src/common/ directory analyzed via GitHub API** (2026-07-03): StringMatcher.cs — SM-1/SM-2 MEDIUM, SM-3/SM-4 LOW. Logger.cs — no actionable findings (only startup code).
- **Protected-file restriction**: `.github/workflows/` and `.github/skills/` CANNOT be auto-pushed. Always create as issues for manual apply.
- **CI Infrastructure Issues**: `check-spelling` v0.0.26 has security advisory (fails all PRs); `dependency-review` fails (Dependency Graph not enabled). NOT caused by our code.
- **Token budget**: Keep monthly summary concise to avoid exhaustion.
- **ResizeOperation architecture**: One instance per file, not reused. F-6/F-8 require batch-level changes.

## Backlog Cursor
Analyzed: ImageResizer Models, auto-cherry-pick.ps1, Program.cs, tools/ (via API), src/common/ManagedCommon/Logger.cs, src/common/Common.Search/FuzzSearch/StringMatcher.cs
Next unexplored areas (via GitHub API):
- src/common/ManagedCommon/SerializationContext/ (AOT serialization context)
- src/common/LanguageModelProvider/ (AI feature, possibly expensive)
- src/common/Display/ (C++ DPI/monitor code)
- src/modules/ high-value targets (FancyZones, Run/Launcher, etc.)

## Tasks Last Run
- 2026-07-03 (run 28646434076): Task 2 (StringMatcher scan), Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-07-02 (run 28574372542): Task 4 (PR check), Task 2 (tools/ scan), Task 7 (Monthly Summary)
- 2026-07-01 (run 28504751794): Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-06-30 (run 28429830976): Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-06-29 (run 28361576460): Task 2 (scan remaining files), Task 4 (PR check), Task 7 (Monthly Summary)
- 2026-06-28 (run 28315942087): Task 4 (PR review), Task 7 (Monthly Summary)

## Monthly Summary Issue
- June 2026: issue #4 — CLOSED in run 28504751794
- July 2026: issue #24 — created in run 28504751794

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
- StringMatcher analysis (SM-1 through SM-4) — newly created this run, pending issue number assignment
