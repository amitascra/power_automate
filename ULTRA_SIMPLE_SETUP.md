# Ultra Simple Setup - Training Name + URL Only
## Accenture Workday Training Notifications

---

## 🎯 What This Does

When Workday enrollment email arrives → Extract training name and URL → Send simple notification: "You are enrolled in [Training]"

**That's it. No due dates, no complexity.**

---

## ⚡ 10-Minute Setup

### Step 1: Create Flow

1. Go to https://make.powerautomate.com
2. Click **"Create"** → **"Automated cloud flow"**
3. Name: `Workday Training Reminder`
4. Trigger: **"When a new email arrives (V3)"**
5. Click **"Create"**

---

### Step 2: Configure Trigger

```
Folder: Inbox
From: accenture@myworkday.com
Subject Filter: (leave blank)
Include Attachments: No
```

---

### Step 3: Add Condition

Click **"+ New step"** → Search **"Condition"**

```
Condition:
Body contains "enrolled in"
```

---

### Step 4: Extract Training Name (If Yes Branch)

Click **"Add an action"** → Search **"Initialize variable"**

```
Name: TrainingName
Type: String
Value: [Click "Expression" tab]
```

**Paste this expression:**
```javascript
trim(first(split(substring(triggerOutputs()?['body/body'], add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12), 200), '.')))
```

**What it does:** Grabs text after "enrolled in" until the first period

---

### Step 5: Extract URL

Click **"Add an action"** → Search **"Initialize variable"**

```
Name: TrainingURL
Type: String
Value: [Click "Expression" tab]
```

**Paste this expression:**
```javascript
if(contains(triggerOutputs()?['body/body'], 'https://'), substring(triggerOutputs()?['body/body'], indexOf(triggerOutputs()?['body/body'], 'https://'), 100), 'https://workday.accenture.com')
```

**What it does:** Finds first https:// link in email

---

### Step 6: Send Notification

**Option A: Teams Message (Recommended)**

Click **"Add an action"** → Search **"Post message in a chat or channel"**

```
Post as: Flow bot
Post in: Chat with Flow bot
Recipient: [Dynamic content] → To (from email trigger)

Message:
📚 Training Enrollment

You are enrolled in:
[Dynamic content] → TrainingName

View details:
[Dynamic content] → TrainingURL
```

**Option B: Email**

Click **"Add an action"** → Search **"Send an email (V2)"**

```
To: [Dynamic content] → To
Subject: Training Enrollment Notification

Body:
You are enrolled in: [Dynamic content] → TrainingName

View details: [Dynamic content] → TrainingURL
```

---

### Step 7: Save and Test

1. Click **"Save"**
2. Forward yourself a Workday training email
3. Check flow run history
4. Verify you got the notification

---

## 📋 Complete Flow Structure

```
┌────────────────────────────────┐
│ Trigger: Email arrives         │
│ From: accenture@myworkday.com  │
└────────────────────────────────┘
              ↓
┌────────────────────────────────┐
│ Condition:                     │
│ Body contains "enrolled in"    │
└────────────────────────────────┘
              ↓
┌────────────────────────────────┐
│ Extract TrainingName           │
│ (text after "enrolled in")     │
└────────────────────────────────┘
              ↓
┌────────────────────────────────┐
│ Extract TrainingURL            │
│ (first https:// link)          │
└────────────────────────────────┘
              ↓
┌────────────────────────────────┐
│ Send Notification:             │
│ "You are enrolled in [Name]"   │
│ Link: [URL]                    │
└────────────────────────────────┘
```

---

## ✅ Example

**Input Email:**
```
You are now enrolled in Amazon Q Developer (Recommended for Custom Engineers).
Click here: https://in.accenture.com/mylearninghelp/contacts
```

**Variables Extracted:**
```
TrainingName: "Amazon Q Developer (Recommended for Custom Engineers)"
TrainingURL: "https://in.accenture.com/mylearninghelp/contacts"
```

**Notification Sent:**
```
📚 Training Enrollment

You are enrolled in:
Amazon Q Developer (Recommended for Custom Engineers)

View details:
https://in.accenture.com/mylearninghelp/contacts
```

---

## 🎨 Customize Your Message

### Simple Version
```
You are enrolled in: [TrainingName]
Link: [TrainingURL]
```

### Friendly Version
```
👋 Hi!

Good news - you're enrolled in:
[TrainingName]

Get started here: [TrainingURL]
```

### Professional Version
```
Training Enrollment Notification

Course: [TrainingName]
Access: [TrainingURL]

Please complete this training at your earliest convenience.
```

---

## 🔧 Copy-Paste Ready Expressions

### Training Name
```javascript
trim(first(split(substring(triggerOutputs()?['body/body'], add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12), 200), '.')))
```

### Training URL
```javascript
if(contains(triggerOutputs()?['body/body'], 'https://'), substring(triggerOutputs()?['body/body'], indexOf(triggerOutputs()?['body/body'], 'https://'), 100), 'https://workday.accenture.com')
```

---

## 🆘 Troubleshooting

### Training Name is Wrong
- Check your email says "enrolled in"
- If it says something else, change the expression
- Example: Change `'enrolled in'` to `'assigned to'`

### URL Not Found
- Email might not have https:// link
- Fallback URL will be used: `https://workday.accenture.com`
- Check email has a clickable link

### Notification Not Received
- Check flow is turned ON
- Verify Teams/Email permissions
- Check spam/junk folder

---

## 💡 That's It!

**Total actions in flow:** 5
1. Trigger (email arrives)
2. Condition (check for "enrolled in")
3. Extract training name
4. Extract URL
5. Send notification

**No storage, no due dates, no complexity.**

---

## 📊 What You Get vs Don't Get

### ✅ You Get
- Instant notification when enrolled
- Training name extracted
- Direct link to training
- Simple, clean message

### ❌ You Don't Get
- No due date tracking
- No daily reminders
- No completion tracking
- No weekly summaries
- No storage/database

**Perfect for:** Quick notifications, self-motivated users, minimal setup

---

**Setup time: 10 minutes | Maintenance: Zero**
