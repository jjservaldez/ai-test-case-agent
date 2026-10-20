# v1 vs v2

v1: 1 run, 40 test cases. v2: 2 runs (44 and 42 test cases); per-case scores below are from v2 run 1, reviewed by Jose Serna with a second-pass peer review. The user input (story, ACs, rooms) was identical in both versions; only the instructions changed.

## Per-case metrics

| Metric | v1 | v2 (run 1) | Change |
|---|---|---|---|
| Test cases | 40 | 44 | +4 |
| Valid | 40 / 40 (100%) | 44 / 44 (100%) | = |
| Executable | 40 / 40, 4 with clarity issues | 42 / 44 (95.5%) | See note 1 |
| Hallucinations (unflagged assumptions) | 0 / 40 | 1 / 44 (2.3%), TC-026 | See note 2 |
| Duplicates | 3 / 40 (7.5%) | 0 / 44 | Better |
| Category OK | 40 / 40 | 44 / 44 | = |
| Expected result strong enough | 35 / 40 (87.5%) | 43 / 44 (97.7%) | Better |
| AC coverage | 12 / 12 | 12 / 12 | = |

## Bug detection (defects found in exploration)

| Bug | v1 | v2 run 1 | v2 run 2 |
|---|---|---|---|
| BUG-01 Rejected booking crashes the page | Reached, weak oracle | Detected | Detected |
| BUG-02 Negative nights and total | Reached, not checked | Detected | Detected |
| BUG-03 Null dates, £NaN | Partial | Detected via URL | Detected via URL |
| BUG-04 Names accept digits only | Not covered | Reached, flagged as unspecified (AC gap) | Not covered |
| BUG-05 Errors do not name the field | Detected | Detected | Detected |
| **Total detected** | **1 / 5** | **4 / 5** | **4 / 5** |

## Notes

1. **Executable.** The v1 review was more lenient (see L-09); its four clarity issues were noted but scored Y. The two v2 failures are concrete: TC-014 describes test data instead of giving it ("30 characters"), and TC-041 gives HTML-escaped injection data (`&lt;script&gt;`), so the test would not exercise injection. The escaping issue appears in both v2 runs.
2. **Hallucinations.** v2 asks the model to "state where the tester must look". Without a description of the UI, it named verification points it could not know and didn't flag them. TC-001 and TC-004 (calendar marks the selected dates) were first scored as hallucinations, then reversed: a later recording showed the calendar does mark selected nights. TC-026 ("booked dates are shown as unavailable") was confirmed as a hallucination: a later recording showed booked dates are not marked on the calendar. In all three cases the model guessed; v3 adds a UI map so it doesn't have to.
3. **Consistency.** About 32 scenarios appear in both v2 runs. Detection is identical for 4 of 5 bugs; the digits-only name scenario appeared in only one run.

## Summary

v2 raised known-bug detection from 1/5 to 4/5 in both runs, raised strong oracles from 87.5% to 97.7% and removed duplicates. It also guessed at UI details it had no way to know (1 confirmed unflagged assumption after re-checking the app) and produced an output-formatting defect in test data (escaped HTML).
