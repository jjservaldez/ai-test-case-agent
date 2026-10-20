# v3 suite-level analysis (runs 1 and 2)

Prompt: `prompts/v3.md` (v2 + UI map + literal test data rule) · Two runs, each in a new chat with no project context.
Run 1 = 48 test cases (`runs/v3-run1.md`) · Run 2 = 46 test cases (`runs/v3-run2.md`).
Per-case rubric scores for run 1 are pending human review.

## Bug detection: v1 vs v2 vs v3

The first five bugs are the original reference set. BUG-06 (calendar click changes the price, found on 2026-10-19) is added as a new check; BUG-07 (Amenities link) is outside the booking flow and not scored.

| Bug | v1 | v2 (runs 1 / 2) | v3 run 1 | v3 run 2 |
|---|---|---|---|---|
| BUG-01 Rejected booking crashes the page | Reached, weak oracle | Detected / Detected | **Detected** (TC-031, TC-032, TC-033, TC-036, TC-048: "clear message, no error page") | **Detected** (TC-025, TC-026, TC-030, TC-032, TC-033) |
| BUG-02 Invalid range not validated (negative or zero nights) | Reached, not checked | Detected / Detected (negative only) | **Detected** (TC-035, TC-048). TC-036 covers 0 nights but its price oracle is off (see below) | **Detected, including 0 nights** (TC-032, TC-033: "does not present 0 nights with a £40 total as bookable") |
| BUG-03 "View Details" loses the dates | Partial | Via URL / Via URL | **Detected through the real path** (TC-038: Similar Rooms → View Details) | **Detected through the real path** (TC-035) |
| BUG-04 Names accept digits only | Not covered | Reached (AC gap) / Not covered | Reached, flagged for the PO (TC-017) | Not covered |
| BUG-05 Errors do not name the field | Detected | Detected / Detected | **Partial**: cases expect "one bullet about Firstname length" or "errors for the four fields", which "size must be between 3 and 18" could pass | **Partial**: same wording (TC-011, TC-012) |
| BUG-06 Calendar click changes the price | n/a | Not covered | **Not covered**: TC-001 and TC-011 check the calendar display, no case clicks a date | **Not covered**: TC-046 navigates the calendar (Next/Back/Today), no date click |
| **Original 5 detected** | **1 / 5** | **4 / 5 · 4 / 5** | **3 / 5 + 1 partial** | **3 / 5 + 1 partial** |

## What improved

- **BUG-03 through the real user path** in both runs. The UI map told the model the Similar Rooms section exists; it built the test on its own. This is the change v3 was designed to produce.
- **Zero-night price check (BUG-02 variant)** explicitly in run 2: "does not present 0 nights with a £40 total as bookable".
- **Test data is literal**: `<script>alert(1)</script>` is unescaped in both runs; no described values found on first read.
- **Expected results are concrete**: exact URLs, prices, confirmation text, where the error box appears.
- **Self-contained preconditions**: cases that need an existing booking give the URL to create it (the v2 TC-026 dependency issue).
- **Shared-demo awareness**: far-future fixed dates per case, and notes like "if the dates are taken, shift +7 days".
- **Check-in window** finally appears (run 1 TC-002 note: booking today after 8:00 PM).
- **New scenarios**: non-existent room ID, dates changed without clicking Check Availability, leap year (run 2), 14-night stay (run 2), same dates in a different room (run 2).

## What regressed or is still missing

- **BUG-05 regressed from detected to partial.** v2 had an explicit UX case ("messages identify the field"). v3's negative cases describe the red box content as "a bullet about Firstname length", which a message without the field name can satisfy. Likely side effect of the UI map describing the error box as "a bulleted list", so the model matched that shape instead of asking for the field name.
- **BUG-06 not reached in either run.** The UI map describes the calendar's display (Today/Back/Next, "Selected" bar) but not that dates can be clicked, so no case interacts with the dates.
- **BUG-04 still depends on the run** (appears in run 1 only), same pattern as v2.
- **Run 1 TC-036 oracle is off**: for a 0-night Single stay it says "Price Summary never shows 1 or more nights or a total of £140". The actual risk is 0 nights with a £40 total, which this oracle allows.

## Consistency between runs

Both runs share the core: end-to-end, boundaries, invalid email/phone, accents, international phone, duplicate and overlapping dates, reversed and same-day dates, past dates, View Details, URL without/with bad parameters, non-existent room, two tabs, double click, Back, refresh, Return home, XSS, SQL. Bug detection is identical across runs for BUG-01, 02, 03, 05 and 06; BUG-04 differs again.

## To confirm in the per-case review

- Hallucinations: the UI map should have reduced them; check that no case still assumes UI behavior outside the map without flagging it.
- Executability: some steps depend on the execution date ("click Next until March 2027 (5 clicks)" is only right if run in October 2026).
- Category: run 1 labels two tabs and double click as Security; arguably Negative/concurrency.
