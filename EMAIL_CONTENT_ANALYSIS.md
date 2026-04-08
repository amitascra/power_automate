# Email Content Analysis - Accenture Workday Training Notification

## 📧 Complete Email Structure

### Header Information
```
From: Accenture Careers <accenture@myworkday.com>
To: YADAV, VANDANA
Subject: VANDANA YADAV
Date: Wed 4/8/2026 11:57
```

### Email Body (Full Text)

```
You are now enrolled in Amazon Q Developer (Recommended for Custom Engineers).

For questions about Workday, copy and paste this link into your browser to 
contact our Learning Hub and click on the "Support Helpdesk" link.

Click here to view the notification details.

This is a valid Accenture email notification. If you have any questions, 
please visit workday.accenture.com directly to see this notification/inbox item.

> Making every day better,
  together on Workday
```

### Footer
```
This email was intended for vandana.a.yadav@accenture.com  Manage Preferences
```

---

## 🔍 Key Elements to Extract

### 1. Training/Course Name
**Location:** First line of email body  
**Pattern:** "You are now enrolled in [TRAINING NAME]"  
**Value:** `Amazon Q Developer (Recommended for Custom Engineers)`

**Extraction Strategy:**
- Text starts after: "enrolled in "
- Text ends at: "." (period) or "\n" (newline)
- Length: Variable (can be short or long with parentheses)

---

### 2. Employee Name
**Location:** Email subject line  
**Pattern:** Direct value in subject  
**Value:** `VANDANA YADAV`

**Extraction Strategy:**
- Simply use: `triggerOutputs()?['subject']`
- No parsing needed

---

### 3. Action Links

#### Link 1: Learning Hub Support
**Text:** "copy and paste this link into your browser to contact our Learning Hub"  
**URL:** `https://in.accenture.com/mylearninghelp/contacts`  
**Purpose:** Support/Help desk

#### Link 2: Notification Details
**Text:** "Click here to view the notification details"  
**URL:** Embedded in "Click here" hyperlink  
**Purpose:** View full training details in Workday

#### Link 3: Direct Workday Access
**Text:** "visit workday.accenture.com directly"  
**URL:** `workday.accenture.com`  
**Purpose:** Direct portal access

**Extraction Strategy:**
- Look for first `https://` in email body
- Primary link is the Learning Hub URL
- May need to extract from HTML `href` attribute

---

### 4. Due Date
**Status:** ❌ **NOT PRESENT** in this email format  
**Workaround:** Set default (e.g., 30 days from enrollment)

---

### 5. Email Recipient
**Location:** "To" field  
**Value:** `YADAV, VANDANA`  
**Alternative:** Extract from footer: `vandana.a.yadav@accenture.com`

---

## 📋 Email Characteristics

### Format Type
- **Style:** Notification/Enrollment confirmation
- **Structure:** Paragraph format (not structured key-value pairs)
- **HTML:** Yes (contains links, formatting)
- **Signature:** Workday branding footer

### Unique Identifiers
- ✅ Contains: "You are now enrolled in"
- ✅ Contains: "enrolled in"
- ✅ Contains: "Accenture email notification"
- ✅ Sender: "accenture@myworkday.com"

### What's Missing (vs Standard Workday)
- ❌ No "Course:" label
- ❌ No "Due Date:" field
- ❌ No "Training ID:" or reference number
- ❌ No estimated completion time
- ❌ No training category/type
- ❌ No manager information

---

## 🎯 Parsing Strategy

### Step 1: Identify Email Type
```javascript
Condition:
- From contains "accenture@myworkday.com"
  AND
- Body contains "You are now enrolled in"
  OR
- Body contains "enrolled in"
```

### Step 2: Extract Training Name

**Method A: Simple substring (Recommended)**
```javascript
substring(
  triggerOutputs()?['body/body'],
  add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
  100
)
```
- Starts 12 characters after "enrolled in" (skips "enrolled in ")
- Grabs 100 characters (enough for full name)
- Result: "Amazon Q Developer (Recommended for Custom Engineers)."

**Method B: Extract until period**
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
- More precise - stops at first period
- Result: "Amazon Q Developer (Recommended for Custom Engineers)"

**Method C: Clean up with trim**
```javascript
trim(
  replace(
    substring(
      triggerOutputs()?['body/body'],
      add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
      100
    ),
    '.',
    ''
  )
)
```
- Removes trailing period and whitespace
- Result: "Amazon Q Developer (Recommended for Custom Engineers)"

### Step 3: Extract Employee Name
```javascript
triggerOutputs()?['subject']
```
- Direct extraction from subject
- Result: "VANDANA YADAV"

### Step 4: Extract Primary Link

**From HTML body:**
```javascript
substring(
  triggerOutputs()?['body/body'],
  indexOf(triggerOutputs()?['body/body'], 'https://'),
  200
)
```
- Finds first https:// URL
- Result: "https://in.accenture.com/mylearninghelp/contacts"

