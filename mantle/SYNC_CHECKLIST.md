# Upstream Sync Checklist — mantle-m365-docs

Walk this whenever upstream moves before merging it into `main`. ThomasKur/M365Documentation publishes no releases or tags, so the weekly `upstream-watch` workflow compares upstream `main` with the commit pinned in `mantle/UPSTREAM_PIN.md` and opens an issue in this repo when they differ.

## 1. Read what changed

- [ ] Read the commit list and compare link in the watcher issue.
- [ ] Check whether upstream touched any file listed in `mantle/DIVERGENCES.md`. Expect merge conflicts there, and decide per entry: keep it, adapt it, or drop it because upstream now covers it (then delete the entry).

## 2. Engagement timing

- [ ] If a client engagement is mid-collection, finish it on the current pin first.

## 3. Merge on a branch

```bash
git switch -c upstream-sync/<sha-prefix> main
git fetch upstream
git merge <upstream commit SHA>
```

The Mantle patches are already on `main`, so the merge carries them forward; resolve any conflicts so that every `DIVERGENCES.md` entry still works.

## 4. Verify

- [ ] Mantle tests: `pwsh -NoProfile -Command "Invoke-Pester -Path ./PSModule/Tests/Connect-M365Doc.RawToken.Tests.ps1,./PSModule/Tests/Connect-M365Doc.MsalOptional.Tests.ps1,./PSModule/Tests/Invoke-DocGraph.ErrorHandling.Tests.ps1"`.
- [ ] Full suite: `pwsh -NoProfile -Command "Invoke-Pester -Path ./PSModule/Tests"`, then compare the failure list with the baseline below. Only new failures matter.
- [ ] In mantle-m365-assessment, `uv run mantle-engagement doctor` reports every Mantle patch present.
- [ ] Live run against the test tenant: `uv run mantle-engagement run test-tenant --tenant <test tenant>`, then open the Tenant Configuration report in the delivery package.

### Known upstream test failures (baseline)

On PowerShell 7 (checked on macOS, 2026-11-17, upstream `963b74b`), 32 upstream tests fail with or without the Mantle patches. Upstream runs no CI for these tests.

- 25 fail with `Unable to find type [DocSection]`: the test files extract the module's classes from the `.psm1` and run them as a scriptblock created at run time, and PowerShell 7 does not resolve those class names in the collector files dot-sourced afterwards. Affected: `CloudPrintAndInformationProtection`, `Copilot`, `Invoke-DocGraph`, `Optimize-M365Doc`, and most of `Write-M365DocWord`.
- 7 assert Windows or Word-dependency behavior (`WordDependencyLoading`, parts of `Write-M365DocWord`).

## 5. Record and merge

- [ ] Update `mantle/UPSTREAM_PIN.md` (commit SHA, published date, review date, reviewer) and this baseline if it changed.
- [ ] Update "Upstream version last reviewed" on each `DIVERGENCES.md` entry.
- [ ] Open a PR to `main` with this checklist completed in the body, merge it, and close the watcher issue.
