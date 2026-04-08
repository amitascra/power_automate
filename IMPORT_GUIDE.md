# Import Guide: Company Power Automate Environment
## How to Import Workday Training Tracker into Your Organization

---

## 🎯 Overview

Since you're using your company's Power Automate environment, you'll need to **manually create the flows** using the blueprints provided. Power Automate doesn't support direct import from GitHub for individual flows (only solution packages).

---

## 📋 Prerequisites Checklist

### Access Requirements
- [ ] Access to make.powerautomate.com with your company account
- [ ] Power Automate license (included in Microsoft 365)
- [ ] Permissions to create flows in your environment
- [ ] Access to Outlook mailbox (your own or shared)
- [ ] Access to Microsoft Teams
- [ ] SharePoint site access OR OneDrive for Business

### Information to Gather
- [ ] **Workday email sender address** (e.g., workday@yourcompany.com)
- [ ] **SharePoint site URL** (if using SharePoint for storage)
- [ ] **Your time zone** for scheduled flows
- [ ] **Sample Workday training email** for testing

---

## 🚀 Step-by-Step Import Process

### Step 1: Set Up Data Storage (10 minutes)

#### Option A: SharePoint List (Recommended for Teams)

1. **Navigate to your SharePoint site**
   - Go to your team's SharePoint site
   - Or create a new site: SharePoint → Create site → Team site

2. **Create the Training Tracker list**
   ```
   Click "New" → "List" → "Blank list"
   Name: "Training Tracker"
   Description: "Workday training and certification tracking"
   ```

3. **Add columns** (click "+ Add column" for each):
   
   | Column Name | Type | Settings |
   |------------|------|----------|
   | UserEmail | Single line of text | Required |
   | TrainingName | Single line of text | Required |
   | DueDate | Date and time | Required, Include time |
   | Status | Choice | Choices: Pending, Due Soon, Overdue, Completed |
   | ActionLink | Hyperlink | Optional |
   | LastReminderSent | Date and time | Optional, Include time |
   | CreatedDate | Date and time | Optional, Include time |
   | CompletedDate | Date and time | Optional, Include time |

4. **Save the SharePoint site URL** - you'll need it for the flows

#### Option B: Excel Online (Simpler Alternative)

1. **Create Excel file in OneDrive or SharePoint**
   - Open OneDrive for Business or SharePoint document library
   - Click "New" → "Excel workbook"
   - Name: "Training_Tracker.xlsx"

2. **Set up the table**
   - Add headers in Row 1: 
     `UserEmail | TrainingName | DueDate | Status | ActionLink | LastReminderSent | CreatedDate | CompletedDate`
   - Select the header row → Insert → Table → Check "My table has headers"
   - Name the table: "TrainingData" (Table Design → Table Name)

3. **Save and note the file location**

---

### Step 2: Create Flow 1 - Email Processor (15 minutes)

1. **Go to Power Automate**
   - Navigate to https://make.powerautomate.com
   - Sign in with your company account
   - Ensure you're in the correct environment (top right)

2. **Create new automated flow**
   ```
   Click "Create" → "Automated cloud flow"
   Flow name: "Workday Training - Email Processor"
   Trigger: Search "When a new email arrives"
   Select: "When a new email arrives (V3)" - Office 365 Outlook
   Click "Create"
   ```

3. **Configure the trigger**
   - Click on the trigger step
   - **Folder**: Inbox
   - **From**: `workday@yourcompany.com` (replace with your actual sender)
   - **Include Attachments**: No
   - Leave other fields blank

4. **Add Condition to filter emails**
   ```
   Click "+ New step" → Search "Condition"
   
   Configure condition:
   - Click "Choose a value"
   - Select "Subject" from dynamic content
   - Select "contains"
   - Value: Training
   
   Click "Add" → "Add row" (OR condition)
   - Subject contains "Certification"
   
   Click "Add" → "Add row" (OR condition)
   - Subject contains "Learning"
   ```

