# Technical Assumptions & Constraints
## Workday Learning & Certification Tracker

---

## Data Source Assumptions

### Email-Based Data Collection
1. **No Direct Workday API Access**
   - Assumption: Organization does not provide API access to Workday
   - Constraint: All data must be extracted from email notifications
   - Impact: Parsing accuracy depends on email format consistency

2. **Email Format Consistency**
   - Assumption: Workday emails follow a consistent template structure
   - Constraint: Changes to email format may break parsing logic
   - Mitigation: Implement multiple parsing patterns with fallbacks

3. **Email Sender Identification**
   - Assumption: Workday emails come from identifiable sender addresses
   - Examples: `workday@company.com`, `noreply@workday.com`
   - Constraint: Must maintain sender whitelist

4. **Email Delivery Reliability**
   - Assumption: Outlook receives all Workday notifications
   - Constraint: Email filtering/spam rules may affect delivery
   - Mitigation: Monitor for missing emails, implement health checks

---

## Authentication & Permissions

### Service Account Requirements
1. **Outlook Mailbox Access**
   - Requirement: Dedicated service account or user mailbox
   - Permissions: Read emails, send emails
   - Consideration: Shared mailbox vs. user mailbox

2. **Microsoft Teams Access**
   - Requirement: Ability to post messages to Teams channels/chats
   - Permissions: Send messages as bot or user
   - Consideration: Bot registration may be required

3. **SharePoint/Excel Access**
   - Requirement: Read/write access to data storage location
   - Permissions: Contribute or higher on SharePoint site
   - Consideration: OneDrive vs. SharePoint storage

4. **Enterprise Authentication**
   - Assumption: OAuth 2.0 or Azure AD authentication
   - Constraint: Multi-factor authentication may require app passwords
   - Security: Service principal recommended for production

---

## Data Storage Assumptions

### Storage Platform Selection
1. **SharePoint List (Recommended)**
   - Pros: Native integration, versioning, permissions, scalability
   - Cons: Requires SharePoint license
   - Capacity: Supports 30M items per list
   - Performance: Indexed columns for queries

2. **Excel Online (Alternative)**
   - Pros: Simple, familiar, no additional licensing
   - Cons: 5MB file size limit, concurrent access issues
   - Capacity: ~1M rows (practical limit ~100K)
   - Performance: Slower for large datasets

3. **Dataverse (Enterprise)**
   - Pros: Scalable, relational, enterprise-grade
   - Cons: Requires Power Apps/Dynamics license
   - Capacity: Based on license tier
   - Performance: Optimized for large-scale operations

### Data Retention
- Assumption: Training records retained for 2+ years for compliance
- Constraint: Storage platform capacity limits
- Mitigation: Implement archival strategy for completed trainings

---

## Flow Execution Assumptions

### Trigger Frequency
1. **Email Trigger**
   - Assumption: Near real-time processing (1-5 minute delay)
   - Constraint: Outlook connector polling interval
   - Impact: Slight delay between email arrival and processing

2. **Scheduled Reminders**
   - Assumption: Daily execution acceptable (not hourly)
   - Constraint: Recurrence trigger minimum interval
   - Recommendation: 9:00 AM local time

3. **Weekly Summary**
   - Assumption: Monday morning delivery preferred
   - Constraint: Single time zone for all users
   - Consideration: Multi-timezone support may require multiple flows

### Concurrency & Throttling
1. **Power Automate Limits**
   - Free/Office 365: 750 runs/day, 15-minute timeout
   - Premium: 100K runs/day, 30-minute timeout
   - Constraint: May need premium for large organizations

2. **Connector Throttling**
   - Outlook: 300 calls/60 seconds
   - Teams: 100 calls/60 seconds
   - SharePoint: 600 calls/60 seconds
   - Mitigation: Implement retry logic, batch operations

3. **Concurrent Processing**
   - Assumption: Multiple emails may arrive simultaneously
   - Constraint: Flow concurrency settings (default: 25)
   - Recommendation: Set concurrency to 1 for data consistency

---

## Email Parsing Assumptions

### Content Structure
1. **HTML vs. Plain Text**
   - Assumption: Emails are HTML-formatted
   - Constraint: Parsing HTML requires text extraction
   - Fallback: Support plain text emails

2. **Required Fields**
   - Training/Certification Name: **Required**
   - Due Date: **Required** (fallback: +30 days)
   - Action Link: **Optional** (fallback: generic portal URL)

3. **Date Format Variations**
   - Supported: MM/DD/YYYY, DD-MMM-YYYY, YYYY-MM-DD
   - Constraint: Locale-specific formats may vary
   - Mitigation: Multiple date parsing patterns

4. **Link Extraction**
   - Assumption: Links are embedded as `<a href="...">` tags
   - Constraint: Plain text URLs may be harder to extract
   - Mitigation: Regex patterns for both formats

---

## Notification Assumptions

### Delivery Channels
1. **Microsoft Teams**
   - Assumption: All users have Teams access
   - Constraint: Users must be in Teams tenant
   - Consideration: Personal chat vs. channel posting

2. **Outlook Email**
   - Assumption: All users have active Outlook mailboxes
   - Constraint: Email overload concerns
   - Mitigation: Consolidate notifications, allow opt-out

### Notification Timing
1. **Daily Reminders**
   - Assumption: 9:00 AM is acceptable time
   - Constraint: Single time zone
   - Consideration: User preference settings

