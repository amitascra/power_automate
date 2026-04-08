# Quick Start Guide
## Workday Learning & Certification Tracker

Get up and running in 30 minutes with this streamlined setup guide.

---

## 🎯 What You'll Build

An automated system that:
- ✅ Monitors Workday training emails
- ✅ Extracts training details automatically
- ✅ Sends daily reminders via Teams & Outlook
- ✅ Escalates alerts before deadlines
- ✅ Tracks completion status

---

## 📋 Prerequisites (5 minutes)

### Required Access
- [ ] Microsoft 365 account with Power Automate
- [ ] Outlook mailbox access
- [ ] Microsoft Teams access
- [ ] SharePoint site (or OneDrive for Excel option)

### Sample Data Needed
- [ ] 1-2 sample Workday training emails
- [ ] Workday sender email address

---

## 🚀 Setup Steps

### Step 1: Create Data Storage (5 minutes)

**Option A: SharePoint List (Recommended)**
```
1. Go to your SharePoint site
2. Click "New" → "List" → "Blank list"
3. Name: "Training Tracker"
4. Add these columns:
   - UserEmail (Single line text)
   - TrainingName (Single line text)
   - DueDate (Date)
   - Status (Choice: Pending, Due Soon, Overdue, Completed)
   - ActionLink (Hyperlink)
   - LastReminderSent (Date and Time)
   - CreatedDate (Date and Time)
5. Click "Save"
```

**Option B: Excel Online (Simpler)**
```
1. Create new Excel file in OneDrive
2. Name: "Training_Tracker.xlsx"
3. Add headers: UserEmail | TrainingName | DueDate | Status | ActionLink | LastReminderSent | CreatedDate
4. Format as Table (Ctrl+T)
5. Name table: "TrainingData"
6. Save
```

---

### Step 2: Create Email Processing Flow (10 minutes)

```
1. Go to https://make.powerautomate.com
2. Click "Create" → "Automated cloud flow"
3. Name: "Workday Training - Email Processor"
4. Search for "When a new email arrives"
5. Click "Create"

Configure Trigger:
- Folder: Inbox
- From: [your-workday-sender@company.com]
- Leave other fields blank

Add Action - Condition:
- Subject contains "Training"

Add Action - Initialize variable (4 times):
- EmailBody: @{triggerOutputs()?['body/body']}
- TrainingName: Extract from email (see parser logic)
- DueDate: Extract from email
- ActionLink: Extract from email

Add Action - Create item (SharePoint) or Add row (Excel):
- Fill in extracted values
- Set Status: "Pending"
- Set CreatedDate: @{utcNow()}

Save and Test with sample email
```

---

### Step 3: Create Daily Reminder Flow (10 minutes)

```
1. Click "Create" → "Scheduled cloud flow"
2. Name: "Workday Training - Daily Reminders"
3. Recurrence: Daily at 9:00 AM
4. Click "Create"

Add Action - Get items (SharePoint) or List rows (Excel):
- Filter: Status eq 'Pending' or Status eq 'Due Soon'

Add Action - Apply to each:
- Select: value from previous step

Inside loop:
  1. Calculate days remaining
  2. Determine status (Overdue/Due Soon/Pending)
  3. Send Teams notification (Post adaptive card)
  4. Send Outlook email
  5. Update LastReminderSent

Save and Test
```

---

### Step 4: Test End-to-End (5 minutes)

```
1. Send yourself a test email matching Workday format:
   Subject: "Action Required: Complete Security Training"
   Body: Include "Course:", "Due Date:", and a link

2. Check flow run history (should trigger within 5 min)

3. Verify data stored in SharePoint/Excel

4. Manually trigger Daily Reminder flow

5. Check Teams and Outlook for notifications
```

---

## 🎨 Customization

### Change Reminder Time
```
Edit Daily Reminder flow → Recurrence trigger → Change time
```

### Adjust Escalation Days
```
Edit Daily Reminder flow → Conditions:
- Change "DaysRemaining <= 3" to your preference
- Change "DaysRemaining <= 1" to your preference
```

### Customize Notification Templates
```
Edit files in templates/ folder:
- teams-notification.json (Teams adaptive card)
- outlook-reminder.html (Email template)
```

---

## 📊 Monitor & Maintain

### Daily Checks (First Week)
```
1. Power Automate → My flows
2. Check "28 day run history"
3. Look for failed runs (red X)
4. Click to see error details
```

### Weekly Review
```
1. Open SharePoint list or Excel file
2. Review data accuracy
3. Check for parsing errors
4. Adjust patterns if needed
```

---

## 🆘 Troubleshooting

### Flow Not Triggering
```
Problem: Email arrives but flow doesn't run
Solution:
- Check "From" address matches exactly
- Verify mailbox permissions
- Check flow is turned ON
```

### Parsing Errors
```
Problem: Training name or date not extracted
Solution:
- Review sample email format
- Check parsers/email-parser-logic.md
- Adjust regex patterns
- Add fallback values
```

### Notifications Not Received
```
Problem: Reminders not appearing in Teams/Outlook
Solution:
- Check Teams permissions
- Verify email addresses
- Check spam/junk folders
- Review connector limits
```

---

## 📚 Next Steps

### Optional Enhancements
1. **Weekly Summary Flow** - See `flows/main-flow-blueprint.md`
2. **Completion Detection** - Auto-update status from completion emails
3. **Manager Dashboard** - Power BI report for team visibility
4. **Calendar Integration** - Add training deadlines to Outlook calendar

### Full Documentation
- `README.md` - Complete project overview
- `flows/main-flow-blueprint.md` - Detailed flow architecture
- `parsers/email-parser-logic.md` - Email parsing patterns
- `docs/technical-assumptions.md` - Constraints and assumptions
- `docs/deployment-guide.md` - Production deployment guide

---

## 💡 Tips for Success

1. **Start Small**: Test with 5-10 users before full rollout
2. **Monitor Closely**: Check daily for first week
3. **Gather Feedback**: Ask users about timing and content
4. **Iterate**: Adjust based on real usage patterns
5. **Document**: Keep notes on customizations made

---

## ✅ Success Criteria

You'll know it's working when:
- ✅ Emails are parsed correctly (90%+ accuracy)
- ✅ Reminders arrive on time
- ✅ Users can click action links successfully
- ✅ Status updates automatically on completion
- ✅ No manual tracking needed

---

## 🎓 Learning Resources

- [Power Automate Documentation](https://docs.microsoft.com/power-automate/)
- [SharePoint Lists Guide](https://support.microsoft.com/sharepoint)
- [Teams Adaptive Cards](https://adaptivecards.io/)
- [Power Automate Community](https://powerusers.microsoft.com/t5/Power-Automate-Community/ct-p/MPACommunity)

---

**Need Help?** Check the full documentation or contact your Power Platform admin.