**Alternative - Extract from href attribute:**
```javascript
substring(
  triggerOutputs()?['body/body'],
  add(indexOf(triggerOutputs()?['body/body'], 'href="'), 6),
  sub(
    indexOf(triggerOutputs()?['body/body'], '"', add(indexOf(triggerOutputs()?['body/body'], 'href="'), 6)),
    add(indexOf(triggerOutputs()?['body/body'], 'href="'), 6)
  )
)
```

### Step 5: Set Default Due Date
```javascript
formatDateTime(addDays(utcNow(), 30), 'MM/dd/yyyy')
```
- Adds 30 days to current date
- Result: "05/08/2026" (if today is 04/08/2026)

---

## 🧪 Test Cases

### Test Case 1: Standard Enrollment
**Input:**
```
You are now enrolled in Amazon Q Developer (Recommended for Custom Engineers).
```
**Expected Output:**
- Training: "Amazon Q Developer (Recommended for Custom Engineers)"

### Test Case 2: Short Course Name
**Input:**
```
You are now enrolled in Security Awareness Training.
```
**Expected Output:**
- Training: "Security Awareness Training"

### Test Case 3: Long Course Name with Special Characters
**Input:**
```
You are now enrolled in Advanced Python Programming: Data Science & Machine Learning (Level 3).
```
**Expected Output:**
- Training: "Advanced Python Programming: Data Science & Machine Learning (Level 3)"

### Test Case 4: Multiple Links
**Input:**
```
...contact our Learning Hub: https://in.accenture.com/mylearninghelp/contacts
Click here to view: https://workday.accenture.com/notification/12345
```
**Expected Output:**
- Link: "https://in.accenture.com/mylearninghelp/contacts" (first link)

---

## 💡 Recommended Flow Configuration

### Trigger Settings
```
When a new email arrives (V3)
- Folder: Inbox
- From: accenture@myworkday.com
- Subject Filter: (leave blank)
- Include Attachments: No
```

### Condition
```
Body contains "enrolled in"
```

### Variables to Initialize
1. **EmployeeName** (String): `triggerOutputs()?['subject']`
2. **TrainingName** (String): Use Method C above (with trim)
3. **DueDate** (String): `formatDateTime(addDays(utcNow(), 30), 'MM/dd/yyyy')`
4. **ActionLink** (String): First https:// URL
5. **EnrollmentDate** (String): `formatDateTime(utcNow(), 'MM/dd/yyyy')`

### Notification Template
```
📚 New Training Enrollment

Hi [EmployeeName],

You've been enrolled in:
[TrainingName]

Enrollment Date: [EnrollmentDate]
Suggested Completion: [DueDate] (30 days)

View details and start training:
[ActionLink]

Questions? Visit the Learning Hub or contact support.

This is an automated reminder from your Workday notification.
```

---

## 🔧 Edge Cases to Handle

### Case 1: Email Body is HTML
- Use HTML parsing or strip tags
- Extract text content only
- Handle `<br>` tags as newlines

### Case 2: Multiple Enrollments in One Email
- Current logic handles first enrollment only
- May need to split by "enrolled in" and process each

### Case 3: Email Forwarded
- "From" field might be different
- Check original sender in email headers

### Case 4: Link Not Present
- Fallback to: `https://workday.accenture.com`
- Or: Skip sending notification

---

## ✅ Final Parsing Expressions (Ready to Use)

### Training Name (Clean)
```javascript
trim(
  first(
    split(
      substring(
        triggerOutputs()?['body/body'],
        add(indexOf(triggerOutputs()?['body/body'], 'enrolled in'), 12),
        200
      ),
      '.'
    )
  )
)
```

### Employee Name
```javascript
triggerOutputs()?['subject']
```

### Due Date (30 days default)
```javascript
formatDateTime(addDays(utcNow(), 30), 'MM/dd/yyyy')
```

### Action Link
```javascript
if(
  contains(triggerOutputs()?['body/body'], 'https://'),
  first(
    split(
      substring(
        triggerOutputs()?['body/body'],
        indexOf(triggerOutputs()?['body/body'], 'https://'),
        200
      ),
      ' '
    )
  ),
  'https://workday.accenture.com'
)
```

### Enrollment Date
```javascript
formatDateTime(utcNow(), 'MM/dd/yyyy')
```

---

## 📊 Summary

| Element | Present | Extraction Method | Difficulty |
|---------|---------|-------------------|------------|
| Training Name | ✅ Yes | Parse after "enrolled in" | Easy |
| Employee Name | ✅ Yes | Subject line | Very Easy |
| Due Date | ❌ No | Default 30 days | N/A |
| Action Link | ✅ Yes | First https:// URL | Easy |
| Enrollment Date | ✅ Implicit | Current date | Very Easy |
| Training ID | ❌ No | N/A | N/A |
| Manager | ❌ No | N/A | N/A |

**Overall Complexity:** Low - Simple text parsing with fallbacks

---

This analysis is based on your actual Accenture Workday email format.
