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

## MSAL.PS optional when a raw token is supplied

- **Files:**
  - `PSModule/M365Documentation/M365Documentation.psd1` (MSAL.PS removed from `RequiredModules`)
  - `PSModule/M365Documentation/M365Documentation.psm1` (no eager MSAL.PS import)
  - `PSModule/M365Documentation/Functions/Connect-M365Doc.ps1` (imports MSAL.PS only for MSAL-based sign-in; `-token` typed by name)
  - `PSModule/Tests/Connect-M365Doc.MsalOptional.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** MSAL.PS (4.37.0) has been unmaintained since 2021 and its identity library conflicts with the Microsoft Graph SDK's when both load in one session. mantle-engagement always passes a raw token (`Connect-M365Doc -AccessToken`), so it never needs MSAL.PS. The module now loads MSAL.PS only for the interactive, client-secret, and custom-token sign-in paths, which still work as before when MSAL.PS is installed.
- **Cost of carrying:** Low. Conflicts are possible if upstream edits the manifest's `RequiredModules`, the dependency block in the `.psm1`, or `Connect-M365Doc`'s parameter block. The manifest and `.psm1` carry upstream signature blocks, which these edits invalidate; that already applied to earlier Mantle patches and only matters under an AllSigned execution policy.
- **Upstream issue / PR:** Offered upstream in [ThomasKur/M365Documentation#93](https://github.com/ThomasKur/M365Documentation/pull/93) (2026-11-17) with the RawToken parameter set.
- **Introduced:** 2026-11-17, fork PR #6 (`mantle/msal-optional` branch)
- **Review date:** 2027-02-17

## README notice pointing to mantle-engagement

- **Files:** `README.md` (notice block above the upstream content)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** The fork's README is otherwise the upstream project's, with upstream run instructions. Mantle staff opening this repo need to know that client engagements run through `mantle-engagement` in mantle-m365-assessment, and that this repo is run directly only for patch development and upstream syncs.
- **Cost of carrying:** Minimal. The notice sits above the upstream content; on sync, keep it at the top if upstream rewrites the README's opening lines.
- **Upstream issue / PR:** Not applicable (Mantle-specific).
- **Introduced:** 2026-11-17, fork PR #4 (`mantle/readme-entry-point` branch)
- **Review date:** 2027-02-17

## Invoke-DocGraph returns nothing on 403 / 401

- **Files:**
  - `PSModule/M365Documentation/Internal/Helper/Invoke-DocGraph.ps1`
  - `PSModule/Tests/Invoke-DocGraph.ErrorHandling.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** On Forbidden and Unauthorized responses, upstream logs a warning and then returns `$value`, which at that point holds the caught `ErrorRecord`. Collectors store it as section data. `Write-M365DocJson` then serializes the error's exception, method, and assembly metadata to depth 20; on a test tenant with ~20 forbidden sections the process grew past 3.7 GB without finishing the file. The fix returns `$null`, matching upstream's existing `AadPremiumLicenseRequired` branch, so forbidden sections are skipped with their warning instead of polluting the output.
- **Cost of carrying:** Low. Two `return $null` lines inside existing branches. A conflict is likely only if upstream rewrites the catch block, which would probably fix the bug itself.
- **Upstream issue / PR:** Offered upstream as [ThomasKur/M365Documentation#92](https://github.com/ThomasKur/M365Documentation/pull/92) (2026-11-17). Drop this entry when it merges and the pin moves past it.
- **Introduced:** 2026-11-17, fork PR #3 (`mantle/docgraph-error-records` branch; #2 was closed when its stacked base branch was deleted)
- **Review date:** 2027-02-17

## Connect-M365Doc RawToken parameter set

- **Files:**
  - `PSModule/M365Documentation/Functions/Connect-M365Doc.ps1`
  - `PSModule/Tests/Connect-M365Doc.RawToken.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** The Mantle engagement orchestrator runs `mantle-m365-safeguards-audit` and `mantle-m365-docs` back-to-back against the same tenant. Upstream `Connect-M365Doc` only accepts an MSAL.PS-acquired token (`CustomToken` parameter set typed to `Microsoft.Identity.Client.AuthenticationResult`), which forces a second browser consent after the Microsoft.Graph session the audit fork already authenticated. The new `RawToken` parameter set lets the orchestrator extract the raw bearer from the live MgGraph session and hand it directly to Connect-M365Doc, giving the operator one browser consent instead of two.
- **Cost of carrying:** Low. Patch is additive — introduces a new parameter set, does not modify existing parameter sets. Likely to merge cleanly on upstream syncs unless upstream refactors the param block. Pester test covers the new path.
- **Upstream issue / PR:** Offered upstream as [ThomasKur/M365Documentation#93](https://github.com/ThomasKur/M365Documentation/pull/93) (2026-11-17), together with the MSAL.PS change. Drop this entry when it merges and the pin moves past it.
- **Introduced:** 2026-11-11, fork PR #1 (`mantle/connect-reuse-token` branch)
- **Review date:** 2027-02-11
