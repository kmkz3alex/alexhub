# Changelog

All notable changes to the Procurement Tool will be documented here.

## [Unreleased]
- Fixed: backupState() now logs a console warning if the automatic localStorage
  backup fails, instead of silently swallowing the error. Verified by simulating
  a localStorage failure and confirming the warning fires, then confirming normal
  backups still succeed silently as before.

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
