# Smart Lead Intake API

A BuildShip workflow (`lead-intake`) that receives contact-form submissions, validates them, classifies each lead with Gemini, saves it to Google Sheets and alerts the team on Slack and Gmail when the lead is **hot**.

## Submission

| Deliverable | Link / file |
|-------------|-------------|
| Workflow canvas screenshots | `01_Workflow/screenshots/` _(add after capturing)_ |
| Shared workflow link | `[ADD SHARED BUILDSHIP WORKFLOW LINK]` (editor: https://app.buildship.com/p/buildship-3zhkbl/flow/d89R4M7X11a4Oh6g2OsF) |
| Google Sheet (8+ test leads, view access for mentor) | https://docs.google.com/spreadsheets/d/1GiriQc99kEuui99oZ7xXmomdQhwumZtoyNgF8J3XWPM/edit?gid=0#gid=0 |
| curl commands for every test case | `03_Test_Requests/curl-commands.md` |
| Postman collection | `03_Test_Requests/lead-intake.postman_collection.json` |
| README | this file |
| Demo recording (3–5 min, incl. one hot-lead alert) | https://drive.google.com/file/d/12yVtt9KG954k7BySYE4-gGRNRklHAjhP/view?usp=drive_link |

![Workflow canvas](01_Workflow/screenshots/workflow-canvas-1.png)

## Endpoint

```
POST https://3zhkbl.buildship.run/leadIntake
Content-Type: application/json
```

### Request body

| Field     | Type   | Required | Rule                                         |
|-----------|--------|----------|----------------------------------------------|
| `name`    | string | yes      | At least 2 characters after trimming        |
| `email`   | string | yes      | Valid email format, stored lower-cased       |
| `company` | string | no       | Trimmed                                      |
| `message` | string | yes      | At least 10 characters after trimming       |
| `source`  | string | no       | Defaults to `"unknown"`                      |

```json
{
  "name": "Sara Ahmed",
  "email": "sara@acme.com",
  "company": "Acme Ltd",
  "message": "We need a chatbot for our store, budget ready, want to start this month.",
  "source": "website-contact-form"
}
```

### Responses

**200 OK** – lead accepted, classified and saved

```json
{
  "success": true,
  "leadId": "L-1791037532295",
  "priority": "hot",
  "serviceInterest": "ai-chatbot",
  "summary": "Acme Ltd needs a chatbot for their store and is ready to start with a budget this month."
}
```

- `priority`: `hot` | `warm` | `cold` | `unscored` (AI unavailable)
- `serviceInterest`: `ai-chatbot` | `web-development` | `mobile-app` | `automation` | `ui-ux-design` | `other` | `unknown`

**400 Bad Request** – validation failed, nothing saved; every problem is listed

```json
{ "success": false, "errors": ["email is not valid", "message must be at least 10 characters"] }
```

## How it works

| # | Node | What it does |
|---|------|--------------|
| 1 | REST API trigger | `POST /leadIntake` with the five body fields |
| 2 | Validate and Clean (script) | Trims text, lower-cases email, checks name/email/message, creates `leadId` and `timestamp` |
| – | Branch `isValid === false` | Returns the 400 error JSON |
| 3 | Classify Lead (script) | Calls Gemini with a strict JSON-only prompt. Tries `gemini-flash-latest`, then `gemini-2.5-flash`. If both fail or the JSON is broken → `priority: "unscored"` |
| 4 | Add Row (Google Sheets) | Appends one row to `Sheet1` of the Leads sheet |
| 5 | Branch `priority === "hot"` | Only hot leads go to step 6 |
| 6 | Notify Slack (script) + Send Email (Gmail) | Slack message via webhook; HTML email to the team. Alert failures are logged but never break the response |
| 7 | Output | Returns the 200 JSON |

Validation runs before the AI call (no AI requests spent on bad data), and the row is saved before alerts (no lead lost if an alert fails).

### Google Sheet columns (row 1)

`timestamp | leadId | name | email | company | message | source | priority | serviceInterest | summary`

## Setup (new teammate)

1. **BuildShip** – open project `buildship-3zhkbl` → flow `lead-intake`.
2. **Secrets** (Settings → Secrets, never paste keys into code):
   - `GEMINI_API_KEY` – free key from Google AI Studio
   - `SLACK_WEBHOOK_URL` – Slack → Apps → Incoming Webhooks → channel webhook URL
3. **Google Sheet** – create a sheet with the 10 headers above in `Sheet1`; paste its URL into the **Add Row** node.
4. **Google auth** – click the fingerprint icon on **Add Row** and **Send Email** and connect a Google account that can edit the sheet and has **Gmail send** permission.
5. **Alert recipient** – set in the **Send Email** node (currently `saad.tanveer11400@gmail.com`).
6. Click **Ship Changes** to deploy, then run the commands in `03_Test_Requests/curl-commands.md` or import the Postman collection.

## Testing

`03_Test_Requests/curl-commands.md` and the Postman collection cover every case in the brief:

| Test | Expected |
|------|----------|
| Clear buying intent | 200, `hot`, row saved, Slack + email alert |
| General question, no budget/timeline | 200, `warm`/`cold`, row saved, no alert |
| Missing email / invalid email | 400 with clear error, nothing saved |
| Message under 10 characters | 400 with clear error, nothing saved |
| AI fails (wrong key) | 200, `unscored`, row still saved |

To test the AI failure: on **Classify Lead**, point *Gemini API Key* at a wrong/empty value, send a valid lead, then switch it back.

## Notes / known issues

- **Gmail alert**: the connected account currently lacks the Gmail send scope (403 "insufficient authentication scopes"). Re-authenticate the Send Email node and allow sending. Slack alerts work.
- **Path**: the live path is `/leadIntake`; the brief says `/lead-intake`. Rename under Connect → Path if needed (and update the URL in the tests).
- `leadId` uses a millisecond timestamp (`L-<ms>`) to avoid collisions between leads sent in the same second.
