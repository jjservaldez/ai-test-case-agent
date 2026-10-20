# Evaluating an LLM test-case generator

A QA case study: can an LLM write manual test cases that actually find bugs, and how do you measure that?

I explored a booking flow by hand, wrote acceptance criteria from what I observed, then asked an LLM (Claude) to generate test cases from those criteria. I reviewed every generated case against a rubric, measured how many of the bugs I had already found its cases would catch, and improved the prompt over three versions.

**App under test:** [Shady Meadows B&B](https://automationintesting.online) booking flow (public demo app for testers) · **Author:** Jose Serna, QA Engineer

## Results

| | v1 | v2 | v3 |
|---|---|---|---|
| What changed | Baseline prompt | + rules for expected results, real-user scenarios | + UI map, literal test data |
| Test cases (run 1) | 40 | 44 | 48 |
| Known bugs detected (of the original 5) | **1** | **4** (both runs) | **3 + 1 partial** (both runs) |
| Expected results strong enough to fail on a real bug | 87.5% | 97.7% | 95.8% |
| Unflagged assumptions (hallucinations) | 0 | 1 | **0** |
| Duplicates | 7.5% | 0% | 0% |

Full comparison: [evaluation/comparison-v1-v2-v3.md](evaluation/comparison-v1-v2-v3.md)

### Key findings

1. **The model reached the right scenarios but wrote weak expected results.** In v1, three tests led straight to real bugs but expected only "the booking is not created". When the page crashed, that was still true, so the tests would have passed. Defining what a correct result must include took detection from 1 to 4 of 5.
2. **Every prompt fix had a side effect.** Asking the model to say where to verify results, without describing the UI, made it guess at UI behavior. Adding a UI map removed the guesses, and also narrowed one oracle so a known bug (unnamed error messages) went from detected to partial.
3. **One run is not a measurement.** Each version was run twice. A digits-only name test appeared in only one of the two runs, every time.
4. **The highest-severity bug came from a person, not the model.** No prompt version found [BUG-06](bugs/BUG-06.md) (the displayed price drops but the original stay is booked). I found it while checking a UI detail before giving it to the model.

## Bugs found

7 bugs, reported with steps, expected vs. actual results and screenshots: [bugs/](bugs/README.md)

| ID | Severity | Summary | Found by |
|---|---|---|---|
| [BUG-01](bugs/BUG-01.md) | High | Booking panel crashes when the server rejects a booking | Exploration; one path from an AI-generated case |
| [BUG-02](bugs/BUG-02.md) | High | Invalid date range gives negative or zero-night prices | Exploration; zero-night variant from an AI-generated case |
| [BUG-06](bugs/BUG-06.md) | High | Clicking a calendar date lowers the price, but the original range is booked | Verifying the UI map |
| [BUG-03](bugs/BUG-03.md) | Medium | "View Details" loses the dates (£NaN, booking fails) | Exploration |
| [BUG-04](bugs/BUG-04.md) | Medium | Names made only of digits are accepted | Exploration |
| [BUG-05](bugs/BUG-05.md) | Low | Error messages don't name the field | Exploration |
| [BUG-07](bugs/BUG-07.md) | Low | "Amenities" link goes nowhere | Exploration |

## Method

1. **Exploratory testing** of the booking flow with screen recordings, then **acceptance criteria** written from observed behavior (there are no official requirements). [exploration.md](exploration.md)
2. **Prompt versions** that differ in one documented way each. Prompt files contain only the text sent to the model; notes live in the [changelog](prompts/CHANGELOG.md). [prompts/](prompts/)
3. **Clean runs:** every run in a new chat with no project context, so the model never saw the known bugs. Raw output in [runs/](runs/).
4. **Rubric review of every case** (valid, executable, hallucination, duplicate, category, expected result strong enough), then a **second-pass peer review**.
5. **Bug detection** measured against the bugs found in exploration, classifying each miss as a model miss, an acceptance-criteria gap, or missing context.

The rubric's key metric is defined as: *imagine a realistic bug in what the case tests; if a tester follows the expected result literally, would they mark Fail?* It comes from the test-oracle problem and the logic of mutation testing.

## Lessons learned

22 lessons, each with what happened, the lesson, and what changed: [lessons-learned.md](lessons-learned.md). A few:

- **L-07:** reaching the scenario is not the same as detecting the bug; the expected result decides.
- **L-09:** polished AI output lowers the reviewer's guard. My first review scored 40/40 on everything; the second pass found duplicates, ambiguous steps and weak oracles.
- **L-13:** improve the prompt with general heuristics, not the known answers, or you overfit to one app.
- **L-20:** the reviewer can be wrong too. A claim about the calendar came from one recording's data and was reversed after checking the app.

## Repository structure

```
prompts/       v1, v2, v3 prompts and CHANGELOG
runs/          raw model output for each run (and the human-reviewed v1 suite)
evaluation/    per-version analysis and the v1 vs v2 vs v3 comparison
  reviews/     per-case rubric scores and peer review comments (CSV)
bugs/          bug reports with screenshot evidence
exploration.md acceptance criteria, observed rules, bug log
lessons-learned.md
```

## Limitations

- Two runs per prompt version; more runs would give a firmer consistency figure.
- One human reviewer plus an AI peer review, not two independent human reviewers.
- The reference bugs were found by the same person who reviewed the output.
- The app is a shared public demo; other users' bookings can change availability mid-session.

## Next steps

Turn the manual process into a script that sends each prompt version to the model through the API, runs it several times, and saves the output for review. That makes more runs per version cheap and is the step from a prompt to an agent.
