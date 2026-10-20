# Final test suite

The test suite I would hand to a team: the v3 run 1 output after my review, with the peer-review fixes applied and regression cases added for the bugs no prompt version covered.

**Full suite (52 cases):** [final-test-suite.csv](final-test-suite.csv) (GitHub shows it as a searchable table). Columns: the v3 fields plus `Source`, `Change from v3` and `Related bug`.

## How it was built

| | Cases |
|---|---|
| Kept from v3 run 1 unchanged | 36 |
| Kept from v3 run 1 and edited | 12 |
| Taken from v2 (scenario v3 dropped) | 1 |
| Added by me as regression cases | 3 |
| **Total** | **52** |

## What I changed and why

| TC | Change |
|---|---|
| TC-001 | Step 6 no longer depends on the run date. |
| TC-017 | Expected result changed from "accepted" to "rejected", following the product question in BUG-04. |
| TC-019 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-020 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-023 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-024 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-027 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-028 | Expected result now requires the message to name the field (would catch BUG-05). |
| TC-029 | Expected result now requires each message to name its field (BUG-05). |
| TC-033 | Category changed from Security to Negative (concurrency, not security). |
| TC-034 | Category changed from Security to Negative (concurrency, not security). |
| TC-036 | Price oracle fixed: the risk for 0 nights is a fees-only total (£40), not £140. |

## What I added

| TC | Title | Source | Why |
|---|---|---|---|
| TC-049 | Clicking a date in the room calendar keeps price, calendar, URL and booking consistent | Added by reviewer (regression) | New. No prompt version generated it. |
| TC-050 | Each error message names the field it refers to | Added by reviewer (regression) | New. v3 had no case that checks messages name the field. |
| TC-051 | Same dates on a different room can be booked | v2 TC-028 | Taken from v2; v3 did not generate it. |
| TC-052 | "Amenities" navigation link goes to the amenities section | Added by reviewer (regression) | New. |

## Bug traceability

Cases that cover each known bug, on the path where it was reproduced or a closely related one. Once a bug is fixed, these become its regression tests.

| Bug | Summary | Cases |
|---|---|---|
| [BUG-01](../bugs/BUG-01.md) | Booking panel crashes when the server rejects a booking | TC-031, TC-032, TC-033, TC-035, TC-036, TC-048 |
| [BUG-02](../bugs/BUG-02.md) | Invalid date range gives negative or zero-night prices | TC-035, TC-036, TC-048 |
| [BUG-03](../bugs/BUG-03.md) | "View Details" loses the dates | TC-038 |
| [BUG-04](../bugs/BUG-04.md) | Names made only of digits are accepted | TC-017 |
| [BUG-05](../bugs/BUG-05.md) | Error messages don't name the field | TC-019, TC-020, TC-023, TC-024, TC-027, TC-028, TC-029, TC-050 |
| [BUG-06](../bugs/BUG-06.md) | Clicking a calendar date lowers the price, but the original range is booked | TC-049 |
| [BUG-07](../bugs/BUG-07.md) | "Amenities" link goes nowhere | TC-052 |

## Acceptance criteria coverage

| AC | Cases |
|---|---|
| AC1 | 11: TC-001, TC-002, TC-003, TC-011, TC-012, TC-013, TC-014, TC-037, TC-038, TC-045, TC-049 |
| AC2 | 9: TC-001, TC-002, TC-003, TC-011, TC-012, TC-013, TC-038, TC-048, TC-049 |
| AC3 | 3: TC-001, TC-004, TC-005 |
| AC4 | 13: TC-006, TC-007, TC-008, TC-016, TC-017, TC-018, TC-019, TC-020, TC-021, TC-022, TC-029, TC-030, TC-046 |
| AC5 | 10: TC-006, TC-007, TC-008, TC-017, TC-023, TC-024, TC-029, TC-030, TC-046, TC-047 |
| AC6 | 6: TC-006, TC-010, TC-025, TC-026, TC-029, TC-030 |
| AC7 | 7: TC-006, TC-007, TC-009, TC-027, TC-028, TC-029, TC-030 |
| AC8 | 17: TC-018, TC-019, TC-020, TC-021, TC-022, TC-023, TC-024, TC-025, TC-026, TC-027, TC-028, TC-029, TC-030, TC-031, TC-032, TC-045, TC-050 |
| AC9 | 22: TC-001, TC-002, TC-006, TC-007, TC-008, TC-009, TC-010, TC-011, TC-012, TC-013, TC-015, TC-016, TC-030, TC-033, TC-034, TC-042, TC-043, TC-044, TC-046, TC-047, TC-049, TC-051 |
| AC10 | 8: TC-015, TC-031, TC-032, TC-033, TC-034, TC-042, TC-044, TC-051 |
| AC11 | 4: TC-035, TC-036, TC-037, TC-048 |
| AC12 | 5: TC-014, TC-038, TC-039, TC-040, TC-041 |

## Known limits

- Dates are fixed (2026–2027). On a later run, shift them forward and recompute the expected totals.
- The app is a shared public demo: if a precondition says a room is free and it is not, move both dates by a week.
- TC-008, TC-034 and TC-046 note checks that need the Admin panel, which I had no credentials for.
- TC-017 depends on a product decision (BUG-04) and is written for the rule I recommend, not the current ACs.