5. **In the "If yes" branch, initialize variables**
   
   Click "Add an action" → Search "Initialize variable"
   
   **Variable 1: EmailBody**
   - Name: `EmailBody`
   - Type: String
   - Value: Click "Add dynamic content" → "Body" (from email trigger)

   **Variable 2: TrainingName**
   - Name: `TrainingName`
   - Type: String
   - Value: Switch to Expression tab, paste:
   ```
   if(contains(variables('EmailBody'), 'Course:'), substring(variables('EmailBody'), add(indexOf(variables('EmailBody'), 'Course:'), 8), 100), triggerOutputs()?['subject'])
   ```

   **Variable 3: DueDate**
   - Name: `DueDate`
   - Type: String
   - Value: Expression:
   ```
   if(contains(variables('EmailBody'), 'Due Date:'), substring(variables('EmailBody'), add(indexOf(variables('EmailBody'), 'Due Date:'), 10), 10), formatDateTime(addDays(utcNow(), 30), 'yyyy-MM-dd'))
   ```

   **Variable 4: ActionLink**
   - Name: `ActionLink`
   - Type: String
   - Value: Expression:
   ```
   if(contains(variables('EmailBody'), 'https://'), substring(variables('EmailBody'), indexOf(variables('EmailBody'), 'https://'), 100), 'https://workday.yourcompany.com')
   ```

   **Variable 5: RecipientEmail**
   - Name: `RecipientEmail`
   - Type: String
   - Value: Dynamic content → "To" (from email trigger)

6. **Add storage action**

   **For SharePoint:**
   ```
   Add action → Search "Create item" → Select SharePoint
   
   Configure:
   - Site Address: [Select your SharePoint site]
   - List Name: Training Tracker
   - UserEmail: [Dynamic] RecipientEmail variable
   - TrainingName: [Dynamic] TrainingName variable
   - DueDate: [Dynamic] DueDate variable
   - Status: Pending (type manually)
   - ActionLink: [Dynamic] ActionLink variable
   - CreatedDate: [Expression] utcNow()
   ```

   **For Excel:**
   ```
   Add action → Search "Add a row into a table" → Select Excel Online
   
   Configure:
   - Location: OneDrive for Business (or SharePoint)
   - Document Library: OneDrive (or Documents)
   - File: Browse to Training_Tracker.xlsx
   - Table: TrainingData
   - Fill in columns with variables (same as above)
   ```

7. **Add error handling**
   ```
   On the storage action (Create item or Add row):
   - Click the "..." menu → "Configure run after"
   - Check "has failed"
   - Click "Done"
   
   Add action below → "Send an email (V2)"
   - To: your-email@company.com
   - Subject: Training Tracker - Parsing Error
   - Body: Include error details using dynamic content
   ```

8. **Save the flow**
   - Click "Save" (top right)
   - Click "Test" → "Manually" → "Test"
   - Send yourself a test email matching the format

---

### Step 3: Create Flow 2 - Daily Reminder (20 minutes)

1. **Create scheduled flow**
   ```
   Create → "Scheduled cloud flow"
   Flow name: "Workday Training - Daily Reminders"
   Starting: [Tomorrow's date]
   Repeat every: 1 Day
   At these hours: 9 (9 AM)
   At these minutes: 0
   Time zone: [Select your time zone]
   Click "Create"
   ```

2. **Get pending trainings**

   **For SharePoint:**
   ```
   Add action → "Get items" (SharePoint)
   - Site Address: [Your SharePoint site]
   - List Name: Training Tracker
   - Filter Query: Status eq 'Pending' or Status eq 'Due Soon'
   - Top Count: 1000
   ```

   **For Excel:**
   ```
   Add action → "List rows present in a table" (Excel)
   - Location: OneDrive/SharePoint
   - Document Library: OneDrive/Documents
   - File: Training_Tracker.xlsx
   - Table: TrainingData
   ```

3. **Process each training**
   ```
   Add action → "Apply to each"
   - Select an output: [Dynamic] value (from Get items/List rows)
   ```

4. **Inside the loop, calculate days remaining**
   ```
   Add action → "Initialize variable" (first time only)
   OR "Set variable" (subsequent iterations)
   
   - Name: DaysRemaining
   - Type: Integer
   - Value: [Expression]
   div(sub(ticks(items('Apply_to_each')?['DueDate']), ticks(utcNow())), 864000000000)
   ```

5. **Determine status with conditions**
   ```
   Add action → "Condition"
   - Condition: DaysRemaining is less than or equal to 0
   
   If yes:
     - Set variable: Status = "Overdue"
     - Set variable: Priority = "Urgent"
   
   If no:
     - Add nested condition: DaysRemaining <= 1
       If yes: Status = "Due Soon", Priority = "High"
       If no: 
         - Add nested condition: DaysRemaining <= 3
           If yes: Status = "Due Soon", Priority = "Medium"
   ```

6. **Send Teams notification**
   ```
   Add action → "Post adaptive card in a chat or channel"
   - Post as: Flow bot
   - Post in: Chat with Flow bot
   - Recipient: [Dynamic] UserEmail from current item
   - Message: Copy from templates/teams-notification.json
     (Replace ${variables} with dynamic content)
   ```

