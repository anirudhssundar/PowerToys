# Efficiency Improver Memory

## Last Updated
2026-06-12

## Build/Test/Benchmark Commands
- **Build**: `tools\build\build-essentials.cmd` (first time), `tools\build\build.cmd` (incremental)
- **PowerShell**: `./tools/build/build-essentials.ps1`, `./tools/build/build.ps1`
- **Tests**: VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe`. Avoid `dotnet test`.
- **Lint**: StyleCop.Analyzers (C#), clang-format (C++), XamlStyler (XAML)
- **Build requires**: Visual Studio 2022 17.4+ on Windows 10 1803+
- **Note**: Build NOT runnable in Linux CI environment

## Work In Progress
- PR created: `efficiency/stringmatcher-hoist-preprocessing` branch
  - Optimizes `StringMatcher.FuzzyMatch` in PowerToys Run launcher
  - Hoists O(n) redundant preprocessing out of per-startIndex loop
  - Eliminates per-character string allocations in inner loop
  - Replaces O(n log n) LINQ sort with O(n) linear scan
  - Estimated ~50x reduction in allocations per FuzzyMatch call

## Completed Work
*(none yet — first run)*

## Optimisation Backlog

### HIGH Priority
- **Code-Level | Launcher StringMatcher**: ✅ PR submitted — hoist preprocessing out of per-startIndex loop. ~50x allocation reduction per keystroke.
- **Code-Level | Launcher StringMatcher**: `CalculateSearchScore` uses `query.Count(c => !char.IsWhiteSpace(c))` LINQ — could be replaced with a simple loop, but this is only called on a match, lower priority.

### MEDIUM Priority
- **Code-Level | SettingsRepository retry loop**: `Watcher_Changed` uses `Thread.Sleep(100)` in a loop (5 retries). Could use exponential backoff or `WaitForChanged`. Minor.
- **Code-Level | ColorPicker UserSettings**: `Thread.Sleep(500)` in retry loops (lines 195, 205).
- **Code-Level | AdvancedPaste UserSettings**: Similar Thread.Sleep retry patterns.
- **Code-Level | Launcher EnvironmentHelper**: `foreach (string varName in newEnvironment.Keys.ToList())` — unnecessary `.ToList()` on a Keys collection.
- **Code-Level | Launcher ResultsViewModel**: `foreach (var nonHistoryResult in sorted.Where(x => ...).ToList())` — unnecessary `.ToList()` materialisation.

### LOW Priority
- **Code-Level | Launcher UserSelectedRecord**: `foreach (var key in Records.Keys.ToList())` — unnecessary `.ToList()`.

## Efficiency Notes
- PowerToys Run launcher is the highest-impact module for code-level efficiency. It runs on every user keystroke, calling StringMatcher.FuzzySearch 6+ times per Win32 program entry.
- Build is Windows-only (VS 2022 required), so no build validation possible in Linux CI.
- Tests are in `src/modules/launcher/Wox.Test/FuzzyMatcherTest.cs`.

## Backlog Cursor
Next: Check for more high-impact allocation patterns in launcher plugins (Microsoft.Plugin.Program Win32Program).

## Tasks Last Run
- 2026-06-12: Task 1 (Discover Commands), Task 2 (Identify Opportunities), Task 3 (Implement), Task 7 (Monthly Summary)

## Previously Checked Off Items
*(none)*
