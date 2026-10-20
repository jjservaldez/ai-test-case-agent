# Lessons Learned

Running log of lessons from building and evaluating the AI test-case agent. Each entry: what happened, the lesson, and how it changes the work.

## Exploration and requirements

### L-01 · Acceptance criteria must come from the real app, not assumptions
**What happened:** The first instinct was to write ACs for the Shady Meadows booking form from what a booking form "usually" does.
**Lesson:** ACs invented without looking at the app make the agent generate tests for rules that don't exist, and the evaluation becomes meaningless.
**Applied:** ACs were derived from five recorded exploratory sessions, and the README states they come from exploration, not official requirements.

### L-02 · In exploration, submit after every change
**What happened:** In one session, invalid names ("Jose123123", symbols) and "test@test" were typed but corrected before clicking "Reserve Now". Only the last state was submitted, so we couldn't tell whether the app accepts those values.
**Lesson:** An input you type but never submit tells you nothing about validation.
**Applied:** Follow-up sessions submitted after each invalid value.

### L-03 · Re-test state-dependent bugs immediately
**What happened:** A second booking of the same dates in the Double room was confirmed 10 minutes after the first, which looked like a double-booking bug. The demo resets its data periodically. When retested right after a confirmed booking (Suite), the duplicate was blocked.
**Lesson:** On shared or demo environments, a state-dependent "bug" can be a data reset. Reproduce it with no time gap before reporting.
**Applied:** The double-booking finding was withdrawn and replaced by the real issue (the page crashes instead of showing a message).

### L-04 · Update conclusions when new evidence appears
**What happened:** Email was first recorded as "only checks not empty" because "test@test" was accepted. A later session showed "test.com" is rejected with "must be a well-formed email address".
**Lesson:** One accepted value doesn't prove there is no validation. The rule was permissive, not missing.
**Applied:** AC6 and the findings were corrected.