7. **Send Outlook email**
   ```
   Add action → "Send an email (V2)"
   - To: [Dynamic] UserEmail
   - Subject: [Dynamic] Combine Status + "Training Reminder: " + TrainingName
   - Body: Copy HTML from templates/outlook-reminder.html
     (Replace ${variables} with dynamic content)
   - Importance: [Expression] if(equals(variables('Priority'), 'Urgent'), 'High', 'Normal')
   ```

8. **Update last reminder sent**
   ```
   Add action → "Update item" (SharePoint) or "Update a row" (Excel)
   - ID/Key: [Dynamic] ID from current item
   - LastReminderSent: [Expression] utcNow()
   - Status: [Dynamic] Status variable
   ```

9. **Save and test**

---

### Step 4: Create Flow 3 - Weekly Summary (Optional, 15 minutes)

1. **Create scheduled flow**
   ```
   Create → "Scheduled cloud flow"
   Flow name: "Workday Training - Weekly Summary"
   Starting: [Next Monday]
   Repeat every: 1 Week
   On these days: Monday
   At these hours: 9
   Time zone: [Your time zone]
   ```

2. **Get all trainings and calculate statistics**
   - Follow similar pattern to Daily Reminder
   - Use filter expressions to count by status
   - Send summary email with statistics

3. **Reference**: See `flows/main-flow-blueprint.md` for detailed steps

---

### Step 5: Create Flow 4 - Completion Detector (Optional, 10 minutes)

1. **Create automated flow**
   ```
   Trigger: "When a new email arrives (V3)"
   - From: workday@yourcompany.com
   - Subject Filter: Completed
   ```

2. **Parse and update status**
   - Extract training name
   - Find matching record
   - Update Status to "Completed"

---

## 🔧 Customization for Your Company

### Update Email Sender
In all flows, replace `workday@company.com` with your actual Workday sender address.

### Adjust Reminder Timing
- Edit Daily Reminder flow → Recurrence trigger
- Change time to match your preference (e.g., 8 AM, 10 AM)

### Customize Notification Content
- Edit the email/Teams message text
- Add your company logo/branding
- Adjust tone and language

### Modify Escalation Days
- In Daily Reminder flow, change conditions:
  - `DaysRemaining <= 3` → Change to your preference
  - `DaysRemaining <= 1` → Change to your preference

---

## ✅ Testing Checklist

- [ ] Send test email matching Workday format
- [ ] Verify email processor flow triggers
- [ ] Check data appears in SharePoint/Excel
- [ ] Manually run Daily Reminder flow
- [ ] Verify Teams notification received
- [ ] Verify Outlook email received
- [ ] Check action links work
- [ ] Test with multiple users

---

## 🆘 Common Issues

### Flow Won't Trigger
- Check flow is turned ON (toggle in flow details)
- Verify email sender address matches exactly
- Check mailbox permissions

### Variables Not Parsing
- Review email format - may differ from sample
- Check `parsers/email-parser-logic.md` for alternative patterns
- Use "Compose" actions to debug expressions

### SharePoint Connection Issues
- Reconnect SharePoint connector
- Verify site permissions
- Check list name spelling

### Teams Notifications Not Appearing
- Verify Teams connector permissions
- Check recipient email format
- Try "Post in channel" instead of chat

---

## 📞 Getting Help

### Internal Resources
- Contact your Power Platform admin
- Check company's Power Automate governance policies
- Review company's SharePoint permissions

### External Resources
- Power Automate Community: https://powerusers.microsoft.com
- Microsoft Documentation: https://docs.microsoft.com/power-automate
- This repository: https://github.com/amitascra/power_automate

---

## 🔒 Security Considerations

### Before Deploying
- [ ] Review with IT/Security team
- [ ] Ensure compliance with data policies
- [ ] Verify email data handling is approved
- [ ] Check if service account is required
- [ ] Confirm SharePoint permissions are appropriate

### Best Practices
- Use service account for production flows
- Limit access to SharePoint list
- Don't hardcode sensitive information
- Enable flow analytics and monitoring
- Document all customizations

---

## 📊 Next Steps After Import

1. **Pilot Test** - Run with 5-10 users for 1 week
2. **Gather Feedback** - Survey users on timing, content, usefulness
3. **Adjust** - Fine-tune based on feedback
4. **Scale** - Roll out to full organization
5. **Monitor** - Check flow run history weekly
6. **Maintain** - Update parsing patterns as Workday emails change

---

## 💡 Pro Tips

1. **Start Simple**: Import Email Processor and Daily Reminder first
2. **Test Thoroughly**: Use your own email for initial testing
3. **Document Changes**: Keep notes on any customizations you make
4. **Version Control**: Export flows periodically as backup
5. **Collaborate**: Share with colleagues who might benefit

---

**Ready to start?** Begin with Step 1 and work through each section. The entire setup should take 60-90 minutes.
