# Prompt changelog

Prompt files contain only the text that is pasted or sent to the model. Notes about each version live here.

## v1 (baseline)
- Senior QA role, anti-hallucination rule (flag assumptions), categories, fixed table format.
- User input: user story, AC1–AC12, room prices.
- Note: run 1 was pasted including the file title "Prompt v1 — Generador de test cases (baseline)". Low impact, but the title is no longer in the file.

## v2
Changes from v1 (instructions only; the user input is identical, so the comparison isolates the effect of the instructions):
1. Oracle rules: expected results must list everything that must hold, including user feedback and app state.
2. Intermediate checks: verify displayed values at each step, not only the final outcome.
3. Real-user scenario checklist.
4. No duplicates; navigation method and verification point must be explicit.

The hints are generic testing heuristics, not descriptions of the defects found in exploration (see lessons-learned L-13).

## v3
Changes from v2:
1. User input: added an "Application UI" map written from the exploration sessions (pages, sections, buttons, URL format, where errors and confirmations appear, what the room calendar shows). It describes what exists, not what fails. Target metrics: hallucinations (unflagged UI assumptions) and reaching BUG-03 through the real user path.
2. Instructions: test data must be literal plain text (no HTML escaping, no described values). Target metric: executability.

This changes two things at once (see L-14). They target different metrics, so each effect can still be attributed.
