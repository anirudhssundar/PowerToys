# Efficiency Improver Memory

## Last Updated
2026-06-15

## Build/Test/Benchmark Commands
- **Build**: `tools\build\build-essentials.cmd` (first time), `tools\build\build.cmd` (incremental)
- **PowerShell**: `./tools/build/build-essentials.ps1`, `./tools/build/build.ps1`
- **Tests**: VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe`. Avoid `dotnet test`.
- **Lint**: StyleCop.Analyzers (C#), clang-format (C++), XamlStyler (XAML)
- **Build requires**: Visual Studio 2022 17.4+ on Windows 10 1803+
- **Note**: Build NOT runnable in Linux CI environment

## Work In Progress
*(none — both planned PRs have been submitted)*

## Completed Work
- 2026-06-15: PR created for StringMatcher hoist preprocessing (branch: efficiency/stringmatcher-hoist-preprocessing-43048c4a1c55ede7)
- 2026-06-15: PR created for ImageResizer filename processing (branch: efficiency/filename-processing-perf-3ee3c271cc336071-1dcb039a498ffd43)

## Optimisation Backlog

### HIGH Priority
- **Code-Level | Launcher StringMatcher**: ✅ PR submitted — hoist preprocessing out of per-startIndex loop. ~95% reduction in ToUpper/Split calls per keystroke.

### MEDIUM Priority
- **Code-Level | ImageResizer GetDestinationPath**: ✅ PR submitted — HashSet + string.Create. −7 allocations per file in batch.
- **Code-Level | SettingsRepository retry loop**: `Watcher_Changed` uses `Thread.Sleep(100)` in a loop (5 retries). Could use exponential backoff or `WaitForChanged`. Minor.
- **Code-Level | ColorPicker UserSettings**: `Thread.Sleep(500)` in retry loops (lines 195, 205).
- **Code-Level | AdvancedPaste UserSettings**: Similar Thread.Sleep retry patterns.
- **Code-Level | Launcher EnvironmentHelper**: `foreach (string varName in newEnvironment.Keys.ToList())` — unnecessary `.ToList()` on a Keys collection.
- **Code-Level | Launcher ResultsViewModel**: `foreach (var nonHistoryResult in sorted.Where(x => ...).ToList())` — unnecessary `.ToList()` materialisation.

### LOW Priority
- **Code-Level | Launcher UserSelectedRecord**: `foreach (var key in Records.Keys.ToList())` — unnecessary `.ToList()`.

## Efficiency Notes
- PowerToys Run launcher is the highest-impact module for code-level efficiency. It runs on every user keystroke.
- Build is Windows-only (VS 2022 required), so no build validation possible in Linux CI.
- Previous runs hit token exhaustion (429 error) before creating PRs. Keep runs focused.

## Backlog Cursor
Next: Investigate high-impact allocation patterns in launcher plugins (Microsoft.Plugin.Program Win32Program). Then look at the MEDIUM-priority Thread.Sleep retry patterns which waste CPU during I/O wait.

## Tasks Last Run
- 2026-06-15: Task 4 (Create PRs for existing branches), Task 7 (Monthly Summary)
- 2026-06-12: Task 1 (Discover Commands), Task 2 (Identify Opportunities), Task 3 (Implement changes) — branch pushed but PR creation failed (token exhaustion)

## Previously Checked Off Items
*(none)*
