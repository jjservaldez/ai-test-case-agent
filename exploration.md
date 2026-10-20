# Exploration: Shady Meadows B&B booking flow

App: https://automationintesting.online · Method: exploratory testing with screen recordings (5 sessions on 2026-10-17, 3 verification sessions on 2026-10-19) · Tester: Jose Serna

> There are no official requirements for this app. The acceptance criteria below were written from observed behavior.

## User story

As a guest, I want to book a room for specific dates by entering my contact details, so that I can secure my stay.

## Acceptance criteria

| ID | Criterion | Observed |
|---|---|---|
| AC1 | Dates chosen on the home page (Check In / Check Out) are passed to the room page and are the dates that get booked | Pass |
| AC2 | The price summary shows nightly rate × number of nights + cleaning fee (£25) + service fee (£15) = total | Pass (e.g. Double, 1 night = £190) |
| AC3 | The first click on "Reserve Now" shows the booking form (Firstname, Lastname, Email, Phone) and a "Cancel" button | Pass |
| AC4 | Firstname is required, 3–18 characters | Pass, but digits-only values are accepted (BUG-04) |
| AC5 | Lastname is required, 3–30 characters | Pass, but digits-only values are accepted (BUG-04) |
| AC6 | Email is required and must be a valid email format | Pass ("test.com" rejected; "test@test" accepted, see observations) |
| AC7 | Phone is required, 11–21 characters | Pass (doesn't check that it contains only digits) |
| AC8 | If any field is invalid, the booking is not created and error messages are shown | Pass, but messages don't name the field (BUG-05) |
| AC9 | With valid data, "Booking Confirmed" is shown with the booked date range | Pass |
| AC10 | Dates already booked for a room cannot be booked again | Booking is blocked, but the page crashes (BUG-01) |
| AC11 | Check Out must be after Check In | Fail (BUG-02, BUG-01) |
| AC12 | Reaching a room page from any path keeps the selected dates or asks the user to select them | Fail (BUG-03) |

## Observed validation rules

- All fields empty → 7 messages: "size must be between 11 and 21", "must not be empty", "Lastname should not be blank", "Firstname should not be blank", "size must be between 3 and 30", "size must be between 3 and 18", "must not be empty".
- Malformed email → "must be a well-formed email address".

## Bug log

Full reports with steps and screenshots: [bugs/](bugs/README.md)

| ID | Severity | Summary | Source |
|---|---|---|---|
| BUG-01 | High | Rejected booking isn't handled; the page crashes ("This page couldn't load", then the panel loads forever) | Exploration (dates taken, reversed dates); same-day path from AI-generated case v2 TC-030 |
| BUG-02 | High | Invalid range isn't validated: negative nights and total (e.g. £-2210), or 0 nights and £40 | Exploration; 0-night variant from v2 TC-030 |
| BUG-03 | Medium | "View Details" in Similar Rooms sends `checkin=null&checkout=null`: £NaN, then "Failed to create booking" | Exploration |
| BUG-04 | Medium | Firstname and Lastname accept values made only of digits | Exploration |
| BUG-05 | Low | Error messages don't name the field; "Failed to create booking" gives no cause | Exploration |
| BUG-06 | High | Clicking a date in the room calendar changes the price to 1 night, but the calendar, URL and booking keep the original range (£190 shown, 3 nights / £490 booked) | Verifying the UI map |
| BUG-07 | Low | "Amenities" navigation link changes the URL to /#amenities but goes nowhere (outside the booking flow) | Exploration |
| OBS-01 | n/a | Rooms listed on the home page for today–tomorrow changed between sessions | Not reproducible consistently; likely other users' bookings on the public demo. Not reported as a bug |

## UI observations (used for the v3 UI map)

- The room calendar always opens on the current month with today highlighted. The selected nights show as a "Selected" bar ("Sel…" for one night); a range in a later month is only visible after clicking Next. (Corrected on 2026-10-19: an earlier note said the calendar didn't show the selection.)
- The calendar doesn't mark dates already booked by other guests.
- Room IDs: Single = 1, Double = 2, Suite = 3.
- The home page accepts the same date for Check In and Check Out without a warning.
- Booking today after the 3:00–8:00 PM check-in window is allowed.
- "test@test" is accepted as an email. Technically a valid format, so recorded as an observation.
- The demo resets its data periodically: state-dependent bugs were confirmed by repeating them immediately.
