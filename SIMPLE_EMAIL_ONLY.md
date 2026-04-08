# Simple Email-Only Approach
## Immediate Reminder When Workday Email Arrives

---

## 🎯 What This Does

**Single Flow:** When a Workday training email arrives → Parse it → Send immediate reminder to user

**No storage, no tracking, no complexity** - just instant notification forwarding.

---

## ⚡ Quick Setup (15 minutes)

### Step 1: Create the Flow

1. **Go to Power Automate**
   - Navigate to https://make.powerautomate.com
   - Sign in with your company account

2. **Create automated flow**
   ```
   Click "Create" → "Automated cloud flow"
   Flow name: "Workday Training - Instant Reminder"
   Trigger: "When a new email arrives (V3)" - Office 365 Outlook
   Click "Create"
   ```

3. **Configure trigger**
   ```
   Folder: Inbox
   From: workday@yourcompany.com
   Subject Filter: (leave blank)
   Include Attachments: No
   ```

---

### Step 2: Add Email Filter

```
Add action → "Condition"

Configure:
- Subject contains "Training"
  OR
- Subject contains "Certification"
  OR
- Subject contains "Learning"
```

---

### Step 3: Parse Email Content

**In the "If yes" branch:**

1. **Initialize variable: TrainingName**
   ```
   Add action → "Initialize variable"
   Name: TrainingName
   Type: String
   Value: [Expression]
   if(
     contains(triggerOutputs()?['body/body'], 'Course:'),
     substring(
       triggerOutputs()?['body/body'],
       add(indexOf(triggerOutputs()?['body/body'], 'Course:'), 8),
       100
     ),
     triggerOutputs()?['subject']
   )
   ```

2. **Initialize variable: DueDate**
   ```
   Add action → "Initialize variable"
   Name: DueDate
   Type: String
   Value: [Expression]
   if(
     contains(triggerOutputs()?['body/body'], 'Due Date:'),
     substring(
       triggerOutputs()?['body/body'],
       add(indexOf(triggerOutputs()?['body/body'], 'Due Date:'), 10),
       10
     ),
     'Not specified'
   )
   ```

3. **Initialize variable: ActionLink**
   ```
   Add action → "Initialize variable"
   Name: ActionLink
   Type: String
   Value: [Expression]
   if(
     contains(triggerOutputs()?['body/body'], 'https://'),
     substring(
       triggerOutputs()?['body/body'],
       indexOf(triggerOutputs()?['body/body'], 'https://'),
       100
     ),
     'https://workday.yourcompany.com'
   )
   ```

---

### Step 4: Send Immediate Reminder

**Option A: Teams Notification**

```
Add action → "Post message in a chat or channel"

Configure:
- Post as: Flow bot
- Post in: Chat with Flow bot
- Recipient: [Dynamic] To (from email trigger)
- Message:
  📚 Training Reminder
  
  Training: [Dynamic] TrainingName
  Due Date: [Dynamic] DueDate
  
  Complete now: [Dynamic] ActionLink
```

**Option B: Outlook Email**

```
Add action → "Send an email (V2)"

Configure:
- To: [Dynamic] To (from email trigger)
- Subject: Reminder: [Dynamic] TrainingName
- Body:
  Hi,
  
  You have been assigned a training:
  
  Training: [Dynamic] TrainingName
  Due Date: [Dynamic] DueDate
  
  Click here to complete: [Dynamic] ActionLink
  
  This is an automated reminder.
- Importance: Normal
```

**Option C: Both Teams AND Email**

Add both actions above - they'll run in parallel.

---

### Step 5: Save and Test

```
1. Click "Save"
2. Send yourself a test email with:
   - From: workday@yourcompany.com (or have someone forward one)
   - Subject: "Action Required: Complete Security Training"
   - Body: Include "Course:", "Due Date:", and a link
3. Check flow run history
4. Verify you received the reminder
```

---

## 📋 Complete Flow Structure

```
┌─────────────────────────────────────────┐
│  Trigger: When email arrives            │
│  From: workday@yourcompany.com          │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Condition: Subject contains            │
│  "Training" OR "Certification"          │
└─────────────────────────────────────────┘
                  ↓
        ┌─────────────────┐
        │     If YES      │
        └─────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Parse Email:                           │
│  - Extract Training Name                │
│  - Extract Due Date                     │
│  - Extract Action Link                  │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Send Reminder:                         │
│  - Teams message                        │
│  - OR Outlook email                     │
│  - OR Both                              │
└─────────────────────────────────────────┘
                  ↓
              ✅ DONE
```

---

## ✅ What You Get

- ✅ **Instant notification** when Workday email arrives
- ✅ **Parsed training details** in clean format
- ✅ **Action link** for quick access
- ✅ **No storage needed** - super simple
- ✅ **No maintenance** - just works

---

## ❌ What You DON'T Get

- ❌ No daily reminders
- ❌ No tracking of completion
- ❌ No escalation alerts (T-3, T-1 days)
- ❌ No weekly summaries
- ❌ No status tracking (pending/completed/overdue)
- ❌ No prevention of duplicate notifications
- ❌ No manager dashboards

---

## 🔧 Customization Options

### Change Notification Timing

**Add a delay before sending:**
```
After parsing, before sending reminder:
Add action → "Delay"
- Count: 1
- Unit: Hour

This sends reminder 1 hour after email arrives instead of immediately
```

### Add More Recipients

**CC your manager:**
```
In "Send an email" action:
- To: [Dynamic] To (original recipient)
- CC: manager@company.com
```

### Customize Message

**Make it more friendly:**
```
Message:
👋 Hey there!

You've got a new training assignment:

📚 Training: [TrainingName]
📅 Due: [DueDate]
🔗 Start here: [ActionLink]

Don't forget to complete it on time! 🎯
```

---

## 🆘 Troubleshooting

### Flow Not Triggering
- Check "From" address matches exactly
- Verify flow is turned ON
- Check flow run history for errors

### Variables Not Parsing
- Email format might be different
- Check actual email content
- Adjust indexOf positions in expressions

### Reminder Not Received
- Check Teams permissions
- Verify email address
- Check spam/junk folder

---

## 💡 When to Upgrade

**Consider adding storage if you need:**
- Daily reminders (not just one-time)
- Tracking who completed what
- Preventing duplicate notifications
- Manager reports
- Escalation before deadlines

**If you need those features, use the full solution in `IMPORT_GUIDE.md`**

---

## 🎯 Perfect For

- ✅ Small teams (< 20 people)
- ✅ Infrequent trainings (1-2 per month)
- ✅ Users who are self-motivated
- ✅ Quick setup needed
- ✅ Minimal maintenance desired

---

## ⚡ Pro Tips

1. **Test with your own email first** before rolling out
2. **Adjust parsing patterns** based on your actual Workday emails
3. **Use Teams for instant visibility** - harder to miss than email
4. **Add emoji** to make notifications stand out
5. **Keep it simple** - this is the beauty of this approach

---

## 📊 Comparison

| Feature | Email-Only | Full Solution |
|---------|-----------|---------------|
| Setup time | 15 min | 90 min |
| Storage needed | No | Yes |
| Daily reminders | No | Yes |
| Completion tracking | No | Yes |
| Escalation alerts | No | Yes |
| Manager reports | No | Yes |
| Maintenance | None | Low |
| Best for | Small teams | Organizations |

---

**Ready to build?** Follow the steps above and you'll have it running in 15 minutes!
