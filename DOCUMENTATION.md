# Real Estate Lead Qualification Workflow

**HOT / COLD lead scoring · n8n workflow · WhatsApp, Calendar and Email**  
**Date:** 23 October 2026  
**Built by:** Optimity Logics | info@optimitylogics.com

> Every new enquiry is checked, scored by AI and followed up automatically.

## 1. At a glance

| Item | Detail |
|---|---|
| Task | Real Estate Lead Qualification and Reply Scheduler |
| What it does | Receives a property enquiry, validates details, skips repeat enquiries, scores the lead with AI, saves it to Google Sheets and replies on WhatsApp. HOT leads also get a Google Meet site visit booked and a confirmation email. |
| Built with | n8n, Google Gemini, Google Sheets, Google Calendar, WhatsApp, Gmail |
| Trigger | A POST request to the webhook path `real-estate`, for example from a website enquiry form |
| Core idea | AI scores the lead from 0 to 100. A score of 80 or more triggers a booked meeting. |

## 2. The business problem

Enquiries arrive from forms and ads at any time. Sales staff manually check the details, look for duplicates, assess purchase readiness, find a calendar slot and write the first reply. This can delay follow-up and duplicate work.

The manual process is to:
1. Check that the name, phone, email, budget and location are supplied.
2. Search the lead list for previous enquiries.
3. Assess buyer readiness using budget, timeline and intent.
4. Find a free calendar slot and send a meeting invite.
5. Send the first WhatsApp message and email, then record the details.

## 3. The solution

An n8n workflow handles each enquiry from receipt through the first reply. It validates the submitted details, checks the phone number against Google Sheets, scores the lead with Google Gemini and saves the lead level. A Switch sends scores of 80 or more to the HOT path, which checks the calendar, chooses the first available hour tomorrow, creates a Google Meet event and messages the lead by WhatsApp and email. Other leads receive a polite WhatsApp reply and the team receives an email.

## 4. How it works

| Part | Node | What it does |
|---|---|---|
| Receive | Webhook1 | Receives a POST request on the `real-estate` path. |
| Validate | Code in JavaScript3 | Requires name, phone, email, budget, location and property type; validates phone and email; extracts BHK when available; stops on invalid data. |
| Duplicate check | Get row(s) in sheet1 + Code in JavaScript4 | Reads the lead sheet and compares phone numbers after removing spaces and symbols. |
| Duplicate handling | If1 + HTTP Request1 | New leads continue to scoring; duplicates receive a WhatsApp message saying the enquiry already exists. |
| Score | Basic LLM Chain1 + Google Gemini Chat Model1 | Scores the lead from 0–100 using five criteria and returns structured JSON. The document identifies the model as `gemini-3.1-pro-preview`. |
| Parse | Structured Output Parser1 | Requires fields such as `lead_score`, `qualification_status`, `budget_assessment`, `timeline`, `intent_strength`, `property_recommendations`, `follow_up_message` and `key_insights`. |
| Save | Append row in sheet1 | Adds the lead and sets status to HOT for scores 80 or above, or COLD below 80. |
| Route | Switch1 | Sends scores 80+ to HOT; all lower scores take the other path. |
| HOT calendar | Get many events1 + Code in JavaScript5 | Reads calendar events for the next 7 days and checks tomorrow from 9 AM to 5 PM in one-hour slots. If all are busy, uses 10 AM the day after tomorrow. |
| HOT meeting | Create an event1 | Creates the site-visit event with a Google Meet link, adds the lead as attendee and includes score, insights and suggested properties. |
| HOT follow-up | WhatsApp HOT + Email Lead - Meeting Link | Sends the lead the meeting date, time and location/link. |
| HOT record and alert | Update row in sheet + Send a message | Updates Meeting Slot and Notes in the sheet, and emails the team the HOT lead details and meeting link. |
| Other leads | WhatsApp COLD + Email COLD | Sends a thank-you message to the lead and emails the team the lead details, score and AI insights. |

## 5. What happens to each enquiry

| Case | Designed behavior |
|---|---|
| New lead, score 80 or more (HOT) | Saves the lead, books the first free hour tomorrow with a Meet link, sends WhatsApp and email confirmations, updates the sheet and emails the team. |
| New lead, score below 80 (COLD) | Saves the lead as COLD, sends a thank-you WhatsApp message and emails the team. |
| Phone number already in the sheet | Does not score or save the lead again; sends a WhatsApp message that the team will contact them. |
| Missing or invalid data | Validator stops the run and identifies the missing field or invalid phone/email. |

