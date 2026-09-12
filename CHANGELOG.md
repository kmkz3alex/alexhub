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
- Added: removing a supplier from a comparison (the X button on a supplier
  slot) now also deletes that supplier's offer file(s) from the current
  order's folder, instead of leaving orphaned PDFs to be cleaned up manually
  one at a time. Reuses the existing offer-to-supplier matching logic already
  used by the historical import feature, and the app's established safe
  file-delete pattern (handles the file already being missing gracefully).
  Deletion is strictly scoped to the current order - offer files are always
  order-specific copies (even ones auto-attached via historical import), so
  removing a supplier from one order's comparison never touches their offer
  files on any other order. The confirmation dialog now clearly separates the
  file-deletion warning (marked with a warning symbol, since it's the more
  consequential part) from the existing quotes/winners warning, and correctly
  fires even when there's a file but no quotes/winners yet - previously that
  specific case would have shown no warning at all. Verified with 4 real test
  scenarios: Cancel leaves everything intact, confirmed deletion removes the
  file from the app, the Files view, AND real disk, deletion is confirmed
  scoped to only the current order, and no file-deletion warning appears when
  no offer file exists.
- Added: material name autocomplete (when adding order items) rebuilt from
  scratch, replacing the native browser <datalist> it previously relied on.
  The native version's filtering behavior was inconsistent and out of the
  app's control - e.g. typing "crane 100" would show some matching historical
  materials but silently omit others depending on word order, with no way to
  fix this in a native datalist. The new dropdown is fully custom-built
  (mirroring the existing supplier-autocomplete pattern already used
  elsewhere in the app), matches every typed word independently regardless
  of order or adjacency (so "rental crane 100" and "crane rental 100" both
  correctly find "CRANE RENTAL 100TON"), and floats as a viewport-anchored
  overlay so it's never clipped by the table even for item rows near the
  bottom of the page - it also intelligently opens upward when there's more
  room above than below. Full keyboard navigation (arrow keys, Enter,
  Escape) included, matching the existing supplier-selector's UX. Results
  capped at 20. Verified across multiple rounds of real testing, including
  catching and fixing two positioning issues (clipped by the table's own
  boundaries, then requiring word-order-independent matching) before the
  final version was confirmed working correctly in all scenarios.
- Added: comparison export now automates the manual Excel touch-up step that
  was previously done by hand after every single export (identified via a
  full workflow walkthrough with the user - this turned out to be the
  biggest time sink in their whole process, entirely outside the app).
  Four pieces: (1) DATE:/Request Number: labels dynamically position above
  whichever supplier column is actually last (previously hardcoded for
  exactly 3 suppliers), with explicit Calibri 20pt formatting - bold labels,
  normal-weight values. DATE auto-fills with today's date; Request Number
  auto-fills from the order's RFQ, or stays empty for ad-hoc orders. (2) Row
  55 auto-generates L1..LN supplier rankings by total WITHOUT VAT (so
  tax-exempt suppliers aren't misrepresented), reusing the exact calculation
  already powering the "Best total" badge in the app. (3) Winner
  highlighting: a single overall winner gets their header name cell colored
  green; winners split across materials get only their specific winning
  price cell colored, not the total or the whole column. (4) Missing prices
  (0/no quote) are auto-filled with the lowest quoted price among other
  suppliers for that material, correctly converted through the comparison's
  exchange rates into the receiving supplier's own currency, with the
  substituted number's text colored red to clearly mark it as borrowed
  rather than a genuine quote - this only affects the exported file, never
  the app's actual stored data. Two real bugs were caught and fixed during
  testing rather than after: an ExcelJS shared-style-object mutation bug
  where coloring one cell bled into the entire header row (fixed by applying
  the same defensive style-cloning pattern already used elsewhere in this
  function), and a currency bug where a borrowed price was copied as a raw
  number instead of being converted through the exchange rate first (fixed
  by normalizing all prices to RON for comparison, then converting the
  chosen value back into the receiving supplier's own currency). Verified
  across many real test scenarios covering every piece and both bugs' fixes.

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
