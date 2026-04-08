# Power Automate Flow Blueprint
## Workday Learning & Certification Tracker

### Flow Type
**Automated Cloud Flow**

---

## Flow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        TRIGGER PHASE                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    When a new email arrives (V3) - Outlook Mail Connector
    - Folder: Inbox (or specific folder)
    - Include Attachments: No
    - From: workday@company.com (or configured sender)
    - Subject Filter: Contains "Training" OR "Certification" OR "Learning"
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        FILTER PHASE                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    Condition: Check if email is from Workday
    - From contains "workday" OR specific sender domain
    - Subject contains keywords: "training", "certification", "learning"
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        PARSE PHASE                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    Parse Email Content (HTML to Text)
    ├─ Extract Training/Certification Name
    │  - Regex: Look for course titles, certification names
    │  - Pattern: "Course: (.*)" or "Training: (.*)"
    │
    ├─ Extract Due Date/Expiry Date
    │  - Regex: Date patterns (MM/DD/YYYY, DD-MMM-YYYY)
    │  - Pattern: "Due Date: (.*)" or "Complete by: (.*)"
    │
    └─ Extract Action Links
       - Regex: Extract URLs from email body
       - Pattern: href="(https://.*workday.*)" or MyTime & Expense links
                              ↓
    Compose JSON Object
    {
      "TrainingName": "...",
      "DueDate": "...",
      "ActionLink": "...",
      "RecipientEmail": "...",
      "Status": "Pending",
      "ExtractedDate": "..."
    }
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DATA STORAGE PHASE                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    Store in SharePoint List or Excel Online (OneDrive)
    Columns:
    - TrainingID (Auto-generated)
    - UserEmail
    - TrainingName
    - DueDate
    - Status (Pending/Completed/Overdue)
    - ActionLink
    - LastReminderSent
    - CreatedDate
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                  SCHEDULED REMINDER FLOW                         │
│                    (Separate Recurrence Flow)                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    Recurrence Trigger: Daily at 9:00 AM
                              ↓
    Get Items from SharePoint/Excel
    - Filter: Status = "Pending" OR Status = "Due Soon"
                              ↓
    For Each Training Item:
      ├─ Calculate Days Until Due
      │  - DaysDiff = DueDate - Today
      │
      ├─ Determine Reminder Type
      │  - If DaysDiff <= 0: Status = "Overdue" → Send URGENT reminder
      │  - If DaysDiff <= 1: Status = "Due Soon" → Send T-1 escalation
      │  - If DaysDiff <= 3: Status = "Due Soon" → Send T-3 escalation
      │  - If DaysDiff > 3 AND < 7: Send daily reminder
      │
      └─ Send Notification
         ├─ Microsoft Teams: Post adaptive card
         │  - Title: "Training Reminder: [TrainingName]"
         │  - Due Date: [DueDate]
         │  - Status: [Status]
         │  - Action Button: [ActionLink]
         │
         └─ Outlook Email: Send formatted email
            - Subject: "[Status] Training Reminder: [TrainingName]"
            - Body: HTML template with action link
            - Priority: High (if Overdue or T-1)
                              ↓
    Update LastReminderSent field in storage
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    WEEKLY SUMMARY FLOW                           │
│                    (Separate Recurrence Flow)                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    Recurrence Trigger: Weekly on Monday at 9:00 AM
                              ↓
    Get All Items from SharePoint/Excel for User
                              ↓
    Calculate Summary Statistics:
    - Total Trainings: Count(All)
    - Completed: Count(Status = "Completed")
    - Pending: Count(Status = "Pending")
    - Overdue: Count(Status = "Overdue")
    - Completion Rate: (Completed / Total) * 100
                              ↓
    Send Weekly Summary via Teams & Outlook
    - Subject: "Weekly Learning Summary"
    - Body: Summary table with statistics
    - Include list of pending/overdue items
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                  COMPLETION DETECTION FLOW                       │
│                    (Email-based trigger)                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
    When a new email arrives from Workday
    - Subject contains: "Completed" OR "Congratulations" OR "Certificate"
                              ↓
    Parse Email for Training Name
                              ↓
    Update SharePoint/Excel:
    - Set Status = "Completed"
    - Set CompletionDate = Today
                              ↓
    Suppress Future Reminders for this item
