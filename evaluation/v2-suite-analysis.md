# v2 suite-level analysis (runs 1 and 2)

Prompt: `prompts/v2.md` · Two runs, each in a new chat with no project context.
Run 1 = 44 test cases (`runs/v2-run1.md`) · Run 2 = 42 test cases (`runs/v2-run2.md`).
Per-case rubric scores for run 1 are pending human review.

## Bug detection: v1 vs v2

| Bug | v1 | v2 run 1 | v2 run 2 |
|---|---|---|---|
| BUG-01 Rejected booking crashes the page | Reached, weak oracle | **Detected** (TC-026, TC-027, TC-029, TC-040: "clear message, no error page, page usable") | **Detected** (TC-025, TC-026, TC-029, TC-038) |
| BUG-02 Negative nights and total | Reached, price not checked | **Detected** (TC-029, TC-031: "never zero or negative nights or totals") | **Detected** (TC-029) |
| BUG-03 Null dates, £NaN | Partial | **Detected via URL** (TC-035 malformed parameters; TC-034 no parameters). The Similar Rooms path is still unknown | **Detected via URL** (TC-033 explicitly: "price summary not showing NaN") |
| BUG-04 Names accept digits only | Not covered | Reached (TC-024 "12345"), oracle accepts either outcome because no AC defines it | **Not covered** |
| BUG-05 Errors do not name the field | Detected | **Detected** (TC-008, TC-009, TC-043) | **Detected** (TC-007, TC-010, TC-017) |
| **Detected** | **1 / 5** | **4 / 5** | **4 / 5** |

BUG-03 is detected through the symptom (bad date parameters), not through the real user path (Similar Rooms → View Details). The agent can't know that path without a description of the UI. Candidate for v3 (UI map).

BUG-04 remains an AC gap. Run 1 handled it correctly: it tested the scenario and flagged that the expected behavior is unspecified ("record behavior"), which is the right call for a requirement gap.

## Coverage of previously omitted scenarios

| Scenario (omitted by v1) | Run 1 | Run 2 |
|---|---|---|
| Past dates | Yes (TC-032) | Yes (TC-030) |
| Accents / apostrophes / hyphens | Yes (TC-022) | Yes (TC-021) |
| Emoji | Yes (TC-024) | Yes (TC-022) |
| Phone with country code / spaces | Yes (TC-020) | Yes (TC-023) |
| Browser Back after confirmation | Yes (TC-038) | Yes (TC-035) |
| Two tabs, same dates | Yes (TC-040) | Yes (TC-038) |
| Booking today | Yes (TC-033) | Yes (TC-042) |
| Booking today after the check-in window | No | No |

New ideas not prompted by the checklist: same dates in a different room (run 1), day above 12 to expose day/month swaps (run 1), month and year boundaries (both), adjacent range on the check-out day (run 2), several invalid fields at once (run 2).

## Consistency between runs

About 32 scenarios appear in both runs (roughly three quarters of each suite). Bug detection is identical for 4 of 5 bugs. The difference that matters: the digits-only name scenario (BUG-04) appeared in run 1 and not in run 2. A single run would have given a different picture depending on which one was taken.

## Issues seen in both runs (to confirm in the per-case review)

- Script injection test data is HTML-escaped: `&lt;script&gt;alert(1)&lt;/script&gt;`. A tester copying it would type the escaped text, so the test would not exercise injection at all.
- Some expected results assume UI behavior that isn't in the ACs and isn't flagged as an assumption, e.g. "the calendar highlights the selected dates" or "the dates are shown as unavailable". (Correction, 2026-10-19: the calendar does mark selected nights with "Sel…"; it always opens on the current month. The "dates are shown as unavailable" claim was checked and is false: booked dates are not marked.)
- Some test data is described instead of given ("Lastname: 31 characters", "Lastname: valid").