2. **Escalation Alerts**
   - T-3 days: Standard priority
   - T-1 day: High priority
   - Overdue: Urgent priority
   - Assumption: Escalation timing is sufficient

### User Interaction
1. **Action Links**
   - Assumption: Links open in user's default browser
   - Constraint: SSO/authentication may be required
   - Consideration: Deep linking to specific training

2. **Completion Detection**
   - Assumption: Workday sends completion confirmation emails
   - Constraint: Must parse completion emails separately
   - Alternative: Manual status update via form

---

## Calendar Integration Assumptions

### Outlook Calendar Access
1. **Availability Reading**
   - Assumption: Read-only access to user calendars
   - Constraint: Privacy concerns, permissions required
   - Use Case: Avoid sending reminders during meetings/PTO

2. **Calendar Event Creation (Future)**
   - Assumption: Users want training deadlines in calendar
   - Constraint: Requires write permissions
   - Consideration: Optional feature

---

## Compliance & Security Assumptions

### Data Privacy
1. **Personal Data Handling**
   - Data Stored: Email, name, training records
   - Assumption: Complies with GDPR/privacy regulations
   - Requirement: Data retention policy, user consent

2. **Access Control**
   - Assumption: Role-based access to training data
   - Constraint: SharePoint/Excel permissions model
   - Requirement: Managers see team data, users see own data

3. **Audit Trail**
   - Assumption: Flow run history provides audit trail
   - Constraint: 28-day retention in Power Automate
   - Mitigation: Export logs to external system if needed

### Read-Only Workday Access
1. **No Write-Back to Workday**
   - Assumption: Flow cannot update Workday directly
   - Constraint: Completion status must be updated in Workday by user
   - Impact: Potential for data sync issues

2. **Email as Source of Truth**
   - Assumption: Workday emails are authoritative
   - Constraint: No validation against Workday database
   - Risk: Email parsing errors may cause inaccuracies

---

## Performance & Scalability Assumptions

### User Volume
1. **Small Organization (< 100 users)**
   - Storage: Excel Online sufficient
   - Flow Tier: Office 365 plan adequate
   - Notifications: Direct email/Teams messages

2. **Medium Organization (100-1000 users)**
   - Storage: SharePoint List recommended
   - Flow Tier: Premium may be needed
   - Notifications: Batch processing required

3. **Large Organization (> 1000 users)**
   - Storage: Dataverse recommended
   - Flow Tier: Premium required
   - Notifications: Advanced batching, multiple flows

### Training Volume
1. **Assumption**: Average 5-10 trainings per user per year
2. **Constraint**: Storage grows linearly with user count
3. **Mitigation**: Archive completed trainings older than 2 years

---

## Error Handling Assumptions

### Email Parsing Failures
1. **Assumption**: 5-10% of emails may fail to parse correctly
2. **Mitigation**: Log errors, send to admin for manual review
3. **Fallback**: Use default values where possible

### Notification Failures
1. **Assumption**: Occasional delivery failures acceptable
2. **Mitigation**: Retry logic (2-3 attempts)
3. **Fallback**: Log failure, retry in next scheduled run

### Data Consistency
1. **Assumption**: Eventual consistency acceptable
2. **Constraint**: No real-time synchronization with Workday
3. **Mitigation**: Daily reconciliation checks

---

## Future Enhancements (Out of Scope)

### Phase 2 Considerations
1. **Manager Dashboard**
   - Power BI report showing team completion rates
   - Requires: Power BI license, data export

2. **Workday API Integration**
   - Direct API access if available in future
   - Eliminates email parsing dependency

3. **Mobile App Notifications**
   - Push notifications via Power Apps
   - Requires: Power Apps license

4. **Gamification**
   - Completion badges, leaderboards
   - Requires: Custom UI, additional storage

5. **Multi-Language Support**
   - Localized notifications
   - Requires: Translation resources

---

## Deployment Assumptions

### Environment
1. **Production Environment**
   - Assumption: Dedicated Power Automate environment
   - Constraint: Environment admin access required
   - Recommendation: Separate dev/test/prod environments

2. **Service Account**
   - Assumption: Dedicated service account for flows
   - Constraint: License requirements
   - Recommendation: Premium connector license if needed

### Testing
1. **Test Data**
   - Assumption: Sample Workday emails available
   - Requirement: Test with real email formats
   - Consideration: Anonymize production data for testing

2. **User Acceptance Testing**
   - Assumption: 2-week pilot with small user group
   - Requirement: Feedback mechanism
   - Success Criteria: 90%+ parsing accuracy, positive user feedback

---

## Known Limitations

1. **Email Format Changes**
   - Risk: Workday email template updates break parsing
   - Mitigation: Monitor for parsing failures, update patterns

2. **Time Zone Handling**
   - Limitation: Single time zone for all users
   - Impact: Reminders may arrive at suboptimal times
   - Future: Multi-timezone support

3. **Completion Detection**
   - Limitation: Relies on completion confirmation emails
   - Impact: Manual completions may not be tracked
   - Alternative: Periodic sync with Workday reports

4. **No Offline Support**
   - Limitation: Requires internet connectivity
   - Impact: No notifications if services down
   - Mitigation: Service health monitoring

5. **Language Support**
   - Limitation: English-only parsing patterns
   - Impact: Non-English emails may fail to parse
   - Future: Multi-language pattern support
