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
- Fixed: exportComparisonXLSX_ExcelJS() now writes real formulas for line-item
  totals (materials rows 6-39, TRANSPORT row 40, DISCOUNT row 41), matching the
  template's own convention (e.g. F6: =E6*C6), instead of pre-computed numbers.
  This was a long-standing regression from an earlier workaround for a
  shared-formula issue; the underlying cleanup (clearTemplateLineTotalFormulas)
  already resolved that issue, so formulas are now safe to write again.
  Verified by exporting a real comparison with multiple suppliers, transport,
  and discount values: confirmed formulas appear correctly in Excel and totals
  recalculate automatically when price or quantity is edited manually.
- Fixed: comparison export no longer requires re-selecting comparison list.xlsx
  every time. The app now remembers the template file via a persistent File
  System Access handle (same pattern already used for the Orders storage
  folder), and always reads the file's current content on each export - so
  future edits to the template are automatically reflected without any code
  changes. Handles three edge cases distinctly: permission denied (clear error,
  stops), template file moved/renamed/deleted (clear message, prompts to
  re-select), and unsupported browsers (falls back to prior per-export prompt
  behavior). Added a "Change Comparison Template" option to the Settings menu
  for intentionally switching files. Verified: first-time pick and save, silent
  reuse on subsequent exports, manual template change via the new menu option,
  and the stale-file recovery path (tested by temporarily renaming the template
  file) - all confirmed working.

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
