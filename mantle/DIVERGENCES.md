# Mantle Divergences from Upstream

Every Mantle-specific change to upstream code requires an entry here, created in the same PR as the change.

Entries are reverse-chronological. Use H2 for each divergence.

## Entry schema

- **File(s):** path(s) touched
- **Upstream version last reviewed:** pinned commit SHA
- **Rationale:** why Mantle needs this deviation
- **Cost of carrying:** merge conflicts expected on upstream sync; test burden; who maintains it
- **Upstream issue / PR:** link if we've tried to push the change upstream
- **Introduced:** date and fork PR #
- **Review date:** when to next re-evaluate whether this can be dropped

---

## Invoke-DocGraph returns nothing on 403 / 401

- **Files:**
  - `PSModule/M365Documentation/Internal/Helper/Invoke-DocGraph.ps1`
  - `PSModule/Tests/Invoke-DocGraph.ErrorHandling.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** On Forbidden and Unauthorized responses, upstream logs a warning and then returns `$value`, which at that point holds the caught `ErrorRecord`. Collectors store it as section data. `Write-M365DocJson` then serializes the error's exception, method, and assembly metadata to depth 20; on a test tenant with ~20 forbidden sections the process grew past 3.7 GB without finishing the file. The fix returns `$null`, matching upstream's existing `AadPremiumLicenseRequired` branch, so forbidden sections are skipped with their warning instead of polluting the output.
- **Cost of carrying:** Low. Two `return $null` lines inside existing branches. A conflict is likely only if upstream rewrites the catch block, which would probably fix the bug itself.
- **Upstream issue / PR:** None opened yet. This is a plain upstream bug and should be reported to ThomasKur/M365Documentation so the entry can be dropped.
- **Introduced:** 2026-11-17, fork PR TBD (`mantle/docgraph-error-records` branch, stacked on `mantle/connect-reuse-token`)
- **Review date:** 2027-02-17

## Connect-M365Doc RawToken parameter set

- **Files:**
  - `PSModule/M365Documentation/Functions/Connect-M365Doc.ps1`
  - `PSModule/Tests/Connect-M365Doc.RawToken.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** The Mantle engagement orchestrator runs `mantle-m365-audit` and `mantle-m365-docs` back-to-back against the same tenant. Upstream `Connect-M365Doc` only accepts an MSAL.PS-acquired token (`CustomToken` parameter set typed to `Microsoft.Identity.Client.AuthenticationResult`), which forces a second browser consent after the Microsoft.Graph session the audit fork already authenticated. The new `RawToken` parameter set lets the orchestrator extract the raw bearer from the live MgGraph session and hand it directly to Connect-M365Doc, giving the operator one browser consent instead of two.
- **Cost of carrying:** Low. Patch is additive — introduces a new parameter set, does not modify existing parameter sets. Likely to merge cleanly on upstream syncs unless upstream refactors the param block. Pester test covers the new path.
- **Upstream issue / PR:** None opened yet. If this proves durable, submit upstream so this entry can be dropped.
- **Introduced:** 2026-11-11, fork PR #1 (`mantle/connect-reuse-token` branch)
- **Review date:** 2027-02-11
