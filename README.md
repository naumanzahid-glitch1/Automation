Enduring Planet FCFO — Gmail Follow-up & Escalation Add-on

A production-ready Google Workspace Add-on (Gmail Sidebar) built with Google Apps Script (V8), @google/clasp, Gemini Pro, Google Sheets, and Google Calendar for Enduring Planet's Fractional CFO (FCFO) practice.

Architecture & Escalation Rules
1. Business-Day Escalation Schedule (Mon–Fri Only, Strictly Skips Sat/Sun)
Stage	Prior Follow-ups Sent	Wait Threshold	Recipient & Action Behavior
Follow-up 1	0	2 Business Days	Standard follow-up to Primary POC.
Follow-up 2	1	2 Business Days	Standard follow-up to Primary POC.
Follow-up 3	2	2 Business Days	Final 2-day cadence follow-up to Primary POC.
Follow-up 4 (Checkpoint)	3	1 Business Day	Stage 4 Escalation Checkpoint: Automatically adds Secondary POC to the CC: field and prompts the user with a required verification checkbox before sending.
Follow-up 5	4	1 Business Day	Daily escalated follow-up with Secondary POC in CC:.
Follow-up 6	5	1 Business Day	Final daily email follow-up with Secondary POC in CC:.
Stage 7 (Calendar Hold)	6+	1 Business Day	Disables all further email drafts and presents the "Schedule 30-Min Calendar Hold" action inviting both Primary POC and Secondary POC.
2. Sidebar Actions
Approve & Send Follow-up: Replies directly to the active Gmail thread using the editable To:, CC:, and multiline AI Follow-up Draft fields, enforces <90 words and Stage 4 verification, increments Follow-up Count, and updates Last Contact Date in the ThreadState sheet.
Close Loop Internally: Sends a structured audit note strictly to the internal group at @enduringplanet.com (never emails the client) and marks the thread RESOLVED_INTERNALLY in the ThreadState sheet.
Schedule 30-Min Calendar Hold (Stage 7): Schedules a 30-minute Google Calendar event on the next business day (Mon–Fri) inviting both Primary and Secondary client contacts.
3. Gemini Pro AI Brain (src/GeminiBrain.js)
Calls the Gemini Pro REST endpoint (gemini-1.5-pro by default, or gemini-2.0-pro via optional GEMINI_MODEL property) using GEMINI_API_KEY stored in Script Properties.
Enforces the 7 C's of Communication:
Clear
Concise (<90 words)
Concrete
Correct
Coherent
Complete
Courteous
Forwarded Thread Evaluation: Automatically detects forwarded threads (Fwd:, FW:, forwarded message headers) and evaluates whether Enduring Planet is explicitly responsible for next steps. If not, displays a recommendation banner advising the operator to click "Close Loop Internally".
Project Structure
text
ep-fcfo-gmail-addon/
├── .clasp.json                  # Clasp configuration pointing to src/
├── .eslintrc.json               # ESLint config for Google Apps Script & Jest
├── package.json                 # Node dependencies (clasp, jest, eslint)
├── README.md                    # Setup, Script Properties, and IT deployment guide
├── src/
│   ├── appsscript.json          # Workspace Add-on manifest & OAuth scopes
│   ├── Code.js                  # Gmail sidebar UI builder & button event handlers
│   ├── GeminiBrain.js           # Gemini Pro REST caller & 7 C's system instructions
│   ├── TrackerService.js        # Google Sheet CRUD & Mon-Fri business day math
│   └── CalendarService.js       # Stage 7 30-minute Google Calendar hold creator
└── tests/
    └── escalationLogic.test.js  # Jest test suite for business day math & stages
