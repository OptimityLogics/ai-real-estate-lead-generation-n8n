# AI Real Estate Lead Generation — n8n Automation

An AI-powered real estate lead management workflow built with n8n, Google Gemini, Google Sheets, WhatsApp, Gmail, and Google Calendar. It validates enquiries, checks for duplicate phone numbers, scores buyer intent, saves lead details, and routes each new lead into a HOT or COLD follow-up path.

## 1. Overview

The workflow receives real estate enquiries through a webhook and processes the submitted lead details.

- Validates required fields, phone number format, and email address.
- Checks Google Sheets for an existing lead with the same phone number.
- Uses Google Gemini to score the enquiry from 0–100.
- Saves the lead details and qualification result to Google Sheets.
- Sends a WhatsApp acknowledgement and an email notification for the COLD path.
- For HOT leads (score 80 or above), checks Google Calendar for an available slot, creates a calendar event with Google Meet details, and updates the lead record.
- Sends the HOT lead a WhatsApp confirmation and an email containing the scheduled meeting details.
- If a phone number already exists, sends a duplicate-enquiry message rather than processing it as a new lead.

## 2. Demo Video

[Demo video link will be added soon]()

## 3. Tech Stack & Tools

- **n8n** — workflow automation and routing
- **Google Gemini** — AI-based lead scoring and qualification
- **Google Sheets** — lead database and status tracking
- **WhatsApp Business Platform** — lead follow-up messages
- **Gmail** — lead notifications and meeting details
- **Google Calendar** — availability checks and meeting scheduling

## 4. Prerequisites

- An n8n workspace
- Google Gemini access
- A Google Sheets spreadsheet with the required lead columns
- Google Calendar and Gmail access
- WhatsApp Business Platform credentials and a configured phone number ID
- A webhook endpoint reachable by the lead capture source

## 5. Import & Setup

1. Import the workflow JSON from the `workflow/` directory into n8n.
2. Connect your Google Gemini, Google Sheets, Google Calendar, Gmail, and WhatsApp credentials.
3. Configure the spreadsheet and WhatsApp phone-number ID for your own environment.
4. Confirm that your spreadsheet includes the columns listed below.
5. Configure the lead-capture form or application to send a POST request to the n8n webhook.
6. Test with sample data before activating the workflow.

**Required lead input fields:** `name`, `phone`, `email`, `budget`, `location`, and `property_type`.

Optional fields include `purpose`, `timeline`, `message`, and `source`. Phone numbers should use international E.164 format, for example `+919876543210`.

## 6. Google Sheets Fields

The workflow reads and writes these lead fields:

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
| Lead Status | Qualification status recorded for the lead |
| Created At | Lead creation timestamp |
| Notes | Meeting and follow-up notes |
| Meeting Slot | Scheduled meeting date and time |

## 7. Lead Qualification & Routing

Google Gemini evaluates the enquiry using the stated budget, location, property type, purchase timeline, and readiness to buy or attend a site visit.

- **HOT (score 80–100):** The workflow checks calendar availability, schedules a Google Calendar event with Google Meet details, updates the lead record, and sends WhatsApp and email confirmations.
- **COLD / non-HOT path (score below 80):** The workflow sends a WhatsApp acknowledgement and an email notification to the configured recipient.
- **Duplicate enquiry:** If the phone number already exists in the sheet, the workflow sends a duplicate-enquiry message and does not continue through the new-lead qualification path.

The workflow currently routes all scores below 80 to the non-HOT path. The lead-status expression in the supplied JSON still labels scores from 50–79 as `WARM` in Google Sheets, so the saved status can differ from the two follow-up paths. If the intended setup is strictly HOT/COLD everywhere, that expression and the AI prompt should also be updated in the workflow JSON.

## 8. How to Test

1. Send a POST request with all required lead fields to the configured webhook.
2. Verify that invalid or missing fields are rejected.
3. Submit the same phone number again and confirm duplicate handling.
4. Test one enquiry scoring 80 or above and verify calendar scheduling, the Google Sheets update, and the WhatsApp/email confirmations.
5. Test an enquiry scoring below 80 and verify the acknowledgement WhatsApp message and email notification.
6. Confirm that the meeting slot and notes are written back to the correct lead row for HOT leads.

Use test contact details and a test spreadsheet while validating the workflow. Actual output depends on connected services and configuration.

## 9. Repository Contents

- [Workflow JSON](workflow/real-estate-lead-generation.json)
- [Detailed Documentation](DOCUMENTATION.md)


## 10. Security & Operational Notes

- Never commit API keys, access tokens, OAuth secrets, or real customer data.
- Reconnect credentials in n8n after importing the workflow.
- Replace spreadsheet IDs and WhatsApp phone-number IDs with your own values before using the workflow.
- Restrict access to spreadsheets containing personal information.
- Test duplicate handling and calendar conflict checks before production use. Calendar slot checks are not a transactional lock, so simultaneous requests may still conflict.
- Review AI-generated scores and follow-up messages before relying on them for customer-facing decisions.

## Built by Optimity Logics

#OptimityLogics #RealEstateAutomation #n8n #LeadGeneration #AIAutomation
