# Changelog

All notable changes to the Procurement Tool will be documented here.

## [Unreleased]
- Fixed: backupState() now logs a console warning if the automatic localStorage
  backup fails, instead of silently swallowing the error. Verified by simulating
  a localStorage failure and confirming the warning fires, then confirming normal
  backups still succeed silently as before.
- Fixed: order ID fallback now uses the shared uid() function instead of its
  own separate, weaker ID generation. Verified: no other code in the file
  depends on the old "ord_" prefix, and confirmed uid() produces a proper
  ID when the fallback path is triggered.
- Fixed: moveFileWithinBase() now logs a console warning if deleting the old
  file fails after copying to the new location, instead of silently continuing.
  Previously this could leave an orphaned duplicate file on disk with no
  indication. Verified using a scratch test folder: confirmed normal moves
  still work silently, and confirmed a forced delete failure now logs a clear
  warning and correctly reproduces the orphaned-duplicate scenario.

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
