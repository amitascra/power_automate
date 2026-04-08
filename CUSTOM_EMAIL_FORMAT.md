# Custom Email Format - Accenture Workday
## Updated Parsing for Your Actual Email Structure

---

## 📧 Your Email Format Analysis

Based on your screenshot, here's what your Workday emails look like:

```
From: Accenture Careers <accenture@myworkday.com>
To: YADAV, VANDANA
Subject: VANDANA YADAV

You are now enrolled in Amazon Q Developer (Recommended for Custom Engineers).

For questions about Workday, copy and paste this link into your browser to 
contact our Learning Hub and click on the "Support Helpdesk" link.

Click here to view the notification details.
[Link: https://in.accenture.com/mylearninghelp/contacts]

This is a valid Accenture email notification. If you have any questions, 
please visit workday.accenture.com directly to see this notification/inbox item.
```

---

## 🔍 Key Differences from Standard Format

### What We Extract:
1. **Training Name**: From the email body text (e.g., "Amazon Q Developer")
2. **Due Date**: NOT present in this email format ❌
3. **Action Link**: "Click here to view the notification details" link
4. **Recipient**: From subject line (employee name)

---

## ✅ Updated Parsing Expressions

### Variable 1: Training Name

**Extract from "enrolled in [TRAINING NAME]" pattern:**

```javascript
if(
  contains(triggerOutputs()?['body/body'], 'enrolled in'),
  trim(
    substring(
      triggerOutputs()?['body/body'],
      add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
      indexOf(
        triggerOutputs()?['body/body'],
        '.',
        add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12)
      )
    )
  ),
  triggerOutputs()?['subject']
)
```

**What this does:**
- Looks for "enrolled in" in the email
- Starts reading 12 characters after (skips "enrolled in ")
- Stops at the next period "."
- Result: "Amazon Q Developer (Recommended for Custom Engineers)"

**Alternative (simpler):**
```javascript
if(
  contains(triggerOutputs()?['body/body'], 'enrolled in'),
  substring(
    triggerOutputs()?['body/body'],
    add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
    100
  ),
  triggerOutputs()?['subject']
)
```

---

### Variable 2: Due Date

**Your emails DON'T have due dates, so use a placeholder:**

```javascript
'No due date specified - Check Workday for details'
```

**OR set a default 30 days:**

```javascript
formatDateTime(addDays(utcNow(), 30), 'MM/dd/yyyy')
```

---

### Variable 3: Action Link

**Extract the link after "Click here":**

```javascript
if(
  contains(triggerOutputs()?['body/body'], 'https://'),
  trim(
    substring(
      triggerOutputs()?['body/body'],
      indexOf(triggerOutputs()?['body/body'], 'https://'),
      200
    )
  ),
  'https://workday.accenture.com'
)
```

**What this does:**
- Finds "https://" in the email
- Grabs 200 characters (captures full URL)
- Result: "https://in.accenture.com/mylearninghelp/contacts"

---

### Variable 4: Employee Name (Bonus)

**Extract from subject line:**

```javascript
triggerOutputs()?['subject']
```

**Result:** "VANDANA YADAV"

---

## 🛠️ Complete Flow Setup for Your Email Format

### Step 1: Trigger Configuration

```
When a new email arrives (V3)
- Folder: Inbox
- From: accenture@myworkday.com
- Subject Filter: (leave blank)
```

---

### Step 2: Condition

```
Condition:
- Body contains "enrolled in"
  OR
- Body contains "You are now enrolled"
```

---

### Step 3: Initialize Variables

**In the "If yes" branch:**

#### Variable 1: EmployeeName
```
Name: EmployeeName
Type: String
Value: [Dynamic content] Subject
```

#### Variable 2: TrainingName
```
Name: TrainingName
Type: String
Value: [Expression]
if(
  contains(triggerOutputs()?['body/body'], 'enrolled in'),
  substring(
    triggerOutputs()?['body/body'],
    add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
    100
  ),
  'Training notification'
)
```

#### Variable 3: DueDate
```
Name: DueDate
Type: String
Value: [Expression]
formatDateTime(addDays(utcNow(), 30), 'MM/dd/yyyy')
```
*Note: Sets default 30 days since your emails don't have due dates*

#### Variable 4: ActionLink
```
Name: ActionLink
Type: String
Value: [Expression]
if(
  contains(triggerOutputs()?['body/body'], 'https://'),
  substring(
    triggerOutputs()?['body/body'],
    indexOf(triggerOutputs()?['body/body'], 'https://'),
    200
  ),
  'https://workday.accenture.com'
)
```

