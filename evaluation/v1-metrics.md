# v1 evaluation (run 1)

Prompt: `prompts/v1.md` · Run in a clean chat with no project context · 40 test cases generated · Reviewed by Jose Serna, with a second-pass peer review.

## Rubric definitions

| Metric | Question asked for each test case |
|---|---|
| Valid | Does it test something that applies to this app, according to the ACs? |
| Executable | Can a tester run it without asking for clarification? |
| Hallucination | Did it invent a feature, field, message or rule without flagging it as an assumption? |
| Duplicate | Does another case test the same thing? |
| Category OK | Is the assigned category correct? |
| Expected result strong enough | Imagine a realistic bug in what the case tests. If a tester follows the expected result literally, would they mark Fail? |

## Per-case results

| Metric | Result |
|---|---|
| Valid | 40 / 40 (100%) |
| Executable | 40 / 40, 4 with clarity issues (TC-002, TC-029, TC-032, TC-036) |
| Hallucinations | 0 / 40 |
| Duplicates | 3 / 40 (7.5%): TC-007 ≈ TC-001, TC-029 = TC-002, TC-040 overlaps TC-003/TC-004 |
| Category OK | 40 / 40 |
| Expected result strong enough | 35 / 40 (87.5%). Weak: TC-014, TC-015, TC-016, TC-017, TC-037 |
| AC coverage | 12 / 12 ACs covered by at least one case |

## Bug detection (against defects found in exploration)

| Bug | Severity | v1 result | Classification |
|---|---|---|---|
| BUG-01 Rejected booking crashes the page | High | Scenario reached (TC-014/015/016), weak oracle: would pass | Agent miss (oracle) |
| BUG-02 Negative nights and total | High | Scenario reached (TC-016), price not checked: would pass | Agent miss (oracle) |
| BUG-03 "View Details" sends null dates | Medium | Partial: TC-030 (URL without dates) is close; the Similar Rooms path is unknown to the agent | Context gap (UI not described) |
| BUG-04 Names accept digits only | Medium | Not covered | AC gap (no AC says letters only) |
| BUG-05 Errors do not name the field | Low | Caught (TC-038; TC-008/TC-012 can also expose it) | Caught |

**Detected with the v1 expected results as written: 1 / 5.** Scenario reached for 4 / 5.

## Scenarios the agent did not generate

Past dates, names with accents or apostrophes (José, O'Brien), emoji, phone with country code or spaces (+52 33 ...), browser Back after a confirmed booking, two tabs booking the same dates, booking today after the check-in window.

## Main finding

v1 is good at reaching the right scenarios and correct on boundary values and price arithmetic, with no hallucinations. Its weakness is the oracle: expected results describe the main outcome ("booking is not created") but not everything that must hold, so 3 of 5 known bugs would pass unnoticed.

## Review process note

The first manual pass scored 40/40 on every per-case metric. The peer review found the duplicates, the clarity issues and the 5 weak oracles. See `lessons-learned.md`, L-09.
