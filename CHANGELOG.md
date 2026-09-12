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
- Added: Import Historical Quotes now has a review-and-confirm fallback for
  materials with no exact historical match. Previously, any difference in
  material name wording (word order, "100T" vs "100TON", etc) caused a silent
  complete miss. Exact matches remain fully automatic and unchanged. For items
  with no exact match, a new modal shows related candidates (materials sharing
  a number and a meaningful word with the item), ranked by how many meaningful
  words are actually shared, letting the user confirm via checkbox before
  anything is imported - nothing happens automatically. Diagnosed from two
  real issues the user encountered: (1) a data-modeling gap where generic
  material names like "mob/demob" don't distinguish equipment variants
  (crane tonnage) - resolved by the user adopting more specific naming going
  forward, no code change; (2) the actual matching bug, fixed here. Verified
  with 5 real test scenarios: exact-match regression, new-candidate-modal
  appearance, precise single-item checkbox selection, Cancel doing nothing,
  and false-positive avoidance (a coincidental number match, e.g. "10" in a
  crane weight vs a "10 GBPS" cable spec, correctly does not crowd out a
  genuinely relevant match once results are ranked by shared-word count
  rather than recency alone - a gap found and fixed during testing itself).

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
