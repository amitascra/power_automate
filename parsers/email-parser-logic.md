# Email Parser Logic
## Workday Training & Certification Email Extraction

## Overview
This document defines the parsing patterns and logic for extracting training/certification data from Workday-generated Outlook emails.

---

## Email Structure Assumptions

### Typical Workday Email Format
```
From: workday@company.com
Subject: Action Required: Complete [Training Name] by [Date]

Body:
Dear [Employee Name],

You have been assigned the following training/certification:

Course/Training: [Training Name]
Due Date: [MM/DD/YYYY] or [DD-MMM-YYYY]
Description: [Brief description]

To complete this training, please click the link below:
[Action Link URL]

Thank you,
Workday Learning Team
```

---

## Parsing Patterns

### 1. Training/Certification Name Extraction

#### Pattern Options
```regex
Option 1: Course:\s*(.+?)(?:\n|$)
Option 2: Training:\s*(.+?)(?:\n|$)
Option 3: Certification:\s*(.+?)(?:\n|$)
Option 4: Subject:\s*(.+?)(?:\n|$)
```

#### Power Automate Expression
```
substring(
  body('Get_email_body'),
  add(indexOf(body('Get_email_body'), 'Course:'), 8),
  sub(
    indexOf(body('Get_email_body'), '\n', add(indexOf(body('Get_email_body'), 'Course:'), 8)),
    add(indexOf(body('Get_email_body'), 'Course:'), 8)
  )
)
```

#### Fallback Strategy
If "Course:" not found, try:
1. "Training:"
2. "Certification:"
3. Extract from email subject line

---

### 2. Due Date Extraction

#### Date Format Patterns
```regex
Common Formats:
- MM/DD/YYYY: \d{2}/\d{2}/\d{4}
- DD-MMM-YYYY: \d{2}-[A-Za-z]{3}-\d{4}
- YYYY-MM-DD: \d{4}-\d{2}-\d{2}
- Month DD, YYYY: [A-Za-z]+\s+\d{1,2},\s+\d{4}

Keywords to search:
- "Due Date:"
- "Complete by:"
- "Deadline:"
- "Expiry Date:"
- "Must complete before:"
```

#### Power Automate Expression
```
substring(
  body('Get_email_body'),
  add(indexOf(body('Get_email_body'), 'Due Date:'), 10),
  10
)
```

#### Date Conversion
```
Convert to ISO format: formatDateTime(variables('ExtractedDate'), 'yyyy-MM-dd')
```

---

### 3. Action Link Extraction

#### URL Pattern
```regex
https?://[^\s<>"]+workday[^\s<>"]*
https?://[^\s<>"]+mytime[^\s<>"]*
https?://[^\s<>"]+learning[^\s<>"]*
```

#### Power Automate Expression (HTML Email)
```
substring(
  body('Get_email_body'),
  add(indexOf(body('Get_email_body'), 'href="'), 6),
  sub(
    indexOf(body('Get_email_body'), '"', add(indexOf(body('Get_email_body'), 'href="'), 6)),
    add(indexOf(body('Get_email_body'), 'href="'), 6)
  )
)
```

#### Multiple Links Handling
If multiple links found:
1. Prioritize links containing "workday"
2. Then links containing "learning" or "training"
3. Exclude unsubscribe/footer links

---

## Power Automate Implementation

### Step 1: Get Email Body
```
Action: Get email (V3)
Output: body('Get_email')
```

### Step 2: Convert HTML to Text (Optional)
```
Action: Compose
Expression: replace(replace(body('Get_email'), '<[^>]*>', ''), '&nbsp;', ' ')
```

### Step 3: Initialize Variables
```
Variable: TrainingName (String)
Variable: DueDate (String)
Variable: ActionLink (String)
Variable: RecipientEmail (String)
```

### Step 4: Extract Training Name
```
Action: Set variable (TrainingName)
Value: 
  if(
    contains(body('Get_email_body'), 'Course:'),
    substring(body('Get_email_body'), add(indexOf(body('Get_email_body'), 'Course:'), 8), 100),
    if(
      contains(body('Get_email_body'), 'Training:'),
      substring(body('Get_email_body'), add(indexOf(body('Get_email_body'), 'Training:'), 10), 100),
      triggerOutputs()?['subject']
    )
  )
```

### Step 5: Extract Due Date
```
Action: Set variable (DueDate)
Value:
  substring(
    body('Get_email_body'),
    add(indexOf(body('Get_email_body'), 'Due Date:'), 10),
    10
  )
```

### Step 6: Extract Action Link
```
Action: Set variable (ActionLink)
Value:
  substring(
    body('Get_email_body'),
    indexOf(body('Get_email_body'), 'https://'),
    sub(
      indexOf(body('Get_email_body'), '"', indexOf(body('Get_email_body'), 'https://')),
      indexOf(body('Get_email_body'), 'https://')
    )
  )
```

---

## Error Handling

### Missing Fields
```
If Training Name not found:
  - Use email subject as fallback
  - Log warning

If Due Date not found:
  - Set default: 30 days from today
  - Flag for manual review
  - Log warning

If Action Link not found:
  - Use generic Workday portal URL
  - Flag for manual review
  - Log warning
```

### Invalid Date Formats
```
Try multiple date parsing approaches:
1. formatDateTime() with different format strings
2. Manual regex extraction and reconstruction
3. Default to 30 days if all fail
```

---

## Testing Scenarios

### Test Case 1: Standard Format
```
Input Email:
Course: Information Security Awareness
Due Date: 04/15/2026
Link: https://workday.company.com/training/12345

Expected Output:
{
  "TrainingName": "Information Security Awareness",
  "DueDate": "2026-04-15",
  "ActionLink": "https://workday.company.com/training/12345"
}
```

### Test Case 2: Alternative Format
```
Input Email:
Training: Annual Compliance Review
Complete by: 15-APR-2026
Access here: https://learning.company.com/course/678

Expected Output:
{
  "TrainingName": "Annual Compliance Review",
  "DueDate": "2026-04-15",
  "ActionLink": "https://learning.company.com/course/678"
}
```

### Test Case 3: Missing Due Date
```
Input Email:
Certification: Project Management Professional
Link: https://workday.company.com/cert/999

Expected Output:
{
  "TrainingName": "Project Management Professional",
  "DueDate": "2026-05-08", (30 days from today)
  "ActionLink": "https://workday.company.com/cert/999",
  "Warning": "Due date not found, using default"
}
```

---

## Optimization Tips

### Performance
- Use `indexOf()` instead of regex for simple patterns (faster in Power Automate)
- Cache email body in variable to avoid repeated calls
- Use `contains()` for quick checks before expensive operations

### Reliability
- Always provide fallback values
- Log all parsing warnings/errors
- Validate extracted data before storage
- Test with real Workday emails from production

### Maintainability
- Document all regex patterns
- Keep parsing logic in separate actions for easier debugging
- Use descriptive variable names
- Comment complex expressions