## 6. Rules the workflow follows

- **Required fields:** `name`, `phone`, `email`, `budget`, `location`, `property_type`.
- **Optional fields:** `message`, `purpose`, `timeline`, `source`.
- **Scoring:** 0–100, with 20 points each for clear budget, clear location, stated property type, timeline within 30 days, and readiness for a site visit or immediate purchase.
- **Lead levels:** HOT 80–100; COLD 0–79. The Switch separates scores 80+ from the rest.
- **Meeting slot:** One hour tomorrow between 9 AM and 5 PM India time, choosing the first available slot. If none is free, use 10 AM the day after tomorrow.
- **Duplicates:** Match phone numbers after removing spaces and symbols.
- **AI output:** Valid JSON matching the configured schema, with up to three property suggestions and two to three insights.

## 7. Data in Google Sheets and Calendar

The `Lead_Name_Sheet` spreadsheet is expected to contain: Lead Name, Phone Number, Email, Message, Budget, Location, Property Type, BHK, Purpose, Timeline, Lead Status, Created At, Meeting Slot and Notes. Each new lead is added, and HOT lead rows are updated after scheduling.

Google Calendar is checked for the next seven days before booking. The webhook input fields are:

| Field | Requirement | Example or note |
|---|---|---|
| `name` | Required | Customer name |
| `phone` | Required | Include country code, e.g. `+919876543210` |
| `email` | Required | Valid email address |
| `budget` | Required | e.g. `80 lakh` |
| `location` | Required | Preferred area or city |
| `property_type` | Required | e.g. `3 BHK flat` |
| `message`, `purpose`, `timeline`, `source` | Optional | Source defaults to Direct Enquiry; other optional values may be unspecified. |

## 8. Tools and technologies

| Tool | Used for | Why |
|---|---|---|
| n8n | Workflow orchestration | Connects webhook, AI, Sheets, Calendar, WhatsApp and Gmail. |
| Google Gemini | Lead scoring and follow-up content | Produces a structured score for routing. |
| Google Sheets | Lead list and duplicate check | Gives the sales team a reviewable record. |
| Google Calendar | Slot check and Meet event | Helps avoid calendar clashes and provides a meeting link. |
| WhatsApp | First reply to the lead | Familiar customer communication channel. |
| Gmail | Lead confirmations and team alerts | Keeps the team informed. |

## 9. Scenarios the workflow is built to handle

These are designed behaviors; test results are not included in the document.

| Scenario | Designed behavior |
|---|---|
| Phone number lacks country code | Validator stops and identifies the bad phone number. |
| Same person submits twice | Second enquiry is not scored or saved; the lead receives an already-received message. |
| Strong lead with clear details and readiness to visit | Scores 80+, meeting is booked and lead is messaged/emailed. |
| Early researcher with vague timeline | Scores lower, is saved, receives a thank-you message and triggers a team email. |
| Every slot tomorrow is busy | Schedules for 10 AM the day after tomorrow. |
| Calendar has no events | Books the 9 AM slot tomorrow. |

## 10. Human approval and safety points

- Nothing is saved or sent until phone and email pass validation.
- A repeat enquiry does not create a second row or meeting.
- The team receives an email for every lead.
- There is no human approval step before a HOT meeting is booked; the team can cancel it in Calendar if needed.
- AI output is constrained to a fixed JSON format before messages are sent.

## 11. Limitations and production needs

- This document does not include recorded test results. Run the scenarios in section 9 before go-live.
- Business-initiated WhatsApp messages may require an approved template; plain text messages may not be delivered outside the 24-hour window.
- The webhook has no response node, so validation errors are visible in n8n executions rather than returned to the sender.
- Meetings are scheduled for tomorrow or the day after; Sundays and holidays are not excluded.
- Two simultaneous leads could receive the same slot. The calendar read has no start date and also reads older events.
- The entire lead sheet is read for each enquiry, which may slow duplicate checks as the sheet grows.
- Team alerts and the calendar are configured for one Google account. Use an approved shared team inbox and calendar.
- The model is a preview version (`gemini-3.1-pro-preview`) and may change or be withdrawn.
- Names, phone numbers and email addresses are personal data; restrict access to the sheet and calendar.

## 12. Key learnings and improvements

One 0–100 score is used to keep routing, sheet status and messages aligned. Recommended improvements are approved WhatsApp templates, a Respond to Webhook node for validation errors, skipping Sundays and holidays, a shared team inbox, meeting reminders and faster duplicate lookup.

---

**Optimity Logics**  
Email: info@optimitylogics.com
