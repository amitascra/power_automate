# Deployment Guide
## Workday Learning & Certification Tracker

---

## Prerequisites

### Required Licenses
- [ ] Microsoft 365 Business Standard or higher (includes Power Automate)
- [ ] Exchange Online (Outlook)
- [ ] Microsoft Teams
- [ ] SharePoint Online (if using SharePoint storage)
- [ ] Power Automate Premium (optional, for high-volume scenarios)

### Required Permissions
- [ ] Power Automate environment access (Maker role)
- [ ] Outlook mailbox access (service account or user)
- [ ] SharePoint site contributor access (if using SharePoint)
- [ ] Teams posting permissions
- [ ] Azure AD app registration (if using service principal)

### Required Information
- [ ] Workday email sender address(es)
- [ ] Sample Workday training notification emails
- [ ] Target user group/distribution list
- [ ] Preferred notification time (e.g., 9:00 AM)
- [ ] Time zone for notifications

---

## Phase 1: Environment Setup

### Step 1: Create Power Automate Environment (Optional)
```
1. Go to Power Platform Admin Center (admin.powerplatform.microsoft.com)
2. Click "Environments" → "New"
3. Name: "Workday Learning Tracker - Production"
4. Type: Production
5. Region: Select appropriate region
6. Create database: Yes (if using Dataverse)
7. Click "Create"
```

### Step 2: Create Service Account (Recommended)
```
1. Azure AD → Users → New User
2. Username: workday-tracker@company.com
3. Assign licenses:
   - Microsoft 365 (for Outlook)
   - Power Automate Premium (if needed)
4. Configure mailbox
5. Disable MFA or configure app password
```

### Step 3: Create Data Storage

#### Option A: SharePoint List (Recommended)
```
1. Navigate to SharePoint site
2. Click "New" → "List"
3. Name: "Training Tracker"
4. Create columns:
   - TrainingID (Single line text, auto-generated)
   - UserEmail (Single line text)
   - TrainingName (Single line text)
   - DueDate (Date)
   - Status (Choice: Pending, Due Soon, Overdue, Completed)
   - ActionLink (Hyperlink)
   - LastReminderSent (Date and Time)
   - CreatedDate (Date and Time)
   - CompletedDate (Date)
   - DaysRemaining (Number, calculated)
5. Set permissions (Contribute for service account)
```

#### Option B: Excel Online
```
1. Create Excel file in OneDrive/SharePoint
2. Name: "Training_Tracker.xlsx"
3. Create table with columns:
   | TrainingID | UserEmail | TrainingName | DueDate | Status | ActionLink | LastReminderSent | CreatedDate | CompletedDate |
4. Format as Table (Insert → Table)
5. Name table: "TrainingData"
```

---

## Phase 2: Flow Creation

### Flow 1: Email Processing Flow

#### Step 1: Create New Flow
```
1. Power Automate (make.powerautomate.com)
2. Click "Create" → "Automated cloud flow"
3. Name: "Workday Training - Email Processor"
4. Trigger: "When a new email arrives (V3)" (Office 365 Outlook)
5. Click "Create"
```

#### Step 2: Configure Trigger
```
Trigger: When a new email arrives (V3)
- Folder: Inbox
- From: workday@company.com (adjust to your sender)
- Subject Filter: Leave blank (we'll filter in next step)
- Include Attachments: No
- Only with Attachments: No
- Importance: Any
```

#### Step 3: Add Filter Condition
```
Action: Condition
- Subject contains "Training" OR
- Subject contains "Certification" OR
- Subject contains "Learning"
```

#### Step 4: Add Parse Email Actions
```
In "If yes" branch:

1. Initialize variable: EmailBody
   - Type: String
   - Value: @{triggerOutputs()?['body/body']}

2. Initialize variable: TrainingName
   - Type: String
   - Value: [Use expression from email-parser-logic.md]

3. Initialize variable: DueDate
   - Type: String
   - Value: [Use expression from email-parser-logic.md]

4. Initialize variable: ActionLink
   - Type: String
   - Value: [Use expression from email-parser-logic.md]

5. Initialize variable: RecipientEmail
   - Type: String
   - Value: @{triggerOutputs()?['body/to']}
```

#### Step 5: Add Data Storage Action
```
For SharePoint:
Action: Create item
- Site Address: [Your SharePoint site]
- List Name: Training Tracker
- Fields:
  - UserEmail: @{variables('RecipientEmail')}
  - TrainingName: @{variables('TrainingName')}
  - DueDate: @{variables('DueDate')}
  - Status: Pending
  - ActionLink: @{variables('ActionLink')}
  - CreatedDate: @{utcNow()}

For Excel:
Action: Add a row into a table
- Location: OneDrive/SharePoint
- Document Library: [Your library]
- File: Training_Tracker.xlsx
- Table: TrainingData
- Fields: [Same as above]
```

