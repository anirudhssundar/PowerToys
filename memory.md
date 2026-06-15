# Efficiency Improver Memory

## Last Updated
2026-06-15

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
- 2026-06-15 (run 27566903334): PR #3 created — telemetry-pr-check.js regex optimization
  - Branch: efficiency/combine-telemetry-regexes
  - Benchmark: 9.12× speedup (89% fewer .test() calls), Node 22, correctness verified
- 2026-06-15 (earlier run): Memory records StringMatcher + ImageResizer PRs but branches not found locally; likely failed silently. Ignore.

## Optimisation Backlog

### HIGH Priority
*(none pending — telemetry regex PR submitted)*

### MEDIUM Priority
- **Code-Level | Launcher EnvironmentHelper**: `foreach (string varName in newEnvironment.Keys.ToList())` — ToList() on Keys collection. Source NOT in sparse checkout.
- **Code-Level | Launcher ResultsViewModel**: `.Where(...).ToList()` materialisation. Source NOT in sparse checkout.
- **Code-Level | ColorPicker/AdvancedPaste UserSettings**: Thread.Sleep retry loops waste CPU during I/O wait. Source NOT in sparse checkout.

### LOW Priority
- **Code-Level | Launcher UserSelectedRecord**: `Records.Keys.ToList()` copy. Source NOT in sparse checkout.

## Efficiency Notes
- **Sparse checkout**: This fork has only ~2% of tracked files. Most PowerToys source (C#/C++) not locally accessible.
- `telemetry-pr-check.js` is the only JavaScript CI script in sparse checkout. PR #3 submitted.
- GitHub code search for this fork returns no results for source patterns — confirms sparse checkout limitation.
- Benchmark tool: `node` (v22 available in Linux CI). Can benchmark JS files.
- Build validation impossible in Linux CI — document in PR test status.

## Backlog Cursor
Next: Look at other available workflow YAML files for CI efficiency (conditional steps, caching opportunities). Or investigate the `winmd-api-search` cache generator Program.cs (available in sparse checkout).

## Tasks Last Run
- 2026-06-15 (run 27566903334): Task 3 (telemetry-pr-check.js regex optimization), Task 7 (Monthly Summary created as issue)
- 2026-06-15 (earlier run): Task 4, Task 7

## Monthly Summary Issue
- June 2026: created in run 27566903334 (issue number TBD — awaits safeoutputs resolution)

## Previously Checked Off Items
*(none)*
