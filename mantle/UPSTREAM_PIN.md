# Upstream Pin — mantle-m365-docs

## Primary upstream

| Item | Value |
|---|---|
| Upstream repo | [ThomasKur/M365Documentation](https://github.com/ThomasKur/M365Documentation) |
| Pinned ref | commit `963b74b0afe43a3bef14357807196eb0feeee7f5` (`upstream/main` as of fork) |
| Upstream published | 2026-10-09 |
| Mantle review date | 2026-11-11 |
| Reviewer | ascheer-mntl |

**Note on pinning strategy:** ThomasKur publishes neither GitHub releases nor tags on this repo. Mantle pins by commit SHA on `upstream/main`. The weekly upstream-watcher action compares `upstream/main` HEAD against this SHA and opens an issue when drift exceeds one commit.

## How to bump

1. The upstream-watcher issue summarizes commits since the pinned SHA.
2. Walk `mantle/SYNC_CHECKLIST.md`.
3. Create branch `upstream-sync/<new-sha-prefix>`, merge `upstream/main` to the new pin SHA, re-apply all patches from `mantle/DIVERGENCES.md`, run tests.
4. Update this file + open a PR to `main`.