#### Step 6: Add Error Handling
```
Add "Configure run after" on storage action:
- Run after: has failed
- Action: Send email notification to admin
  - To: admin@company.com
  - Subject: "Training Tracker - Parsing Error"
  - Body: Include email details and error message
```

#### Step 7: Save and Test
```
1. Click "Save"
2. Send test email matching trigger criteria
3. Check flow run history
4. Verify data stored correctly
```

---

### Flow 2: Daily Reminder Flow

#### Step 1: Create Scheduled Flow
```
1. Power Automate → Create → Scheduled cloud flow
2. Name: "Workday Training - Daily Reminders"
3. Recurrence: Daily at 9:00 AM
4. Time zone: [Your time zone]
5. Click "Create"
```

#### Step 2: Get Pending Trainings
```
For SharePoint:
Action: Get items
- Site Address: [Your SharePoint site]
- List Name: Training Tracker
- Filter Query: Status eq 'Pending' or Status eq 'Due Soon'
- Top Count: 1000

For Excel:
Action: List rows present in a table
- Location: OneDrive/SharePoint
- Document Library: [Your library]
- File: Training_Tracker.xlsx
- Table: TrainingData
- Filter Query: Status eq 'Pending' or Status eq 'Due Soon'
```

#### Step 3: Process Each Training
```
Action: Apply to each
- Select output from previous step: value

Inside loop:
1. Calculate days remaining:
   Variable: DaysRemaining
   Value: @{div(sub(ticks(items('Apply_to_each')?['DueDate']), ticks(utcNow())), 864000000000)}

2. Determine status and priority:
   Condition: DaysRemaining <= 0
   - If yes: Status = "Overdue", Priority = "Urgent"
   - If no: 
     - Condition: DaysRemaining <= 1
       - If yes: Status = "Due Soon", Priority = "High"
     - Condition: DaysRemaining <= 3
       - If yes: Status = "Due Soon", Priority = "Medium"
```

#### Step 4: Send Notifications
```
Action: Post adaptive card in a chat or channel (Teams)
- Post as: Flow bot
- Post in: Chat with Flow bot
- Recipient: @{items('Apply_to_each')?['UserEmail']}
- Message: [Use teams-notification.json template]
- Update message: No

Action: Send an email (V2) (Outlook)
- To: @{items('Apply_to_each')?['UserEmail']}
- Subject: [@{variables('Status')}] Training Reminder: @{items('Apply_to_each')?['TrainingName']}
- Body: [Use outlook-reminder.html template]
- Importance: @{if(equals(variables('Priority'), 'Urgent'), 'High', 'Normal')}
```

#### Step 5: Update Last Reminder Sent
```
For SharePoint:
Action: Update item
- Site Address: [Your SharePoint site]
- List Name: Training Tracker
- Id: @{items('Apply_to_each')?['ID']}
- LastReminderSent: @{utcNow()}
- Status: @{variables('Status')}

For Excel:
Action: Update a row
- [Similar configuration]
```

#### Step 6: Save and Test
```
1. Click "Save"
2. Click "Test" → "Manually"
3. Verify notifications sent
4. Check data updated
```

---

### Flow 3: Weekly Summary Flow

#### Step 1: Create Scheduled Flow
```
1. Power Automate → Create → Scheduled cloud flow
2. Name: "Workday Training - Weekly Summary"
3. Recurrence: Weekly on Monday at 9:00 AM
4. Click "Create"
```

#### Step 2: Get All User Trainings
```
Action: Get items (SharePoint) or List rows (Excel)
- Filter by user email (if per-user summary)
- Or get all items (if organization-wide summary)
```

#### Step 3: Calculate Statistics
```
1. Initialize variable: TotalCount
   Value: @{length(body('Get_items')?['value'])}

2. Initialize variable: CompletedCount
   Value: @{length(filter(body('Get_items')?['value'], item => equals(item?['Status'], 'Completed')))}

3. Initialize variable: PendingCount
   Value: @{length(filter(body('Get_items')?['value'], item => equals(item?['Status'], 'Pending')))}

4. Initialize variable: OverdueCount
   Value: @{length(filter(body('Get_items')?['value'], item => equals(item?['Status'], 'Overdue')))}

5. Initialize variable: CompletionRate
   Value: @{if(greater(variables('TotalCount'), 0), mul(div(variables('CompletedCount'), variables('TotalCount')), 100), 0)}
```