---

### Step 4: Send Reminder

**Teams Message:**
```
Post message in a chat or channel

Recipient: [Dynamic] To (from email trigger)

Message:
📚 New Training Enrollment

Hi [EmployeeName],

You've been enrolled in:
[TrainingName]

Suggested completion: [DueDate]

View details: [ActionLink]

This is an automated reminder from your Workday notification.
```

**Outlook Email:**
```
Send an email (V2)

To: [Dynamic] To
Subject: Training Reminder: [TrainingName]

Body:
Hi [EmployeeName],

You have been enrolled in a new training:

Training: [TrainingName]
Suggested Completion: [DueDate]

Click here to view details and start the training:
[ActionLink]

This is an automated reminder based on your Workday enrollment notification.

Best regards,
Learning & Development Team
```

---

## 📋 Updated Flow Structure

```
┌─────────────────────────────────────────┐
│  Trigger: Email from                     │
│  accenture@myworkday.com                 │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Condition: Body contains                │
│  "enrolled in"                           │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Extract:                                │
│  - Employee Name (from subject)          │
│  - Training Name (after "enrolled in")   │
│  - Due Date (default 30 days)            │
│  - Action Link (https:// URL)            │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│  Send Reminder via Teams/Email           │
└─────────────────────────────────────────┘
```

---

## 🎯 Example Output

**Input Email:**
```
You are now enrolled in Amazon Q Developer (Recommended for Custom Engineers).
Click here to view the notification details.
https://in.accenture.com/mylearninghelp/contacts
```

**Extracted Variables:**
```
EmployeeName: "VANDANA YADAV"
TrainingName: "Amazon Q Developer (Recommended for Custom Engineers)"
DueDate: "05/08/2026" (30 days from today)
ActionLink: "https://in.accenture.com/mylearninghelp/contacts"
```

**Reminder Message:**
```
📚 New Training Enrollment

Hi VANDANA YADAV,

You've been enrolled in:
Amazon Q Developer (Recommended for Custom Engineers)

Suggested completion: 05/08/2026

View details: https://in.accenture.com/mylearninghelp/contacts
```

---

## 🔧 Troubleshooting Your Format

### If Training Name Extraction Fails

**Your email might have variations. Try these alternatives:**

**Option 1: Extract everything between "enrolled in" and first period:**
```javascript
substring(
  triggerOutputs()?['body/body'],
  add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
  sub(
    indexOf(triggerOutputs()?['body/body'], '.', add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12)),
    add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12)
  )
)
```

**Option 2: Extract first line of email body:**
```javascript
first(split(triggerOutputs()?['body/body'], '\n'))
```

---

### If Link Extraction Fails

**Your email might have multiple links. To get the FIRST link:**
```javascript
substring(
  triggerOutputs()?['body/body'],
  indexOf(triggerOutputs()?['body/body'], 'https://'),
  sub(
    coalesce(
      indexOf(triggerOutputs()?['body/body'], ' ', indexOf(triggerOutputs()?['body/body'], 'https://')),
      indexOf(triggerOutputs()?['body/body'], '\n', indexOf(triggerOutputs()?['body/body'], 'https://'))
    ),
    indexOf(triggerOutputs()?['body/body'], 'https://')
  )
)
```

---

## ✅ Testing with Your Email

1. **Forward yourself the Workday email** you showed in the screenshot
2. **Run the flow**
3. **Check the variables** in flow run history
4. **Verify output:**
   - Training Name should be: "Amazon Q Developer (Recommended for Custom Engineers)"
   - Link should be: "https://in.accenture.com/mylearninghelp/contacts"
   - Employee Name should be: "VANDANA YADAV"

---

## 💡 Key Differences for Accenture Workday

| Element | Standard Workday | Your Accenture Format |
|---------|-----------------|----------------------|
| Sender | workday@company.com | accenture@myworkday.com |
| Subject | Training name | Employee name |
| Training Name | "Course: [name]" | "enrolled in [name]" |
| Due Date | "Due Date: MM/DD/YYYY" | ❌ Not present |
| Link Label | Various | "Click here to view" |
| Format | Structured | Paragraph style |

---

## 🚀 Quick Start for Your Format

1. Use the expressions above (not the generic ones)
2. Set sender to: `accenture@myworkday.com`
3. Condition: Body contains "enrolled in"
4. Default due date to 30 days
5. Test with your actual email

---

**This guide is specifically for your Accenture Workday email format!**
