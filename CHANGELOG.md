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
- Added: exporting a comparison now automatically copies the order's
  Comparisons folder path to the clipboard on success, reusing the exact
  same path-building logic as the existing "Copy Order Path" button - so
  reaching the exported file is now export, switch to File Explorer, paste,
  instead of the previous multi-step Copy Order Path -> switch app -> paste
  -> navigate into the Excel subfolder dance. Also moved the export's folder
  permission check to the very start of the function, before any workbook
  building begins, so that if a folder picker is genuinely needed (a stale
  or expired stored permission), it appears immediately after the click
  while the browser's "user activation" window is still fresh, rather than
  risking failure if it were needed deep inside the save process later.
  Two real issues were diagnosed and resolved during testing, neither of
  which turned out to be bugs in this feature itself: an open Excel file
  blocking the app from overwriting it (a user workflow issue, not a code
  issue - now more robust regardless thanks to the permission-timing fix
  above), and Live Server's file watcher reloading the page mid-export
  whenever a file changed inside the Orders/ folder, silently interrupting
  the in-progress clipboard write (fixed separately by configuring Live
  Server to ignore Orders/** - see accompanying config commit).
- Changed: the historical-match candidate rule (from the earlier
  review-and-confirm fallback in Import Historical Quotes) no longer
  requires a shared number. Numbers and words are now treated as one
  combined pool of significant tokens, and a match requires at least 2
  shared tokens total - 2 words, 2 numbers, or 1 of each. This fixes
  materials with no number in their name at all (e.g. "LAPTOP") which
  previously could never get any suggestions, since the old rule
  unconditionally required a number to overlap. This is a deliberately
  looser rule than before, accepted as a reasonable trade-off since this
  only ever produces a reviewable suggestion, never an automatic import -
  the user is always the one searching for and approving matches, so a few
  extra irrelevant suggestions cost little in exchange for covering
  non-numbered materials.
- Fixed: long material names were being silently cut off in two places -
  the item-name autocomplete dropdown (when typing a new order item) and
  the historical-quotes review modal - due to a flexbox layout quirk and
  ellipsis-based CSS truncation respectively, not because the underlying
  data was actually incomplete. Both now wrap onto multiple lines so the
  full name is always visible, which matters in particular when the
  distinguishing detail (an item code, a spec, a size) sits at the very
  end of a long name.
- Fixed and improved: Tracking tab delivered-status handling, found via a
  real bug report (an order marked Delivered/Completed in the Orders tab
  was still showing as "Late" in Tracking). Root cause: the Late/On-time
  badge was computed from an old per-item delivered-percentage field the
  user no longer uses, completely independent from the actual order-level
  Delivered flag shown in the Orders tab - the two could silently diverge.
  Removed the unused per-item percentage tracking entirely (filter logic,
  summary counters, and the per-row "X%" pill, which now reads "Delivered"
  / "Not delivered" instead) in favor of the order-level flag as the single
  source of truth everywhere. Also: (1) the stage filter (Ongoing/On
  hold/Completed/Canceled) now defaults to excluding On hold and Canceled
  orders from summary counts, since a canceled or on-hold order showing as
  "Late" was misleading - still toggleable back on for a quick look; (2)
  setting an order's stage to Completed now automatically marks it
  Delivered too, since the user's workflow never completes an order without
  delivering it - deliberately one-directional, so changing the stage away
  from Completed later does not silently un-mark delivered; (3) the
  previously separate "Only Late"/"Due soon" toggle buttons and the
  static, non-interactive summary chips have been unified into one row of
  five clickable, multi-select chips (Late/Due soon/Scheduled/No date/
  Delivered) - any combination can now be viewed together (e.g. Late +
  Due soon at once), which was not previously possible, and Scheduled/No
  date/Delivered can now be isolated at all for the first time. A new
  shared trackingCategoryOf() helper guarantees the chip counts and the
  actual filtering can never drift out of sync with each other going
  forward. Verified across many real test scenarios covering every piece
  of this change.
- Fixed/Removed: further cleanup of per-item delivery tracking, found while
  reviewing the Tracking tab after the previous delivered-status fix. (1)
  The Tracking tab row was showing two separate "Delivered" pills side by
  side - the new order-level pill from the previous fix, plus the existing
  order status pill (o.status), which could also independently say
  "Delivered" (or disagree, e.g. showing stale "Partially Delivered"). The
  duplicate pill has been removed; the remaining status pill is now
  color-coded instead (green for Delivered, info-blue for Ordered, plain
  for Draft). (2) recomputeStatus() no longer produces "Partially
  Delivered" at all - this state was computed from unused per-item
  deliveredQty/delivered fields; status is now purely Delivered (manual
  flag) > Ordered > Draft, consistent with the order-level flag being the
  single source of truth. (3) The same "Partially delivered" label was also
  removed from the Orders tab's status pills, which computed it
  independently from the same unused per-item fields. (4) The Tracking
  tab's "Items" quick-glance modal (opened via the Items button, for
  checking item details without leaving the tab) is now read-only -
  per-item delivered checkboxes and the Save button were removed, since
  that Save button recomputed and could silently overwrite the order-level
  Delivered flag from stale per-item checkbox state, a real risk that no
  longer exists once the checkboxes are gone. Verified across multiple
  real test scenarios covering all four changes.
- Changed: Materials tab overhaul, following a full walkthrough of what the
  user actually uses this tab for versus what it was showing. (1) The
  "Group materials" toggle and the old flat/grouped table duality have been
  removed entirely - the table now always shows exactly one row per
  material name, regardless of how many suppliers or orders it appeared
  in, with Best Supplier/Price/Currency for at-a-glance cheapest-price
  comparison. A new "View Orders" modal (replacing the old "Open order"
  button) lists every order that material has ever appeared in across all
  suppliers - date, supplier, price, currency - each with its own
  "Open Order" button to jump straight to that specific order. (2) The
  200-row hard cap (which was applied to raw quotes BEFORE grouping,
  silently showing fewer than 100 distinct materials once duplicates
  collapsed) is now a 100-default with a "Show More" button (+100 per
  click), correctly counting grouped materials instead. (3) The supplier
  filter dropdown has been replaced with a searchable autocomplete
  (reusing the same proven floating-overlay, word-token-matching component
  built for material name entry) - this fixes a real, confirmed bug where
  selecting "All suppliers" after choosing a specific supplier silently
  failed to reset the filter, caused by a <select> element's value being
  set before its <option> children existed. (4) Confirmed the existing
  material search bar already uses word-order-independent, multi-field
  token matching (comparable to or better than the matching built
  elsewhere in the app) - no changes needed there. (5) Clarified that
  "Only winners" is not redundant with the Best Supplier column (it
  changes "best" from "cheapest ever quoted" to "cheapest among suppliers
  actually selected") and kept it. (6) Removed a redundant summary pill
  row duplicating figures already shown in the KPI grid, and trimmed the
  KPI grid and per-supplier stats box to remove further duplicated
  figures, keeping only the per-supplier quote/win breakdown as genuinely
  unique information. (7) Kept the RFQ Intelligence box (best historical
  price, last winner, expected range for a searched material), confirmed
  useful after trying it live. Verified across many real test scenarios
  covering every piece of this redesign.
- Refactored (Phase 1 of code restructuring, done deliberately with an
  eventual online/multi-user version in mind - see project notes):
  consolidated duplicated file-writing logic in the storage helper layer.
  A structural audit found the same 4-line "get file handle, create
  writable, write, close" sequence independently reimplemented in four
  separate places (the comparison export's direct-write path, its
  showSaveFilePicker path setup, and saveComparisonXlsxToOrderFolder()),
  none of them using the saveBlobToPath() helper that already existed for
  exactly this purpose. Also found and removed ensurePath(), a function
  functionally identical to the more general getDirHandleFromSegments()
  (confirmed via code comparison and a full-file search showing exactly
  one caller). Introduced writeBlobToDir() as the new single source of
  truth for the write sequence; saveBlobToPath() and
  saveComparisonXlsxToOrderFolder() both now route through the shared
  helpers instead of duplicating the logic. This is a pure refactor with
  no intended behavior change, verified across multiple real export
  scenarios (initial save, overwrite/re-export). Note: after this change,
  the user observed the exported Excel file no longer opens in Windows
  Protected View, a longstanding minor annoyance - not confirmed as
  caused by this refactor (nothing here specifically targeted that
  behavior), but noted as a possible side effect worth continued
  observation rather than a confirmed fix.
- Refactored (Phase 2 of code restructuring): added clear, visible
  [SECTION] opening and closing markers around the File & Local Storage
  Helpers cluster (File System Access API handles, IndexedDB handle
  storage, the two lazy library loaders). This cluster was previously
  unlabeled, sitting in the middle of the broader UI Helpers section with
  only an informal one-line divider comment - now clearly delimited,
  matching the visible marker style already used for the file's other
  major sections. Purely additive comments; no code was moved and no
  behavior changed.
- Refactored (Phase 3 of code restructuring, final phase): a complete sweep
  of every File System Access API call in the file confirmed none exist
  outside the two known clusters (the storage-helper layer, and the
  comparison export path) - no hidden or forgotten direct-storage calls
  anywhere else. Found one genuine leftover from Phase 1: the export's
  stale-.xlsx cleanup loop (which keeps only one canonical file per order's
  Comparisons folder) was still calling removeEntry directly instead of
  going through a shared helper, missed in Phase 1 since that phase focused
  specifically on the write sequence rather than this cleanup loop. Added
  removeFileFromDir() as the single source of truth for this operation,
  mirroring the writeBlobToDir() pattern from Phase 1; deleteFileAtPath()
  and the cleanup loop both now route through it instead of duplicating the
  logic. Pure refactor, no intended behavior change - verified by
  deliberately triggering the cleanup scenario (a stray/renamed .xlsx file
  in an order's Comparisons folder) and confirming it's still correctly
  removed on export. This closes out the restructuring effort: storage
  logic is now fully consolidated behind a clean, well-isolated helper
  layer, genuinely easier to swap for cloud storage whenever the online/
  multi-user version is eventually built - without any speculative
  cloud-specific code written prematurely.
- Fixed: deleting an item from an order was silently corrupting comparison
  prices and winner selections for every material after the deleted one.
  Root cause: cmp.quotes and cmp.winners are keyed by array INDEX (idx),
  not by any stable per-item identifier - the "Remove" button only removed
  the item from order.items via splice(), never touching quotes/winners,
  so those stayed pinned to their old positions. After deleting a middle
  item, the material that shifted into that slot would display the price
  and winner that used to belong to the deleted item, cascading for every
  item after it (reported by the user as "completely breaks my comparison
  list"). Confirmed the app only ever appends new items at the end
  (never inserts mid-list or reorders), so this was scoped as a targeted
  fix to the deletion action specifically, rather than a deeper migration
  to stable per-item IDs (logged separately as a future architectural
  improvement, since a full migration would touch ~20+ call sites for a
  bug class that - given current usage patterns - only deletion can
  actually trigger today). The Remove button now shifts cmp.quotes and
  cmp.winners down to follow their correct material before removing it
  from the item list, and removes the now-duplicate trailing entry.
  Verified with a 3-item test order across three real scenarios: deleting
  from the start, middle, and end of the list, each confirming the
  remaining materials keep their own correct prices/winners, plus a
  post-deletion Excel export confirming the exported data matches what's
  shown in the app.
- Fixed: the comparison export's L1..LN supplier ranking (row 55) did not
  account for the missing-price substitution feature added in an earlier
  fix (a supplier with no quote for a material gets the lowest quoted
  price from another supplier copied in, shown in red text). Root cause:
  ranking was computed from the app's live cmp.quotes data via
  supplierTotalsRON(), which has no visibility into the substitution step -
  that step only writes directly into the Excel cell, never back into the
  app's own data (by design, so substituted prices never influence live
  in-app decision-making). The practical effect: a supplier missing a
  quote for one material had that material's cost silently excluded from
  their ranking total entirely, understating what they'd actually cost
  based on the document as printed. Ranking now sums the actual exported
  price cells (correctly including any substituted values) multiplied by
  quantity, converted through exchange rates, with transport/discount
  computed the same way as before (unaffected by substitution, which only
  applies to materials). Deliberately confirmed with the user: the app's
  own "Best total" badge should continue to reflect only real quotes, not
  substituted placeholders - this fix only changes the EXPORTED document's
  ranking to match what's actually printed there, and the two may now
  legitimately disagree in cases involving substitution, which is
  intentional. Verified across three scenarios: the exact reported case
  (a missing quote correctly counted in ranking), a full-real-quotes case
  confirming no change in behavior when nothing needed substituting, and a
  cross-check confirming the export still matches the app's own badge in
  the no-substitution case.
- Added: "Copy Table" button in the read-only Items view (Orders tab), for
  quickly preparing an RFQ materials list to send to suppliers by email.
  Copies a Material/Unit/Qty table (no price/supplier columns) to the
  clipboard. Rather than plain text, this writes a genuine rich HTML table
  with visible cell borders alongside a tab-separated plain-text fallback,
  using the browser's rich clipboard API (navigator.clipboard.write with
  ClipboardItem) - so pasting into Outlook, Word, or Gmail produces an
  actual bordered grid, matching the user's existing "copy a range from
  Excel, paste into Outlook" workflow, rather than unformatted text. Paste
  destinations that only accept plain text automatically get the
  tab-separated fallback instead. Material names are HTML-escaped to
  avoid broken output for any name containing &, <, or > characters.
  Design decided collaboratively before building: considered and rejected
  two simpler plain-text-only formats (a manually-aligned column table,
  and a single-line-per-material format) once the user clarified their
  actual habit was pasting a genuine formatted Excel range, not plain
  text. Verified: the button and copy action itself, a real paste into
  Outlook Classic showing a correctly bordered table, a real paste into
  Gmail (not originally part of the plan, tried by the user and also
  confirmed working), a plain-text-only destination correctly falling
  back to readable tab-separated text, and correct display of a material
  name containing a special character.
- Fixed: completed the per-item delivered-status cleanup across the two
  remaining views found to still use it (following up on the inconsistency
  noted while building the RFQ item table copy feature). (1) The read-only
  Items view (Orders tab) now shows a single "Order status: Delivered/Not
  delivered" line instead of a per-item Delivered column that always
  showed the same value repeated on every row, and its footer now shows
  one combined total instead of a meaningless Delivered/Pending split.
  (2) The Order editor's own item-editing table: per explicit user
  decision, removed the entire Delivered Qty/Remaining/Status apparatus
  (not just the redundant boolean fields) - the underlying quantity-based
  partial-delivery tracking was confirmed unused, since the user only
  relies on the single order-level Delivered flag. (3) The Suppliers tab's
  per-supplier order history card: removed the same stale per-item
  Delivered column, which duplicated the card's already-correct
  order-level "Status: ..." line shown above the table - no new status
  indicator needed there, since a correct one already existed. Also
  investigated and ruled out a suspected orphaned CSS cleanup (.row-
  delivered/.row-pending never actually had style rules defined for them
  in the stylesheet - applying those classes was already a no-op before
  today, not something that became dead code as a result of this fix).
  Verified across all three views plus a general save/reload regression
  check.
- Noted but not yet acted on: while building this feature, found that the
  same Items view's existing "Delivered: Yes/No" column and Delivered/
  Pending totals still read the old per-item `it.delivered` field, rather
  than the order-level statusFlags.delivered flag established as the
  single source of truth earlier this session (Fix 12/13). Logged as a
  known inconsistency to revisit, not fixed as part of this feature.

## [v1.2.2.39] - Baseline
- First version tracked in git.
- Established as the current production baseline (commit f31ed4b on main).
- No functional changes from the prior untracked v1.2.2.39 — this entry marks
  the start of proper version history, not a code change.