Step 1: Google Sheet Database Setup
Create a new Google Sheet in Enduring Planet's shared Google Drive (e.g., EP FCFO Escalation Tracker).
Copy the Spreadsheet ID from the URL: https://docs.google.com/spreadsheets/d/<TRACKER_SHEET_ID>/edit
Create two tabs (or run initializeTrackerSheet() once in the Apps Script editor to auto-create them):
Tab 1: Clients
Column A (Domain)	Column B (Client Name)	Column C (Primary POC)	Column D (Secondary POC)	Column E (Internal Group)
heliosclean.com	Helios Clean Energy	ceo@heliosclean.com	coo@heliosclean.com	fcfo-ops@enduringplanet.com
terravolt.energy	TerraVolt Energy	finance@terravolt.energy	vp-ops@terravolt.energy	fcfo-ops@enduringplanet.com
Tab 2: ThreadState
Column A (Thread ID)	Column B (Follow-up Count)	Column C (Last Contact Date)	Column D (Status)
18f9a2b3c4d5e6f7	3	2026-10-23T16:00:00.000Z	ACTIVE
Step 2: Local Setup & Running Tests
bash
# Install dependencies
npm install
# Run the Jest unit test suite
npm test
# Run linter
npm run lint
Step 3: Pushing to Google Apps Script via @google/clasp

Enable the Apps Script API: Visit https://script.google.com/home/usersettings
 and toggle Google Apps Script API to ON.

Authenticate clasp:

bash
npx clasp login

Create or Link Your Apps Script Project:

Option A (Link existing project): Open your Apps Script project -> Project Settings -> copy the Script ID and paste it into .clasp.json under "scriptId".
Option B (Create new project via CLI):
bash
npx clasp create --type standalone --title "Enduring Planet FCFO Gmail Add-on" --rootDir ./src

Push Code and Manifest:

bash
npx clasp push

(When prompted to overwrite appsscript.json, confirm with y.)

Step 4: Configure Script Properties
Open the project in the Apps Script Editor:
bash
npx clasp open
Navigate to Project Settings (gear icon on the left sidebar) -> scroll down to Script Properties.
Click Add script property and configure the following keys:
Property	Value	Description
GEMINI_API_KEY	AIzaSy...	Google AI Studio / Vertex Gemini API Key for gemini-1.5-pro / gemini-2.0-pro.
TRACKER_SHEET_ID	1AbC...	The Google Spreadsheet ID containing the Clients and ThreadState tabs.
GEMINI_MODEL (Optional)	gemini-1.5-pro	Defaults to gemini-1.5-pro. Can be set to gemini-2.0-pro.
In the Apps Script editor, select initializeTrackerSheet from the function dropdown and click Run once to verify spreadsheet permissions and initialize the Clients and ThreadState headers.
Step 5: IT & Google Workspace Deployment Guide
A. Single-User / Team Test Deployment (Immediate Testing)
In the Apps Script Editor, click Deploy -> Test deployments.
Under Application(s): Gmail, click Install.
Open Gmail
, open any email thread, and click the Enduring Planet FCFO icon in the right-hand Google Workspace side panel.
B. Domain-Wide Internal Deployment (@enduringplanet.com)
Associate with a Standard Google Cloud Project:
Open Google Cloud Console
 inside the enduringplanet.com organization and create/select a project (e.g., ep-fcfo-workspace-addon).
Enable the Google Workspace Marketplace SDK, Gmail API, Google Sheets API, and Google Calendar API.
Configure the OAuth Consent Screen as Internal (restricted to @enduringplanet.com users) and add the 7 scopes declared in src/appsscript.json.
Copy the GCP Project Number into Apps Script Project Settings -> Google Cloud Platform (GCP) Project.
Create a Versioned Deployment:
bash
npx clasp deploy --description "v1.0.0 Production FCFO Release"
Copy the generated Deployment ID.
Publish Internationally to @enduringplanet.com via Workspace Marketplace SDK:
In Google Cloud Console -> APIs & Services -> Google Workspace Marketplace SDK -> App Configuration:
Set App Visibility to Private (only available to users in enduringplanet.com).
Set Installation Settings to allow Admin Install (Domain-wide) and Individual Install.
Check Google Workspace Add-on, select Gmail, and paste the Deployment ID.
In Google Admin Console (admin.google.com) -> Apps -> Google Workspace Marketplace apps -> Apps list:
Select Enduring Planet FCFO Escalation Assistant and enable it for the Fractional CFO organizational unit (or entire domain).
