# AI Real Estate Lead Generation — n8n Automation

An AI-powered real estate lead qualification and follow-up workflow built with n8n, Google Gemini, Google Sheets, WhatsApp, Gmail, and Google Calendar. It validates incoming enquiries, detects duplicate leads, scores buyer intent, records lead details, and routes leads for follow-up.

## 1. Overview

This automation receives real estate enquiries through a webhook and processes the submitted lead details.

- Validates required fields, phone number format, and email address.
- Checks Google Sheets for an existing lead with the same phone number.
- Uses Google Gemini to score the lead from 0–100 and classify it as **HOT**, **WARM**, or **COLD**.
- Saves lead details and qualification status to Google Sheets.
- Sends WhatsApp follow-up messages and email notifications.
- For qualified HOT leads, checks Google Calendar for an available slot, creates a calendar event with Google Meet details, and updates the lead record.
- Sends a booking confirmation and meeting details to the lead.

## 2. Demo Video

[Watch the Real Estate Automation Demo](https://youtu.be/MCaHumjmY0Q?si=14aWk0-fsDMai-B-)

## 3. Tech Stack & Tools

- **n8n** — workflow automation and routing
- **Google Gemini** — AI lead scoring and qualification
- **Google Sheets** — lead database and status tracking
- **WhatsApp Business Platform** — lead follow-up messages
- **Gmail** — lead notifications and meeting details
- **Google Calendar** — availability checks and meeting scheduling

## 4. Prerequisites

- An n8n workspace
- Google Gemini API access
- Google Sheets spreadsheet with the required lead columns
- Google Calendar and Gmail access
- WhatsApp Business Platform credentials and a configured phone number ID
- A webhook endpoint reachable by the lead capture source

## 5. Import & Setup

1. Import the workflow JSON from the `workflow/` directory into n8n.
2. Connect your Google Gemini, Google Sheets, Google Calendar, Gmail, and WhatsApp credentials.
3. Replace spreadsheet and WhatsApp phone-number placeholders with your own configuration.
4. Confirm that your spreadsheet includes the columns listed below.
5. Configure the lead-capture form or application to send a POST request to the n8n webhook.
6. Test with sample data before activating the workflow.

**Required lead input fields:** `name`, `phone`, `email`, `budget`, `location`, and `property_type`.

Optional fields include `purpose`, `timeline`, `message`, and `source`. Phone numbers should use international E.164 format, for example `+919876543210`.

## 6. Google Sheets Fields

The lead sheet is expected to contain these columns:

| Column | Purpose |
| --- | --- |
| Lead Name | Name supplied by the prospect |
| Phone Number | Contact number used for duplicate checks |
| Email | Email address for follow-up and meeting details |
| Message | Original enquiry message |
| Budget | Prospect's stated budget |
| Location | Preferred property location |
| Property Type | Requested property type |
| BHK | Bedroom configuration extracted where available |
| Purpose | Buying or other stated purpose |
| Timeline | Expected purchase timeline |
| Lead Status | AI-derived HOT, WARM, or COLD status |
| Created At | Lead creation timestamp |
| Notes | Meeting and follow-up notes |
| Meeting Slot | Scheduled meeting date and time |

## 7. Lead Qualification Logic

The AI evaluates the lead against five criteria: budget, location, property type, timeline, and readiness to purchase or attend a site visit. Each criterion can contribute up to 20 points.

- **HOT (80–100):** Prioritize urgent follow-up and attempt meeting scheduling.
- **WARM (50–79):** Interested lead that may need additional nurturing.
- **COLD (0–49):** Early-stage enquiry for a general acknowledgement and follow-up.

The AI score supports prioritization; the sales team should verify the prospect's requirements before making business decisions.

## 8. How to Test

1. Send a POST request with all required lead fields to the configured webhook.
2. Verify that invalid or missing fields are rejected.
3. Submit the same phone number again and confirm duplicate handling.
4. Test HOT, WARM, and COLD examples and inspect the Google Sheets record.
5. For HOT leads, verify calendar availability, event creation, email delivery, and WhatsApp notification.
6. Confirm that the meeting slot and notes are written back to the correct lead row.

Use test contact details and a test spreadsheet while validating the workflow. Actual output depends on your connected services and configuration.

## 9. Repository Contents

- [Workflow JSON](workflow/real-estate-lead-generation.json)
- [Detailed Documentation](DOCUMENTATION.md)
- [Sample Spreadsheet](database/real-estate-leads.xlsx)

## 10. Security & Operational Notes

- Never commit API keys, access tokens, OAuth secrets, or real customer data.
- Reconnect credentials in n8n after importing the workflow.
- Replace all spreadsheet IDs and WhatsApp phone-number IDs with your own values.
- Restrict access to spreadsheets containing personal information.
- Test duplicate handling and calendar conflict checks before production use. Calendar slot checks are not a transactional lock, so simultaneous requests may still conflict.
- Review AI-generated scores and follow-up messages before relying on them for customer-facing decisions.

## Built by Optimity Logics

#OptimityLogics #RealEstateAutomation #n8n #LeadGeneration #AIAutomation
