# Zapier Automation: Gmail Receipts → Google Drive

This automation saves receipts from Gmail to a Drive folder automatically. Runs without you thinking about it.

---

## What You'll Build

```
Gmail (new email matching search)
    → Google Drive (upload attachment)
    → Optional: Slack/Email notification
```

**Time to set up:** 15-20 minutes
**Zapier plan needed:** Free tier works (100 tasks/month), but Starter ($29/mo) recommended for volume

---

## Prerequisites

- Zapier account ([zapier.com](https://zapier.com))
- Gmail account (the one receiving receipts)
- Google Drive folder created for receipts

---

## Step 1: Create Your Drive Folder Structure

In Google Drive, create:

```
Novela/
└── Finance/
    └── Receipts/
        └── 2024/
            └── Auto-Imported/
```

The "Auto-Imported" folder is where Zapier will drop files. You (or your VA) can sort them later.

---

## Step 2: Create the Zap

### 2.1 Start a New Zap

1. Go to [zapier.com](https://zapier.com) → Create → Zaps
2. Click "Create Zap"

### 2.2 Set Up the Trigger (Gmail)

1. **App:** Search for "Gmail"
2. **Event:** Select "New Attachment"
3. **Account:** Connect your Gmail account (authorize access)
4. **Search String:** This is the key part. Use:

```
from:(receipts OR invoice OR billing OR payment) OR subject:(receipt OR invoice OR order confirmation OR payment)
```

Or more specific senders you know send receipts:

```
from:(stripe.com OR amazon.co.uk OR revolut.com OR notion.so OR slack.com OR zoom.us OR xero.com OR freeagent.com)
```

5. **Test trigger:** Zapier will find a recent matching email to test with

### 2.3 Set Up the Action (Google Drive)

1. **App:** Search for "Google Drive"
2. **Event:** Select "Upload File"
3. **Account:** Connect your Google Drive account
4. **Drive:** My Google Drive
5. **Folder:** Navigate to: Novela → Finance → Receipts → 2024 → Auto-Imported
6. **File:** Select the attachment from the Gmail trigger (dropdown will show it)
7. **File Name:** Use this format to auto-name files:

```
{{zap_meta_human_now}} - {{from_name}} - {{attachment_file_name}}
```

This creates: `2024-03-15 - Stripe - invoice_12345.pdf`

Or for cleaner names:
```
{{zap_meta_human_now}}_{{from_name}}
```

8. **Convert to Document:** Leave as "No"

### 2.4 Test the Zap

1. Click "Test action"
2. Check your Drive folder - the test file should appear
3. Verify the file name format looks right

### 2.5 Turn It On

1. Name your Zap: "Gmail Receipts → Drive"
2. Click "Publish" or "Turn on Zap"

---

## Step 3: Optional Enhancements

### Add a Slack Notification

After the Google Drive step, add another action:

1. **App:** Slack
2. **Event:** Send Channel Message
3. **Channel:** #receipts or your personal DM
4. **Message:**
```
📧 New receipt saved to Drive
From: {{from_name}}
Subject: {{subject}}
File: {{attachment_file_name}}
```

### Add to a Google Sheet Log

Track all receipts in a spreadsheet:

1. **App:** Google Sheets
2. **Event:** Create Spreadsheet Row
3. **Spreadsheet:** "Receipt Log"
4. **Columns:**
   - Date: `{{zap_meta_human_now}}`
   - From: `{{from_name}}`
   - Subject: `{{subject}}`
   - Amount: (leave blank - manual or VA fills in)
   - Category: (leave blank)
   - Uploaded to FreeAgent: No

---

## Alternative: Multi-Step Zap for Better Organization

For more control, use a Zap with filters:

```
Gmail (New Attachment)
    → Filter (only if attachment is PDF or image)
    → Formatter (extract date, clean filename)
    → Google Drive (upload)
    → Google Sheets (log it)
    → Slack (notify)
```

This requires Zapier Starter plan ($29/mo) for multi-step Zaps.

---

## Recommended Gmail Search Strings

### Catch-All (Broad)
```
has:attachment (receipt OR invoice OR payment OR order confirmation) newer_than:1d
```

### Known Senders (Precise)
```
has:attachment from:(stripe.com OR paypal.com OR amazon.co.uk OR xero.com OR gocardless.com OR wise.com OR revolut.com)
```

### SaaS Subscriptions
```
has:attachment from:(notion.so OR slack.com OR zoom.us OR figma.com OR github.com OR vercel.com OR aws.amazon.com OR google.com OR microsoft.com)
```

### Combine Them
Create multiple Zaps with different search strings if one gets too complex.

---

## Limitations & Workarounds

| Limitation | Workaround |
|------------|------------|
| Zapier free tier: 100 tasks/month | Upgrade, or be specific with search strings |
| Only catches emails with attachments | Some receipts are inline HTML - these need manual handling |
| Doesn't upload to FreeAgent | FreeAgent doesn't have a Zapier integration. VA still uploads manually, but at least files are in Drive |
| Won't categorise receipts | Use Google Sheets log + VA reviews weekly |

---

## If You Want Full Automation to FreeAgent

FreeAgent doesn't have native Zapier support, but you can:

### Option A: Use Dext (formerly Receipt Bank)
1. Sign up for [Dext](https://dext.com) (~£24/mo)
2. Forward receipts to your Dext email address
3. Dext extracts data and pushes to FreeAgent automatically

**Zapier flow:**
```
Gmail (new receipt) → Gmail (forward email to Dext address)
```

### Option B: Use AutoEntry
Similar to Dext, integrates with FreeAgent.

### Option C: Manual with VA
Keep the Zapier → Drive flow. VA uploads from Drive to FreeAgent weekly (30 mins).

---

## Quick Setup Checklist

- [ ] Create Drive folder: Novela/Finance/Receipts/2024/Auto-Imported
- [ ] Sign up for Zapier (free to start)
- [ ] Create Zap: Gmail → Google Drive
- [ ] Set Gmail search string (start with known senders)
- [ ] Test with a real receipt email
- [ ] Turn on Zap
- [ ] Optional: Add Slack notification
- [ ] Optional: Add Google Sheet log
- [ ] Tell VA: "Check Auto-Imported folder weekly, upload to FreeAgent"

---

## Time Saved

| Task | Before | After |
|------|--------|-------|
| Finding receipts in Gmail | 30 mins/week | 0 |
| Downloading attachments | 20 mins/week | 0 |
| Saving to Drive | 10 mins/week | 0 |
| Uploading to FreeAgent | 30 mins/week | 30 mins/week (VA) |

**Net savings:** ~1 hour/week of your time, assuming VA handles FreeAgent upload.
