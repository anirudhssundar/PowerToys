# Efficiency Improver Memory

## Last Updated
2026-06-16 (run 27609572469)

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
- 2026-06-15 (run 27566903334): PR #3 MERGED — telemetry-pr-check.js regex optimization
  - Benchmark: 9.12× speedup (89% fewer .test() calls), Node 22, correctness verified
- 2026-06-15 (run 27576993655): PR #6 submitted — dump-prs-since-commit.ps1 batch git-show
  - Estimated 10–30× speedup on typical PowerToys release milestones (200–400 commits)
- 2026-06-16 (run 27609572469): PR submitted — telemetry-pr-check.yml sparse-checkout (v2)
  - Branch: efficiency/sparse-checkout-telemetry-ci-v2
  - ~99.99% checkout data reduction per PR invocation (~12 KB vs hundreds of MB)
  - Closes issue #9 and #8 (both failed-push fallback issues from previous runs)

## Optimisation Backlog

### HIGH Priority
*(none pending)*

### MEDIUM Priority
- **Code-Level | Launcher EnvironmentHelper**: `foreach (string varName in newEnvironment.Keys.ToList())` — ToList() on Keys collection. Source NOT in sparse checkout.
- **Code-Level | Launcher ResultsViewModel**: `.Where(...).ToList()` materialisation. Source NOT in sparse checkout.
- **Code-Level | ColorPicker/AdvancedPaste UserSettings**: Thread.Sleep retry loops waste CPU during I/O wait. Source NOT in sparse checkout.

### LOW Priority
- **Network & I/O | msstore-submissions.yml**: fetches ALL releases without pagination; 8 jq subprocess invocations where 1 would suffice. Runs only on release events.
- **Network & I/O | package-submissions.yml**: wingetcreate.exe downloaded fresh on every release without caching. Runs only on release events.
- **Code-Level | Launcher UserSelectedRecord**: `Records.Keys.ToList()` copy. Source NOT in sparse checkout.

## Efficiency Notes
- **Sparse checkout**: This fork has only ~2% of tracked files. Most PowerToys source (C#/C++) not locally accessible.
- Available workflow files: telemetry-pr-check.yml, auto-label-issues.yml, dependency-review.yml, spelling2.yml, efficiency-improver.lock.yml, msstore-submissions.yml, package-submissions.yml, manual-batch-issue-deduplication.yml, automatic-issue-deduplication.yml
- All workflow files now analyzed for efficiency opportunities.
- `auto-label-issues.yml` uses github-script (no checkout) — no checkout optimization possible
- `dependency-review.yml` uses checkout but dependency-review-action needs it for manifest files
- `spelling2.yml` uses check-spelling action with internal checkout — not controllable
- `automatic-issue-deduplication.yml` uses pelikhan/action-genai-issue-dedup — no checkout
- `msstore-submissions.yml` / `package-submissions.yml` run only on releases — LOW impact
- Benchmark tool: `node` (v22 available in Linux CI). Can benchmark JS files.
- Build validation impossible in Linux CI — document in PR test status.

## Backlog Cursor
All available workflow files now analyzed. Next focus: wait for open PRs to be reviewed/merged, or investigate sparse checkout limitations for other opportunities.

## Tasks Last Run
- 2026-06-16 (run 27609572469): Task 3 (sparse-checkout-telemetry-ci-v2 PR), Task 2 (workflow scan), Task 7 (Monthly Summary updated)
- 2026-06-15 (run 27581376978): Task 3 (sparse-checkout-telemetry-ci PR — failed push), Task 4 (PR #6 review), Task 7 (Monthly Summary updated)
- 2026-06-15 (run 27576993655): Task 3 (dump-prs-since-commit.ps1 batch git-show PR)
- 2026-06-15 (run 27566903334): Task 3 (telemetry regex PR — merged), Task 7 (Monthly Summary created)

## Monthly Summary Issue
- June 2026: issue #4 — updated in run 27609572469

## Previously Checked Off Items
*(none)*

## Open PRs (Efficiency Improver)
- #6: perf(dump-prs-since-commit): replace N git-show calls with single git log batch
- new (branch efficiency/sparse-checkout-telemetry-ci-v2): sparse-checkout for telemetry-pr-check.yml (PR number TBA — assigned after workflow completes)
