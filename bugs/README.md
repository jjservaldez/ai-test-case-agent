# Bug reports — Shady Meadows B&B booking flow

App: https://automationintesting.online (public demo) · Tester: Jose Serna · Found: 2026-10-17 to 2026-10-19 · Environment: Google Chrome on macOS

| ID | Severity | Title | How it was found | Status |
|---|---|---|---|---|
| [BUG-01](BUG-01.md) | High | Booking panel crashes when the server rejects a booking | Exploratory testing; third reproduction path from AI-generated case v2 TC-030 | Open |
| [BUG-02](BUG-02.md) | High | Invalid date range is not validated: price shows negative or zero nights | Exploratory testing; zero-night variant from AI-generated case v2 TC-030 | Open |
| [BUG-06](BUG-06.md) | High | Clicking a date in the room calendar lowers the displayed price, but the original range is booked | Found while verifying the room calendar for the AI agent's UI map | Open |
| [BUG-03](BUG-03.md) | Medium | "View Details" in Similar Rooms loses the selected dates (£NaN, booking fails) | Exploratory testing | Open |
| [BUG-04](BUG-04.md) | Medium | First and last name accept values made only of digits | Exploratory testing | Open (needs product decision) |
| [BUG-05](BUG-05.md) | Low | Validation messages don't name the field; generic failure message | Exploratory testing | Open |
| [BUG-07](BUG-07.md) | Low | "Amenities" navigation link doesn't go to any section | Exploratory testing | Open (outside booking flow) |

## Severity scale

- **High:** wrong price or booking, data integrity, or the user can't complete or recover from the flow.
- **Medium:** a path through the flow is broken or bad data is accepted, with a workaround.
- **Low:** cosmetic, unclear messaging, or outside the main flow.

## Notes on this environment

- The app is a public demo. Other users book the same default dates (today and tomorrow), and the data is reset periodically. State-dependent bugs were reproduced with no time gap and with far-future dates to avoid interference.
- One behavior was investigated and **not** reported as a bug: the rooms listed on the home page for today–tomorrow changed between sessions (OBS-01 in the exploration notes). The steps didn't give consistent results, and the most likely cause is other users' bookings.