```

---

## Key Components

### 1. Trigger
- **Connector**: Office 365 Outlook
- **Action**: When a new email arrives (V3)
- **Configuration**:
  - Folder: Inbox
  - From: workday@company.com
  - Subject Filter: Training|Certification|Learning

### 2. Parse Email Content
- **Action**: Compose (to build JSON)
- **Expressions**:
  ```
  Training Name: 
  substring(body('Get_email'), indexOf(body('Get_email'), 'Course:'), 100)
  
  Due Date:
  substring(body('Get_email'), indexOf(body('Get_email'), 'Due Date:'), 20)
  
  Action Link:
  substring(body('Get_email'), indexOf(body('Get_email'), 'https://'), 100)
  ```

### 3. Data Storage Options
**Option A: SharePoint List**
- Pros: Native Power Automate integration, versioning, permissions
- Cons: Requires SharePoint site

**Option B: Excel Online (OneDrive/SharePoint)**
- Pros: Simple, familiar interface
- Cons: Concurrent access limitations

**Option C: Dataverse**
- Pros: Enterprise-grade, scalable, relationships
- Cons: Licensing requirements

### 4. Reminder Logic (Conditions)
```
If DaysDiff <= 0:
  Status = "Overdue"
  Priority = "High"
  Send urgent reminder

Else If DaysDiff = 1:
  Status = "Due Soon"
  Send T-1 escalation alert

Else If DaysDiff = 3:
  Status = "Due Soon"
  Send T-3 escalation alert

Else If DaysDiff > 3 AND Status = "Pending":
  Send daily reminder
```

### 5. Notification Templates
**Teams Adaptive Card**:
```json
{
  "type": "AdaptiveCard",
  "body": [
    {
      "type": "TextBlock",
      "text": "Training Reminder",
      "weight": "Bolder",
      "size": "Large"
    },
    {
      "type": "FactSet",
      "facts": [
        {"title": "Training:", "value": "[TrainingName]"},
        {"title": "Due Date:", "value": "[DueDate]"},
        {"title": "Status:", "value": "[Status]"}
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.OpenUrl",
      "title": "Complete Training",
      "url": "[ActionLink]"
    }
  ]
}
```

**Outlook Email Template**: See `templates/outlook-reminder.html`

---

## Error Handling

### Email Parsing Failures
- **Scope**: Try-Catch around parse operations
- **Action**: Log to separate error list, send notification to admin

### Storage Failures
- **Retry Policy**: 3 attempts with exponential backoff
- **Fallback**: Send email to admin with parsed data

### Notification Failures
- **Retry Policy**: 2 attempts
- **Fallback**: Log failure, retry in next scheduled run

---

## Performance Considerations

### Throttling Limits
- **Outlook Connector**: 300 calls/60 seconds
- **Teams Connector**: 100 calls/60 seconds
- **SharePoint**: 600 calls/60 seconds

### Optimization
- Batch notifications where possible
- Use "Apply to each" with concurrency control (degree: 1-50)
- Filter data at source (SharePoint views, Excel filters)

---

## Testing Strategy

### Unit Testing
1. Test email parsing with sample Workday emails
2. Test date calculation logic
3. Test status transitions

### Integration Testing
1. End-to-end flow with test email
2. Verify storage updates
3. Verify notifications sent

### User Acceptance Testing
1. Test with real Workday emails
2. Verify reminder timing
3. Verify action links work correctly

---

## Deployment Checklist

- [ ] Create SharePoint list or Excel table
- [ ] Configure Outlook connector with service account
- [ ] Configure Teams connector
- [ ] Set up email filters/rules
- [ ] Test with sample emails
- [ ] Enable flows
- [ ] Monitor for 1 week
- [ ] Gather user feedback
- [ ] Adjust reminder timing if needed
