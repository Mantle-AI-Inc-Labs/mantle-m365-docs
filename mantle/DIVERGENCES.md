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

## Connect-M365Doc RawToken parameter set

- **Files:**
  - `PSModule/M365Documentation/Functions/Connect-M365Doc.ps1`
  - `PSModule/Tests/Connect-M365Doc.RawToken.Tests.ps1` (new)
- **Upstream version last reviewed:** `963b74b0afe43a3bef14357807196eb0feeee7f5`
- **Rationale:** The Mantle engagement orchestrator runs `mantle-m365-audit` and `mantle-m365-docs` back-to-back against the same tenant. Upstream `Connect-M365Doc` only accepts an MSAL.PS-acquired token (`CustomToken` parameter set typed to `Microsoft.Identity.Client.AuthenticationResult`), which forces a second browser consent after the Microsoft.Graph session the audit fork already authenticated. The new `RawToken` parameter set lets the orchestrator extract the raw bearer from the live MgGraph session and hand it directly to Connect-M365Doc, giving the operator one browser consent instead of two.
- **Cost of carrying:** Low. Patch is additive — introduces a new parameter set, does not modify existing parameter sets. Likely to merge cleanly on upstream syncs unless upstream refactors the param block. Pester test covers the new path.
- **Upstream issue / PR:** None opened yet. If this proves durable, submit upstream so this entry can be dropped.
- **Introduced:** 2026-11-11, fork PR TBD (`mantle/connect-reuse-token` branch)
- **Review date:** 2027-02-11