### L-05 · Group symptoms by likely root cause
**What happened:** The page crashed in two different flows: booking dates already taken, and booking with Check Out before Check In.
**Lesson:** Two crashes with the same pattern (backend rejects, frontend doesn't handle the error) are one bug with two reproduction paths. One report with both paths is stronger than two separate reports.
**Applied:** Merged into BUG-01.

## Evaluating the agent

### L-06 · Run the agent in a clean context
**What happened:** The chat where the exploration happened already knew every bug.
**Lesson:** Generating test cases in that same context would leak the answers and inflate the results.
**Applied:** Prompt v1 was run in a new chat with no project context.

### L-07 · The agent finds the right scenarios but writes weak oracles
**What happened:** v1 generated cases that reach 3 of the 5 known bugs (TC-014, TC-015, TC-016), but their expected result was only "the booking is not created". A page crash also produces no booking, so a tester following the case literally would mark Pass.
**Lesson:** Coverage of scenarios is not the same as the ability to detect defects. The expected result (the test oracle) decides whether a bug is caught.
**Applied:** Added the metric "Expected result strong enough" and planned v2 to require checks on user-facing messages, no error page, and price consistency.

### L-08 · Separate agent misses from requirement gaps
**What happened:** No v1 case caught BUG-04 (names accept digits only), but no AC said names must be letters.
**Lesson:** A miss can come from the agent or from incomplete requirements. Classifying each miss avoids blaming the agent for a gap in the input.
**Applied:** Misses are tagged as "agent miss" or "AC gap" in the evaluation.

### L-09 · Polished AI output lowers the reviewer's guard
**What happened:** The first manual review of the 40 v1 cases scored 40/40 on validity, executability, no hallucination, no duplicates and correct category. A peer review then found two duplicates (TC-029 = TC-002, TC-007 ≈ TC-001), ambiguous steps (TC-002, TC-029, TC-032, TC-036) and 5 weak oracles.
**Lesson:** Good formatting, fluent English and correct arithmetic make AI output look finished. A perfect score across the board is a signal to review again, not a result.
**Applied:** Every review now gets a second pass that looks specifically for duplicates, ambiguity and weak oracles.

### L-10 · Define custom metrics explicitly
**What happened:** "Expected result strong enough" was unclear until it was tied to a concrete question.
**Lesson:** A metric you invented needs a written definition, or reviewers will score it differently.
**Applied:** Definition: *imagine a realistic bug in what the case tests; if a tester follows the expected result literally, would they mark Fail?* Based on the test-oracle concept and the logic of mutation testing.

### L-13 · Improve the prompt with general heuristics, not the known answers
**What happened:** While writing v2, the easy fix was to tell the agent "check that the price is never negative" or "test the Similar Rooms link", which would catch BUG-02 and BUG-03 directly.
**Lesson:** Hints that describe the known bugs inflate the score on this app and teach nothing that transfers to the next one. That is overfitting the prompt to the test set.
**Applied:** v2 uses generic heuristics ("derived values must stay consistent with the inputs", "for rejected actions, check message, no error page, page still usable").

### L-14 · Change one variable per prompt version
**What happened:** v2 could have added both new instructions and a map of the UI (button names, the Similar Rooms section, URL parameters).
**Lesson:** If two things change at once, you can't tell which one moved the results.
**Applied:** v2 changes only the instructions; the user input is identical to v1. A UI map, if needed, goes in v3.

### L-15 · Keep documentation out of the prompt file
**What happened:** The v2 prompt file started with a "Changes from v1" note that said the hints were "not descriptions of the known bugs". Pasting the whole file would have told the model that known bugs exist. Caught by Jose before running it.
**Lesson:** Anything in the file you paste is part of the prompt. Notes for humans can leak information or bias the model.
**Applied:** Prompt files now hold only the text sent to the model; version notes moved to `prompts/CHANGELOG.md`.

### L-16 · One run is not a measurement
**What happened:** v2 was run twice with the identical prompt. Both runs detected 4 of 5 known bugs, but only run 1 tested names with digits only (BUG-04 scenario). Run 2 never generated it.
**Lesson:** LLM output varies between runs. A result from a single run can over- or under-state what the prompt does.
**Applied:** Each prompt version is run at least twice, and results report both runs plus the overlap between them.

### L-17 · A prompt fix can create a new failure mode
**What happened:** v2 told the model to "state where the tester must look" to make oracles concrete. Without a description of the UI, it named verification points it could not know ("the calendar highlights the selected dates", "the dates are shown as unavailable") without flagging them as assumptions. Three cases were first scored as hallucinations; two were reversed after re-checking the app (see L-20), and one was confirmed (booked dates are not marked on the calendar).
**Lesson:** Every instruction has side effects. Re-run the whole rubric after each change, not only the metric you were trying to improve.
**Applied:** v1 vs v2 compares all metrics. v3 gives the model a UI map so it doesn't have to guess.

### L-18 · Check output formatting, not only content
**What happened:** In both v2 runs, the script-injection test data came out HTML-escaped (`&lt;script&gt;`). It reads fine in a rendered table, but a tester copying it would type the escaped text and the test would check nothing.
**Lesson:** LLM output can be correct in meaning and broken as an artifact. Test data has to be usable exactly as written.

### L-19 · Ask "what made you think that?" before changing a score
**What happened:** TC-026 and TC-030 were first scored as not executable. TC-026 needed the precondition from another case to make sense; for TC-030, the reviewer didn't expect the app to accept the same date for Check In and Check Out, then checked it in the app and found it does.
**Lesson:** A reviewer's "N" often points at something real (here, a cross-case dependency and an untested assumption about the app). Asking for the reason before overriding it surfaces those, and checking the app settles doubts better than arguing.

### L-20 · The reviewer can be wrong too: check the app
**What happened:** The peer review marked TC-001 and TC-004 as hallucinations because "the room calendar doesn't highlight the selected dates". That came from an earlier recording where the selected range was in December while the calendar showed October. A later recording with dates in the current month showed the calendar does mark the selected nights ("Sel…"). The two scores were reversed and the UI map for v3 was corrected before running it.
**Lesson:** An observation from one session can be an artifact of that session's data. Before a review claim becomes a score, or a fact goes into a prompt, confirm it in the app.

### L-21 · Two observations that contradict each other are not a reproduction
**What happened:** In one session, the booked room disappeared from the home page after "Return home". In the next session, the same steps showed all three rooms, and a different room disappeared when checking today–tomorrow. The app is a public demo where other people book the same default dates.
**Lesson:** When the same steps give different results and the environment is shared, the likely cause is the data, not the code. Calling it "reproduced" would put a weak bug in the portfolio.
**Applied:** Logged as an observation (OBS-01), not a bug. A bug needs consistent steps and results in data you control.

### L-22 · Score the test case, not the app
**What happened:** v3 TC-036 (same-day Check In and Check Out) expects a clear message. In the app no message appears, so the first review marked the case as not executable and as a hallucination.
**Lesson:** When the app doesn't do what a correct expected result says, the test fails, and that is the test doing its job. The rubric judges whether the case is valid, runnable and well-specified, not whether the app passes it. The real issue in TC-036 was elsewhere: its price check guarded against the wrong value (£140) instead of the actual risk (0 nights, £40 in fees).

## Writing test cases

### L-11 · Say where to verify
**What happened:** TC-002, TC-029 and TC-032 expect the room page to "show" the dates without saying where. The calendar does mark the selected nights ("Sel…"), but it always opens on the current month, so a range in a later month is only visible after clicking Next. The URL and the nights in the price summary are the reliable checks.
**Lesson:** An expected result needs a concrete place to look, or the tester has to guess.

### L-12 · Browser Back and in-app navigation are different tests
**What happened:** TC-032 says "return to the home page" without saying how.
**Lesson:** Browser Back can restore a cached page with old state, while a nav link reloads it. They exercise different behavior and should be separate cases.
