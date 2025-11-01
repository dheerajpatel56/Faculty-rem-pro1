# Faculty Class Reminder Automation
> Automated academic email reminders using **n8n**, **Google Sheets**, **Gemini AI**, and **Gmail**.

---

 🧠 Overview
This n8n workflow automatically sends **email reminders** to faculty members whose classes are scheduled to start **within the next 5 minutes**.

It connects Google Sheets, Gemini AI, and Gmail to create a fully automated, AI-powered academic reminder system.

---

 🚀 Features
- 📄 Reads faculty timetable from Google Sheets  
- ⏰ Converts Excel serial timestamps to readable date/time  
- 🔍 Filters only classes starting in the next 5 minutes  
- 🤖 Uses Gemini AI to generate polite, formal reminders  
- 📧 Sends personalized emails via Gmail automatically  
- 🔁 Runs every minute via Cron scheduling  

---

 🧩 Workflow Architecture

[ Cron Trigger (Every Minute) ]
↓
[ Read Faculty Timetable (Google Sheets) ]
↓
[ Code Node: Convert Excel Time → ISO Date ]
↓
[ Filter Node: Class starts within next 5 minutes ]
↓
[ Gemini AI Agent: Generate Reminder Message ]
↓
[ Code Node: Parse AI JSON Output ]
↓
[ Gmail Node: Send Email Reminder ]


---

 ⚙️ Workflow Details

 1️⃣ Cron Trigger
Runs every minute to check for upcoming classes.  
**Interval:** Every 1 minute  

---

 2️⃣ Google Sheets Node
Reads faculty timetable data.

| FacultyName | Email | Subject | Class | StartTime | EndTime | Room | Status | Notes |
|--------------|--------|----------|--------|------------|----------|--------|--------|--------|
| Name | haxxxxxxxxxxxxxxxxxxxxm | Python Lab | CSE-A | 2025-11-01 12:30 | 2025-11-01 13:30 | Lab-2 | Pending | Lab session |

**Sheet ID:**  
`1xxxxxxxxxxxxxxxxxxxxxxxxxx`

---

### 3️⃣ Code Node — Convert Excel Time

```js
function excelToDate(excelSerial) {
  if (!excelSerial) return "";
  const excelEpoch = new Date(1899, 11, 30);
  const days = Math.floor(excelSerial);
  const msFromDays = days * 24 * 60 * 60 * 1000;
  const fraction = excelSerial - days;
  const msFromFraction = fraction * 24 * 60 * 60 * 1000;
  const jsDate = new Date(excelEpoch.getTime() + msFromDays + msFromFraction);
  return jsDate.toISOString();
}

for (const item of $input.all()) {
  const row = item.json;
  row.StartTime = excelToDate(parseFloat(row.StartTime));
  row.EndTime = excelToDate(parseFloat(row.EndTime));
}
return $input.all();


Converts Excel serial numbers like 45962.52083333 → 2025-11-01T12:30:00.000Z.

4️⃣ Filter Node — Upcoming & Pending Classes
const now = new Date();
const fiveMinLater = new Date(now.getTime() + 5 * 60 * 1000);

return $input.all().filter(item => {
  const row = item.json;
  if (!row.StartTime || row.Status?.toLowerCase() !== "pending") return false;
  const start = new Date(row.StartTime);
  return start >= now && start <= fiveMinLater;
});


Logic:

Include only rows with Status = Pending

Start time is within the next 5 minutes

5️⃣ Gemini AI Agent — Generate Reminder

Model: gemini-2.5-flash

Prompt:

You are an academic automation assistant AI.

Given the following data, generate a short, polite, formal email reminder.

Output strictly in JSON array format:
[
  {
    "email": "[Faculty Email]",
    "subject": "Class Reminder: [Subject] ([Class])",
    "message": "[Email Body]"
  }
]

6️⃣ Code Node — Parse Gemini JSON Output
const rawOutput = $json.output || "";

try {
  const cleaned = rawOutput.replace(/```json|```/g, "").trim();
  const parsed = JSON.parse(cleaned);
  return parsed.map(obj => ({ json: obj }));
} catch (error) {
  return [{ json: { error: "Failed to parse Gemini output", raw: rawOutput } }];
}


This converts Gemini’s string output to valid JSON objects for Gmail.

7️⃣ Gmail Node — Send Email Reminder

Config:

To: {{ $json["email"] }}

Subject: {{ $json["subject"] }}

Message: {{ $json["message"] }}

Example Email:

Subject: Class Reminder: Python Lab (CSE-A)

Dear Hasini,
This is a kind reminder that your class on Python Lab for CSE-A
is scheduled to begin at 12:30 PM in Lab-2.

Please arrive on time.

Best regards,
Academic Automation System

🧰 Requirements

n8n (Cloud or Self-Hosted)

Google API (Sheets + Gmail)

Gemini API Key

Google Service Account JSON

Cron Scheduling Enabled

🔒 Environment Variables
GEMINI_API_KEY=your_gemini_api_key_here
GOOGLE_SHEETS_CREDENTIALS=path_to_your_service_account.json

📈 Optional Enhancements

✅ Update “Status” column to Done after email is sent

🧾 Log all sent messages in a separate sheet

⚠️ Add retry logic for failed emails

🌙 Schedule reminders only during active hours

👨‍💻 Author

Dheeraj Patel

Built with ❤️ using n8n, Google APIs, and Gemini AI

🏷 License

MIT License — feel free to use, modify, and improve.

🖼 Preview

(Add a workflow screenshot here later)

<img width="1466" height="468" alt="Screenshot 2025-11-01 150128" src="https://github.com/user-attachments/assets/5bb2e023-e35d-4c46-a1b9-83b5d1f496f9" />



---

