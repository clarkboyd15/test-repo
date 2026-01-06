# Daily Morning Email - Setup Guide

This sends your morning checklist to clark@novela.academy at 8am every day, automatically. Free, runs forever, takes 5 minutes to set up.

---

## Setup Steps

### 1. Open Google Apps Script

Go to: [script.google.com](https://script.google.com)

Click **New Project**

### 2. Paste This Code

Delete any existing code and paste:

```javascript
function sendMorningChecklist() {
  const email = "clark@novela.academy";
  const today = new Date();
  const dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
  const dayName = dayNames[today.getDay()];

  // Get today's calendar events
  const calendar = CalendarApp.getDefaultCalendar();
  const events = calendar.getEventsForDay(today);

  let calendarSection = "";
  if (events.length > 0) {
    calendarSection = "<h3>📅 Today's Calendar</h3><ul>";
    events.forEach(event => {
      const startTime = event.getStartTime();
      const timeStr = Utilities.formatDate(startTime, Session.getScriptTimeZone(), "HH:mm");
      calendarSection += `<li><strong>${timeStr}</strong> - ${event.getTitle()}</li>`;
    });
    calendarSection += "</ul>";
  } else {
    calendarSection = "<h3>📅 Today's Calendar</h3><p>No meetings scheduled - protect this focus time!</p>";
  }

  // Themed day content
  const themedDays = {
    1: { theme: "Strategy & Planning", focus: "High-level thinking, investor prep, set the week's priorities" },
    2: { theme: "Sales & Outreach", focus: "HubSpot deep dive, prospecting, follow-ups" },
    3: { theme: "Customers & Product", focus: "Customer check-ins, classes, content work" },
    4: { theme: "Marketing & Comms", focus: "LinkedIn posts, newsletter, messaging strategy" },
    5: { theme: "Admin & Reflection", focus: "Receipts, Revolut, CEO review, week wrap-up" },
    0: { theme: "Rest", focus: "Recharge for the week ahead" },
    6: { theme: "Rest", focus: "Recharge for the week ahead" }
  };

  const dayTheme = themedDays[today.getDay()];

  const subject = `☀️ ${dayName} - ${dayTheme.theme}`;

  const htmlBody = `
    <div style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;">

      <h1 style="color: #1a1a1a; margin-bottom: 5px;">Good morning, Clark</h1>
      <p style="color: #666; font-size: 18px; margin-top: 0;">It's ${dayName}. Today's theme: <strong>${dayTheme.theme}</strong></p>
      <p style="color: #888; font-size: 14px;">${dayTheme.focus}</p>

      <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;">

      <h2 style="color: #1a1a1a;">🎯 Before You Start</h2>

      <div style="background: #f8f9fa; padding: 20px; border-radius: 8px; margin-bottom: 20px;">
        <p style="margin: 0 0 15px 0; font-weight: bold;">What are your 3 must-dos today?</p>
        <p style="margin: 0 0 10px 0; color: #666;">1. ________________________________</p>
        <p style="margin: 0 0 10px 0; color: #666;">2. ________________________________</p>
        <p style="margin: 0; color: #666;">3. ________________________________</p>
      </div>

      <h3>✅ Morning Checklist</h3>
      <ul style="line-height: 2;">
        <li>☐ Written my 3 must-dos? (paper, next to screen)</li>
        <li>☐ Checked dashboard for red flags?</li>
        <li>☐ Quick inbox scan (5 min MAX - flag, don't reply)</li>
        <li>☐ Calendar reviewed - I know what's happening today</li>
        <li>☐ Phone on DND until first focus block done</li>
      </ul>

      ${calendarSection}

      <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;">

      <h3>🐰 Rabbithole Check</h3>
      <p style="color: #666;">When you notice drift, ask:</p>
      <ol style="color: #666; line-height: 1.8;">
        <li>Is this my #1 priority today?</li>
        <li>Will this matter in a week?</li>
        <li>Am I the only one who can do this?</li>
      </ol>
      <p style="color: #888; font-style: italic;">If any answer is "no" → capture it, return to plan.</p>

      <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;">

      <p style="color: #888; font-size: 12px;">This email was sent automatically. Edit the script at script.google.com to customize.</p>

    </div>
  `;

  MailApp.sendEmail({
    to: email,
    subject: subject,
    htmlBody: htmlBody
  });
}

// Optional: Send a test email right now
function testEmail() {
  sendMorningChecklist();
}
```

### 3. Save the Project

- Click **File → Save** (or Ctrl+S)
- Name it "Morning Checklist"

### 4. Test It First

1. Click the dropdown next to "Run" and select `testEmail`
2. Click **Run**
3. First time: Google will ask for permissions - click through and authorize
4. Check your inbox - you should receive the email

### 5. Set Up the Daily Trigger

1. Click the **clock icon** (Triggers) in the left sidebar
2. Click **+ Add Trigger**
3. Configure:
   - Function: `sendMorningChecklist`
   - Event source: `Time-driven`
   - Type: `Day timer`
   - Time of day: `8am to 9am`
4. Click **Save**

Done. You'll get this email every morning at ~8am.

---

## What the Email Includes

- Today's themed day (from your weekly rhythm)
- Space to write your 3 must-dos
- Morning checklist
- Today's calendar events (pulled automatically)
- Rabbithole exit reminder

---

## Customization Options

### Change the email address
Edit line 2: `const email = "clark@novela.academy";`

### Change the time
Edit the trigger (step 5) - pick a different time window

### Add HubSpot data
Requires HubSpot API - I can write this if you want. Would show:
- Deals closing this week
- Overdue follow-ups
- Pipeline value

### Add weather
Can integrate OpenWeather API if useful.

---

## Troubleshooting

**"Authorization required"**
Click through and allow access. The script needs permission to send email and read your calendar.

**Email not arriving**
- Check spam folder
- Check the trigger was created (clock icon → should show a trigger)
- Run `testEmail` manually to see if there's an error

**Wrong timezone**
Go to Project Settings (gear icon) → Set timezone to UK
