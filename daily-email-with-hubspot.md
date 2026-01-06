# Daily Morning Email - With HubSpot Integration

Enhanced version that pulls live data from HubSpot into your morning email.

---

## What You'll Get

Your 8am email will now include:

```
📊 HubSpot Snapshot
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pipeline Value: £45,230

Deals Closing This Week:
• Acme Corp - £12,000 (Proposal stage)
• Beta Ltd - £8,500 (Negotiation stage)

⚠️ Overdue Follow-ups (3):
• Call John at Acme - Due 2 days ago
• Send proposal to Beta - Due yesterday
• Check in with Gamma - Due 3 days ago
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Setup Steps

### Step 1: Get Your HubSpot API Key

1. Log into HubSpot
2. Click **Settings** (gear icon, top right)
3. Go to **Integrations → Private Apps**
4. Click **Create a private app**
5. Name it: "Morning Dashboard"
6. Go to **Scopes** tab and enable:
   - `crm.objects.deals.read`
   - `crm.objects.contacts.read`
   - `crm.objects.owners.read`
   - `sales-email-read` (for tasks)
7. Click **Create app**
8. Copy the access token (starts with `pat-...`)

**Keep this token private** - it gives access to your HubSpot data.

### Step 2: Create the Google Apps Script

1. Go to [script.google.com](https://script.google.com)
2. Click **New Project**
3. Delete existing code and paste the code below
4. Replace `YOUR_HUBSPOT_API_KEY` with your token from Step 1
5. Replace the email address with yours

### Step 3: Paste This Code

```javascript
// ============================================
// CONFIGURATION - Edit these values
// ============================================
const CONFIG = {
  email: "clark@novela.academy",
  hubspotApiKey: "YOUR_HUBSPOT_API_KEY", // paste your pat-... token here
  currency: "£",
  staleTaskDays: 0 // 0 = show overdue only, 1 = due today, etc.
};

// ============================================
// MAIN FUNCTION - Sends the morning email
// ============================================
function sendMorningChecklist() {
  const today = new Date();
  const dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
  const dayName = dayNames[today.getDay()];

  // Get calendar events
  const calendarHtml = getCalendarHtml(today);

  // Get themed day
  const dayTheme = getThemedDay(today.getDay());

  // Get HubSpot data
  const hubspotHtml = getHubSpotHtml();

  const subject = `☀️ ${dayName} - ${dayTheme.theme}`;

  const htmlBody = `
    <div style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; background: #ffffff;">

      <h1 style="color: #1a1a1a; margin-bottom: 5px;">Good morning, Clark</h1>
      <p style="color: #666; font-size: 18px; margin-top: 0;">It's ${dayName}. Today's theme: <strong>${dayTheme.theme}</strong></p>
      <p style="color: #888; font-size: 14px;">${dayTheme.focus}</p>

      <hr style="border: none; border-top: 1px solid #eee; margin: 20px 0;">

      ${hubspotHtml}

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

      ${calendarHtml}

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

      <p style="color: #888; font-size: 12px;">
        <a href="https://app.hubspot.com" style="color: #888;">Open HubSpot</a> |
        <a href="https://calendar.google.com" style="color: #888;">Open Calendar</a> |
        <a href="https://script.google.com" style="color: #888;">Edit this email</a>
      </p>

    </div>
  `;

  MailApp.sendEmail({
    to: CONFIG.email,
    subject: subject,
    htmlBody: htmlBody
  });
}

// ============================================
// HUBSPOT FUNCTIONS
// ============================================

