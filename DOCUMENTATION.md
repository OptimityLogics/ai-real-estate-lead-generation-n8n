# Real Estate Lead Qualification Workflow — Documentation

**Built by Optimity Logics**  
**Platform:** n8n  
**Integrations:** Google Gemini, Google Sheets, Google Calendar, WhatsApp Business Platform, Gmail

## 1. Purpose

This workflow automates the initial processing of real-estate enquiries. It validates submitted details, detects duplicate phone numbers, scores buyer intent with AI, records the lead in Google Sheets, and sends follow-up messages. HOT leads also enter an automated meeting-scheduling path.

## 2. Business Problem

Sales teams often manually validate enquiry details, search for duplicates, assess purchase readiness, find calendar availability, and write initial responses. This can delay follow-up and result in repeated work. The workflow automates these first steps so the team can prioritize and respond faster.

## 3. Workflow Summary

1. **Receive:** A POST request arrives at the n8n webhook path `real-estate`.
2. **Validate:** Required fields, international phone number format, and email format are checked.
3. **Check duplicates:** The workflow compares the submitted phone number with existing Google Sheets rows after removing non-digit characters.
4. **Score:** Google Gemini returns a structured qualification result with a score, lead level, suggested properties, follow-up message, and insights.
5. **Save:** New lead information and its HOT/WARM/COLD status are written to Google Sheets.
6. **Route:** Scores of 80 or more enter the HOT path. Scores below 80 enter the lower-score path.
7. **Follow up:** WhatsApp messages and email alerts are sent. HOT leads also get a Google Calendar event with Google Meet details, a confirmation email, and an updated sheet row.

## 4. Required Input

The webhook expects a JSON body with these required fields:

| Field | Required | Example |
| --- | --- | --- |
| `name` | Yes | Alex Patel |
| `phone` | Yes | +919876543210 |
| `email` | Yes | alex@example.com |
| `budget` | Yes | 80 lakh |
| `location` | Yes | Shela, Ahmedabad |
| `property_type` | Yes | 3 BHK flat |
| `message` | No | Looking for a family home |
| `purpose` | No | Self-use |
| `timeline` | No | Within 30 days |
| `source` | No | Website enquiry |

Phone numbers must use international E.164 format. The workflow stops with an error if required information is missing or phone/email validation fails.

## 5. AI Scoring Rules

The prompt awards up to 20 points for each of these criteria:

- Budget clearly specified
- Preferred location clearly specified
- Property type specified
- Timeline within 30 days
- Readiness for a site visit or immediate purchase

| Score | Lead level | Intended handling |
| --- | --- | --- |
| 80–100 | HOT | Schedule a meeting and send priority follow-up |
| 50–79 | WARM | Save the lead and send the general acknowledgement |
| 0–49 | COLD | Save the lead and send the general acknowledgement |

The AI response is constrained to a JSON structure containing the score, qualification status, budget assessment, timeline, intent strength, up to three property recommendations, a follow-up message, and key insights.

## 6. Duplicate Handling

The workflow reads the existing lead sheet and compares phone numbers after stripping spaces and symbols. If the phone number already exists, the lead is not scored or added again. A WhatsApp message informs the prospect that the enquiry is already recorded and the team will contact them.

## 7. Meeting Scheduling

For HOT leads, the workflow reads calendar events, checks one-hour slots for tomorrow from 9:00 AM through 5:00 PM India time, and selects the first available slot. If no slot is free, it falls back to 10:00 AM the following day. It then creates a calendar event, adds the prospect as an attendee, includes Google Meet details, messages the prospect, updates the sheet, and sends email notifications.

**Important:** The slot finder is not a transactional lock. Two simultaneous executions could select the same slot. Sundays and holidays are not excluded by the current logic.

## 8. Google Sheets

The lead sheet is expected to contain these columns:

- Lead Name
- Phone Number
- Email
- Message
- Budget
- Location
- Property Type
- BHK
- Purpose
- Timeline
- Lead Status
- Created At
- Meeting Slot
- Notes

The workflow reads existing rows for duplicate detection, appends each new lead, and updates the meeting slot and notes for HOT leads.

## 9. Connected Services

| Service | Role |
| --- | --- |
| n8n | Orchestrates the workflow |
| Google Gemini | Scores leads and generates follow-up content |
| Google Sheets | Stores leads and supports duplicate detection |
| Google Calendar | Checks availability and creates meeting events |
| WhatsApp Business Platform | Sends lead-facing messages |
| Gmail | Sends lead confirmations and internal alerts |

## 10. Test Scenarios

Test these scenarios before production use:

1. A valid new lead with a score of 80 or above.
2. A valid lead with a score below 80.
3. A repeated phone number.
4. Missing required fields.
5. Invalid phone number or email.
6. A calendar with no events.
7. A calendar where all preferred slots are occupied.
8. Confirmation that the correct lead row is updated after a meeting is created.

The scenarios above describe intended behavior; test results are not included in this document.

## 11. Known Limitations and Production Checklist

- There is no separate WARM branch. WARM leads are handled by the same lower-score path as COLD leads, although their sheet status remains WARM.
- The lower-score email node is named “Email COLD,” even for WARM leads.
- Business-initiated WhatsApp messages may require an approved message template, depending on the conversation window and WhatsApp policy.
- The webhook has no response node, so callers do not receive a structured success or validation-error response.
- Calendar scheduling does not exclude Sundays or holidays and may have race conditions under concurrent requests.
- The entire lead sheet is read for each duplicate check, which may become slow as the sheet grows.
- The current workflow uses a specific calendar and team email. Replace these with an approved shared team calendar and inbox.
- The configured Gemini model is a preview model and may change availability.
- Limit access to sheets and calendars containing names, phone numbers, email addresses, budgets, and other personal information.

Before go-live, test every scenario, review the scoring prompt, add a dedicated WARM path, configure approved WhatsApp templates, add a webhook response, and secure all credentials.

## 12. Security

The workflow JSON committed to a public repository must not contain live API keys, access tokens, OAuth secrets, personal contact details, or real customer records. Reconnect credentials inside n8n and use placeholder values for spreadsheet IDs and phone-number IDs in public files.

---

**Optimity Logics**  
Email: info@optimitylogics.com
