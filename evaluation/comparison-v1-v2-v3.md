# Prompt comparison: v1 vs v2 vs v3

App: Shady Meadows B&B booking flow (https://automationintesting.online) · Input: one user story and 12 acceptance criteria derived from exploratory testing · Each run in a new chat with no project context · Per-case scores from run 1 of each version, reviewed by Jose Serna with a second-pass peer review.

| Version | What changed | Runs |
|---|---|---|
| v1 | Baseline: senior QA role, flag assumptions, categories, table format | 1 (40 cases) |
| v2 | + oracle rules (message, no error page, page usable, derived values consistent), real-user scenario checklist, no duplicates, explicit navigation | 2 (44, 42 cases) |
| v3 | + UI map written from exploration (describes what exists, not what fails), literal test data rule | 2 (48, 46 cases) |

## Per-case metrics (run 1 of each version)

| Metric | v1 | v2 | v3 |
|---|---|---|---|
| Test cases | 40 | 44 | 48 |
| Valid | 100% | 100% | 100% |
| Executable | 100%* | 95.5% | 100% |
| Unflagged assumptions (hallucinations) | 0 | 1 | **0** |
| Duplicates | 7.5% | 0% | 0% |
| Category OK | 100% | 100% | 100% |
| Expected result strong enough | 87.5% | 97.7% | 95.8% |

\* The v1 review was more lenient (lessons learned, L-09); four clarity issues were noted but scored as executable.

The two "not strong enough" cases in v3 are TC-017 (digits-only names expected to be accepted, because no AC says otherwise: a requirement gap) and TC-036 (the 0-night price check guards against £140 instead of the actual £40).

## Known-bug detection

Detection here is **analytical**: each case was read against each bug to decide whether a tester following it literally would mark Fail. The cases were not executed against the app. Execution results will replace this column once the final suite is run.

| Bug | v1 | v2 (run 1 / run 2) | v3 (run 1 / run 2) |
|---|---|---|---|
| BUG-01 Rejected booking crashes the page | Reached, weak oracle | ✅ / ✅ | ✅ / ✅ |
| BUG-02 Invalid date range not validated | Reached, not checked | ✅ / ✅ | ✅ / ✅ (run 2 checks the 0-night £40 case) |
| BUG-03 "View Details" loses the dates | Partial | ✅ via URL / ✅ via URL | ✅ **real path** / ✅ **real path** |
| BUG-04 Names accept digits only | ❌ | Reached (AC gap) / ❌ | Reached (AC gap) / ❌ |
| BUG-05 Errors don't name the field | ✅ | ✅ / ✅ | Partial / Partial |
| BUG-06 Calendar click changes the price | n/a | ❌ / ❌ | ❌ / ❌ |
| **Original 5 detected** | **1** | **4 / 4** | **3 + 1 partial / 3 + 1 partial** |

## What each version taught

- **v1 → v2:** the agent already reached the right scenarios; its weakness was the oracle. Telling it what "correct" must include (message, no error page, consistent prices) took detection from 1 to 4 of 5.
- **v2 → v3:** asking it to say where to verify, without describing the UI, made it guess. A UI map removed the guesses (1 → 0 unflagged assumptions) and let it reach BUG-03 through the real user path. It also narrowed one oracle: error checks became "a bullet about Firstname length", which no longer requires the message to name the field (BUG-05).
- **Across all versions:** BUG-04 appears in only one of two runs each time, so single runs are not reliable. BUG-06 was never reached; it was found by the tester while verifying a fact for the UI map.

## Details behind the v1 → v2 change

- **Executable (v2, 95.5%).** TC-014 describes test data instead of giving it ("30 characters"), and TC-041 gives HTML-escaped injection data (`&lt;script&gt;`), so the test would not exercise injection. The escaping issue appeared in both v2 runs; v3's literal-data rule fixed it.
- **Hallucinations (v2, 1).** v2 asks the model to state where the tester must look, but gave it no description of the UI. TC-001 and TC-004 (the calendar marks the selected dates) were first scored as hallucinations and then reversed after re-checking the app. TC-026 ("booked dates are shown as unavailable") was confirmed: booked dates are not marked on the calendar. In all three cases the model guessed, which is why v3 adds a UI map.
- **Consistency (v2).** About 32 scenarios appear in both runs. Detection is identical for 4 of 5 bugs; the digits-only name scenario appeared in only one run.

## Where the bugs came from

| Source | Bugs |
|---|---|
| Exploratory testing (Jose) | BUG-01 to BUG-05 |
| A v2 test case (TC-030, same-day dates) | Third reproduction path of BUG-01 and the 0-night variant of BUG-02 |
| Verifying the UI map against the app (Jose) | BUG-06 (High), BUG-07 (Low, outside the booking flow) |

## Known limitations

- Two runs per version; more runs would give a firmer consistency number.
- One reviewer plus an AI peer review, not two independent human reviewers.
- The reference bugs come from the same person who reviews the output.
- Public demo: other users' bookings can change availability during a session (see OBS-01).