function getHubSpotHtml() {
  try {
    const pipelineValue = getPipelineValue();
    const dealsClosingThisWeek = getDealsClosingThisWeek();
    const overdueTasks = getOverdueTasks();

    let html = `
      <div style="background: #f0f4ff; padding: 20px; border-radius: 8px; border-left: 4px solid #4a6cf7;">
        <h2 style="color: #1a1a1a; margin-top: 0;">📊 HubSpot Snapshot</h2>

        <p style="font-size: 24px; margin: 10px 0;">
          <strong>Pipeline Value:</strong> ${CONFIG.currency}${formatNumber(pipelineValue)}
        </p>
    `;

    // Deals closing this week
    if (dealsClosingThisWeek.length > 0) {
      html += `<h3 style="margin-bottom: 10px;">Deals Closing This Week:</h3><ul style="margin: 0; padding-left: 20px;">`;
      dealsClosingThisWeek.forEach(deal => {
        html += `<li style="margin-bottom: 5px;"><strong>${deal.name}</strong> - ${CONFIG.currency}${formatNumber(deal.amount)} (${deal.stage})</li>`;
      });
      html += `</ul>`;
    } else {
      html += `<p style="color: #666;">No deals closing this week.</p>`;
    }

    // Overdue tasks
    if (overdueTasks.length > 0) {
      html += `
        <h3 style="color: #dc3545; margin-bottom: 10px;">⚠️ Overdue Follow-ups (${overdueTasks.length}):</h3>
        <ul style="margin: 0; padding-left: 20px;">
      `;
      overdueTasks.slice(0, 5).forEach(task => { // Show max 5
        html += `<li style="margin-bottom: 5px; color: #dc3545;">${task.subject} - Due ${task.dueText}</li>`;
      });
      if (overdueTasks.length > 5) {
        html += `<li style="color: #666;">...and ${overdueTasks.length - 5} more</li>`;
      }
      html += `</ul>`;
    } else {
      html += `<p style="color: #28a745;">✓ No overdue follow-ups!</p>`;
    }

    html += `
        <p style="margin-top: 15px; margin-bottom: 0;">
          <a href="https://app.hubspot.com/contacts/deals" style="color: #4a6cf7; text-decoration: none;">Open HubSpot Deals →</a>
        </p>
      </div>
    `;

    return html;

  } catch (error) {
    console.error("HubSpot error:", error);
    return `
      <div style="background: #fff3cd; padding: 20px; border-radius: 8px; border-left: 4px solid #ffc107;">
        <h3 style="margin-top: 0;">⚠️ Couldn't load HubSpot data</h3>
        <p style="color: #666; margin-bottom: 0;">Error: ${error.message}</p>
        <p style="color: #888; font-size: 12px;">Check your API key in the script settings.</p>
      </div>
    `;
  }
}

function getPipelineValue() {
  const url = "https://api.hubapi.com/crm/v3/objects/deals?limit=100&properties=amount,dealstage";
  const response = callHubSpotApi(url);

  let total = 0;
  if (response && response.results) {
    response.results.forEach(deal => {
      const amount = parseFloat(deal.properties.amount) || 0;
      // Only count open deals (not won or lost)
      const stage = deal.properties.dealstage || "";
      if (!stage.toLowerCase().includes("won") && !stage.toLowerCase().includes("lost") && !stage.toLowerCase().includes("closed")) {
        total += amount;
      }
    });
  }
  return total;
}

function getDealsClosingThisWeek() {
  const today = new Date();
  const endOfWeek = new Date(today);
  endOfWeek.setDate(today.getDate() + (7 - today.getDay())); // End of this week (Sunday)

  const url = "https://api.hubapi.com/crm/v3/objects/deals?limit=100&properties=dealname,amount,closedate,dealstage";
  const response = callHubSpotApi(url);

  const deals = [];
  if (response && response.results) {
    response.results.forEach(deal => {
      const closeDate = deal.properties.closedate ? new Date(deal.properties.closedate) : null;
      if (closeDate && closeDate >= today && closeDate <= endOfWeek) {
        deals.push({
          name: deal.properties.dealname || "Unnamed deal",
          amount: parseFloat(deal.properties.amount) || 0,
          stage: deal.properties.dealstage || "Unknown stage",
          closeDate: closeDate
        });
      }
    });
  }

  // Sort by close date
  deals.sort((a, b) => a.closeDate - b.closeDate);
  return deals;
}

function getOverdueTasks() {
  const today = new Date();
  today.setHours(0, 0, 0, 0);

  // Get tasks/engagements
  const url = "https://api.hubapi.com/crm/v3/objects/tasks?limit=100&properties=hs_task_subject,hs_task_status,hs_timestamp";
  const response = callHubSpotApi(url);

  const tasks = [];
  if (response && response.results) {
    response.results.forEach(task => {
      const status = task.properties.hs_task_status || "";
      const dueDate = task.properties.hs_timestamp ? new Date(parseInt(task.properties.hs_timestamp)) : null;

      // Only include incomplete tasks that are overdue
      if (dueDate && dueDate < today && status !== "COMPLETED") {
        const daysOverdue = Math.floor((today - dueDate) / (1000 * 60 * 60 * 24));
        tasks.push({
          subject: task.properties.hs_task_subject || "Untitled task",
          dueDate: dueDate,
          dueText: daysOverdue === 1 ? "yesterday" : `${daysOverdue} days ago`
        });
      }
    });
  }

  // Sort by most overdue first
  tasks.sort((a, b) => a.dueDate - b.dueDate);
  return tasks;
}

function callHubSpotApi(url) {
  const options = {
    method: "GET",
    headers: {
      "Authorization": "Bearer " + CONFIG.hubspotApiKey,
      "Content-Type": "application/json"
    },
    muteHttpExceptions: true
  };

  const response = UrlFetchApp.fetch(url, options);
  const responseCode = response.getResponseCode();

  if (responseCode !== 200) {
    throw new Error(`HubSpot API returned ${responseCode}: ${response.getContentText()}`);
  }

  return JSON.parse(response.getContentText());
}

