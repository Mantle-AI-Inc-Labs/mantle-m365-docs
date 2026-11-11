# Upstream Sync Impact Assessment — mantle-m365-docs

Walk this checklist before merging upstream drift into `main`.

## 1. Engagement impact

- [ ] Which clients are currently engaged?
- [ ] Do any in-flight Config Reference deliverables depend on collectors in the changed set?
- [ ] If yes: hold the sync until the engagement ships, OR scope the sync to non-affected collectors.

## 2. Collector surface changes

- [ ] Did upstream add new `Get-*.ps1` collectors? If yes, review for inclusion in Mantle's standard tier (baseline / full) via the next catalog update.
- [ ] Did upstream remove or rename existing collectors? If yes: update `mantle-m365-render/config_reference_projection.json` so the curation catalog doesn't reference stale section names.
- [ ] Did upstream change the Graph endpoint for any collector? If yes: update the scope list used by the orchestrator's `Connect-MgGraph` call.

## 3. JSON tree changes

- [ ] Did upstream change the structure of `Write-M365DocJson` output? If yes: bump `mantle-m365-render`'s input-schema version.
- [ ] Did upstream modify any `Data/LabelTranslation/*.json` entry? If yes: re-vendor the affected file into `mantle-m365-render/src/mantle_render/translation_data/` and note in `DIVERGENCES.md` if any Mantle customization was overwritten.

## 4. Mantle patch compatibility

- [ ] Does the upstream drift conflict with any patch in `DIVERGENCES.md`?
- [ ] The high-priority patch to watch is the `Connect-M365Doc -ExistingMgGraphContext` enhancement — if `Connect-M365Doc.ps1` changed upstream, re-apply carefully.

## 5. Regression test

- [ ] Run the sync branch's collectors against a known-state reference tenant (sandbox) and compare the JSON tree diff.
- [ ] Zero unexpected diffs required before merge.

## Merge workflow

```bash
git checkout -b upstream-sync/<sha-prefix>
git fetch upstream
git merge upstream/main
# re-apply patches from mantle/DIVERGENCES.md
# run tests
# PR to main of fork with this checklist completed in the PR body
```