#### Step 4: Send Summary
```
Action: Send an email (V2)
- To: [User or distribution list]
- Subject: "Weekly Learning Summary - @{formatDateTime(utcNow(), 'MMM dd, yyyy')}"
- Body: HTML template with statistics and pending items list
```

---

### Flow 4: Completion Detection Flow

#### Step 1: Create Email Trigger Flow
```
1. Power Automate → Create → Automated cloud flow
2. Name: "Workday Training - Completion Detector"
3. Trigger: "When a new email arrives (V3)"
4. Configure trigger:
   - From: workday@company.com
   - Subject Filter: "Completed" OR "Congratulations" OR "Certificate"
```

#### Step 2: Parse Completion Email
```
Extract training name from email body
(Similar to email processing flow)
```

#### Step 3: Update Training Status
```
For SharePoint:
Action: Get items
- Filter Query: TrainingName eq '@{variables('TrainingName')}' and Status ne 'Completed'

Action: Update item
- Status: Completed
- CompletedDate: @{utcNow()}

For Excel:
Action: List rows
- Filter: TrainingName eq '@{variables('TrainingName')}'

Action: Update row
- Status: Completed
- CompletedDate: @{utcNow()}
```

---

## Phase 3: Testing

### Unit Testing
```
1. Test email parsing with sample emails
2. Test date calculations
3. Test notification templates
4. Test error handling
```

### Integration Testing
```
1. Send test Workday email
2. Verify parsing and storage
3. Wait for scheduled reminder (or trigger manually)
4. Verify notifications received
5. Send completion email
6. Verify status updated
```

### User Acceptance Testing
```
1. Select 5-10 pilot users
2. Enable flows for pilot group
3. Monitor for 1 week
4. Collect feedback
5. Adjust as needed
```

---

## Phase 4: Production Deployment

### Pre-Deployment Checklist
- [ ] All flows tested successfully
- [ ] Service account configured
- [ ] Data storage created and accessible
- [ ] Notification templates finalized
- [ ] Error handling implemented
- [ ] Admin notifications configured
- [ ] Documentation completed
- [ ] User communication prepared

### Deployment Steps
```
1. Export flows from dev environment
2. Import to production environment
3. Update connections to production resources
4. Configure trigger conditions for production
5. Enable flows
6. Monitor for 24 hours
7. Communicate to users
```

### Post-Deployment Monitoring
```
Week 1:
- Check flow run history daily
- Monitor error rates
- Verify notification delivery
- Collect user feedback

Week 2-4:
- Check flow run history weekly
- Analyze parsing accuracy
- Review completion rates
- Adjust reminder timing if needed
```

---

## Troubleshooting

### Common Issues

#### Flow Not Triggering
```
1. Check trigger conditions
2. Verify email sender address
3. Check mailbox permissions
4. Review flow run history for errors
```

#### Parsing Failures
```
1. Review email format
2. Check regex patterns
3. Verify variable initialization
4. Add logging to identify failure point
```

#### Notification Not Delivered
```
1. Check Teams permissions
2. Verify email addresses
3. Review connector throttling limits
4. Check spam/junk folders
```

#### Data Not Updating
```
1. Verify SharePoint/Excel permissions
2. Check for concurrent access issues
3. Review update action configuration
4. Check for data type mismatches
```

---

## Maintenance

### Regular Tasks
```
Weekly:
- Review flow run history
- Check error logs
- Monitor parsing accuracy

Monthly:
- Review user feedback
- Update parsing patterns if needed
- Archive old completed trainings
- Review performance metrics

Quarterly:
- Review and update documentation
- Assess need for enhancements
- Review license usage
- Conduct user survey
```

### Updates and Enhancements
```
1. Test changes in dev environment
2. Document changes
3. Communicate to users
4. Deploy to production
5. Monitor for issues
```

---

## Rollback Plan

### If Critical Issues Occur
```
1. Disable affected flows
2. Notify users of temporary outage
3. Investigate root cause
4. Fix issue in dev environment
5. Test thoroughly
6. Redeploy to production
7. Re-enable flows
8. Notify users of resolution
```

---

## Support

### User Support
```
- Create FAQ document
- Provide help desk contact
- Set up feedback channel (Teams/Email)
- Schedule office hours for questions
```

### Admin Support
```
- Document admin procedures
- Create runbook for common issues
- Set up monitoring alerts
- Establish escalation path
```