// ============================================
// CALENDAR FUNCTIONS
// ============================================

function getCalendarHtml(today) {
  const calendar = CalendarApp.getDefaultCalendar();
  const events = calendar.getEventsForDay(today);

  let html = "";
  if (events.length > 0) {
    html = "<h3>📅 Today's Calendar</h3><ul style='line-height: 1.8;'>";
    events.forEach(event => {
      const startTime = event.getStartTime();
      const timeStr = Utilities.formatDate(startTime, Session.getScriptTimeZone(), "HH:mm");
      html += `<li><strong>${timeStr}</strong> - ${event.getTitle()}</li>`;
    });
    html += "</ul>";
  } else {
    html = "<h3>📅 Today's Calendar</h3><p style='color: #28a745;'>No meetings scheduled - protect this focus time!</p>";
  }
  return html;
}

// ============================================
// HELPER FUNCTIONS
// ============================================

function getThemedDay(dayNumber) {
  const themedDays = {
    1: { theme: "Strategy & Planning", focus: "High-level thinking, investor prep, set the week's priorities" },
    2: { theme: "Sales & Outreach", focus: "HubSpot deep dive, prospecting, follow-ups" },
    3: { theme: "Customers & Product", focus: "Customer check-ins, classes, content work" },
    4: { theme: "Marketing & Comms", focus: "LinkedIn posts, newsletter, messaging strategy" },
    5: { theme: "Admin & Reflection", focus: "Receipts, Revolut, CEO review, week wrap-up" },
    0: { theme: "Rest", focus: "Recharge for the week ahead" },
    6: { theme: "Rest", focus: "Recharge for the week ahead" }
  };
  return themedDays[dayNumber];
}

function formatNumber(num) {
  return num.toLocaleString('en-GB', { minimumFractionDigits: 0, maximumFractionDigits: 0 });
}

// ============================================
// TEST FUNCTIONS
// ============================================

function testEmail() {
  sendMorningChecklist();
}

function testHubSpotConnection() {
  try {
    const pipeline = getPipelineValue();
    console.log("Pipeline value: " + CONFIG.currency + pipeline);

    const deals = getDealsClosingThisWeek();
    console.log("Deals closing this week: " + deals.length);

    const tasks = getOverdueTasks();
    console.log("Overdue tasks: " + tasks.length);

    console.log("✓ HubSpot connection working!");
  } catch (error) {
    console.error("✗ HubSpot connection failed: " + error.message);
  }
}
```

### Step 4: Test the HubSpot Connection

1. Click the dropdown next to "Run" and select `testHubSpotConnection`
2. Click **Run**
3. Click **View → Logs** to see the output
4. Should show your pipeline value and task counts

### Step 5: Send a Test Email

1. Select `testEmail` from the dropdown
2. Click **Run**
3. Check your inbox

### Step 6: Set Up Daily Trigger

1. Click the **clock icon** (Triggers)
2. Click **+ Add Trigger**
3. Configure:
   - Function: `sendMorningChecklist`
   - Event source: `Time-driven`
   - Type: `Day timer`
   - Time: `8am to 9am`
4. Click **Save**

---

## Troubleshooting

### "HubSpot API returned 401"
Your API key is wrong or expired. Generate a new one in HubSpot → Settings → Integrations → Private Apps.

### "HubSpot API returned 403"
Missing permissions. Edit your private app and add the required scopes (see Step 1).

### Pipeline value shows £0
- Check you have deals in HubSpot with amounts filled in
- Make sure deals aren't marked as "Closed Won" or "Closed Lost"

### No overdue tasks showing
- Tasks need to have a due date set in HubSpot
- Check you have incomplete tasks with past due dates

### "Couldn't load HubSpot data" in email
Run `testHubSpotConnection` and check the logs for the specific error.

---

## Customization

### Change currency
Edit line 6: `currency: "£"` → `currency: "$"` or `currency: "€"`

### Show tasks due today (not just overdue)
Edit line 7: `staleTaskDays: 0` → `staleTaskDays: 1`

### Add more deal properties
In `getDealsClosingThisWeek()`, add properties to the URL:
```javascript
const url = "...&properties=dealname,amount,closedate,dealstage,hubspot_owner_id";
```

### Change deal stages to exclude
In `getPipelineValue()`, edit the stage filter:
```javascript
if (!stage.includes("closedwon") && !stage.includes("closedlost")) {
```

---

## Security Note

Your HubSpot API key is stored in the script. This is fine for personal use, but:
- Don't share the script with the key in it
- If you think the key is compromised, regenerate it in HubSpot
- The key only has read access (can't modify your HubSpot data)
