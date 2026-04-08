# Power Automate Expressions Reference
## Common Expressions for Workday Training Tracker

---

## Date & Time Calculations

### Calculate Days Remaining
```
div(sub(ticks(variables('DueDate')), ticks(utcNow())), 864000000000)
```
**Explanation**: Converts date difference to days (864000000000 ticks = 1 day)

### Format Date for Display
```
formatDateTime(variables('DueDate'), 'MMM dd, yyyy')
```
**Output**: Apr 15, 2026

### Add Days to Current Date
```
addDays(utcNow(), 30)
```
**Use Case**: Default due date if not found in email

### Get Current Date/Time
```
utcNow()
```

### Convert to ISO Format
```
formatDateTime(variables('DueDate'), 'yyyy-MM-dd')
```

---

## String Manipulation

### Extract Substring After Keyword
```
substring(
  variables('EmailBody'),
  add(indexOf(variables('EmailBody'), 'Course:'), 8),
  sub(
    indexOf(variables('EmailBody'), '\n', add(indexOf(variables('EmailBody'), 'Course:'), 8)),
    add(indexOf(variables('EmailBody'), 'Course:'), 8)
  )
)
```
**Use Case**: Extract training name after "Course:"

### Check if String Contains Text
```
contains(variables('EmailBody'), 'Training')
```
**Returns**: true/false

### Replace Text
```
replace(variables('EmailBody'), '<br>', '\n')
```
**Use Case**: Clean HTML from email body

### Trim Whitespace
```
trim(variables('TrainingName'))
```

### Convert to Lowercase
```
toLower(variables('Status'))
```

### Convert to Uppercase
```
toUpper(variables('Status'))
```

---

## Conditional Logic

### If-Then-Else
```
if(
  lessOrEquals(variables('DaysRemaining'), 0),
  'Overdue',
  if(
    lessOrEquals(variables('DaysRemaining'), 3),
    'Due Soon',
    'Pending'
  )
)
```

### Multiple OR Conditions
```
or(
  or(
    contains(triggerOutputs()?['subject'], 'Training'),
    contains(triggerOutputs()?['subject'], 'Certification')
  ),
  contains(triggerOutputs()?['subject'], 'Learning')
)
```

### Multiple AND Conditions
```
and(
  equals(variables('Status'), 'Pending'),
  lessOrEquals(variables('DaysRemaining'), 3)
)
```

---

## Array Operations

### Get Array Length
```
length(body('Get_items')?['value'])
```

### Filter Array
```
filter(
  body('Get_items')?['value'],
  item => equals(item?['Status'], 'Completed')
)
```
**Use Case**: Count completed trainings

### Get First Item
```
first(body('Get_items')?['value'])
```

### Get Last Item
```
last(body('Get_items')?['value'])
```

### Check if Array is Empty
```
empty(body('Get_items')?['value'])
```

---

## Email Trigger Outputs

### Get Email Subject
```
triggerOutputs()?['subject']
```

### Get Email Body (HTML)
```
triggerOutputs()?['body/body']
```

### Get Email From Address
```
triggerOutputs()?['from']
```

### Get Email To Address
```
triggerOutputs()?['to']
```

### Get Email Received Time
```
triggerOutputs()?['receivedDateTime']
```

### Get Email Importance
```
triggerOutputs()?['importance']
```

---

## HTML Parsing

### Strip HTML Tags
```
replace(
  replace(
    variables('EmailBody'),
    '<[^>]*>',
    ''
  ),
  '&nbsp;',
  ' '
)
```

### Extract URL from href
```
substring(
  variables('EmailBody'),
  add(indexOf(variables('EmailBody'), 'href="'), 6),
  sub(
    indexOf(variables('EmailBody'), '"', add(indexOf(variables('EmailBody'), 'href="'), 6)),
    add(indexOf(variables('EmailBody'), 'href="'), 6)
  )
)
```

---

## Calculations

### Calculate Percentage
```
if(
  greater(variables('TotalCount'), 0),
  mul(div(variables('CompletedCount'), variables('TotalCount')), 100),
  0
)
```

### Round Number
```
round(variables('CompletionRate'), 2)
```

### Absolute Value
```
abs(variables('DaysRemaining'))
```

---

## Dynamic Content

### Access SharePoint List Item
```
items('Apply_to_each')?['TrainingName']
items('Apply_to_each')?['DueDate']
items('Apply_to_each')?['ID']
```

### Access Excel Row
```
items('Apply_to_each')?['TrainingName']
```

### Previous Action Output
```
body('Get_items')?['value']
outputs('Create_item')?['ID']
```

---

## Error Handling

### Check if Variable is Null
```
if(
  empty(variables('TrainingName')),
  'Default Value',
  variables('TrainingName')
)
```

### Coalesce (First Non-Null)
```
coalesce(
  variables('ExtractedDate'),
  addDays(utcNow(), 30)
)
```

### Try-Catch Pattern
```
Use "Configure run after" on actions:
- Run after: has failed
- Then: Send error notification
```

---

## Useful Patterns

### Extract Date with Multiple Formats
```
if(
  contains(variables('EmailBody'), 'Due Date:'),
  substring(variables('EmailBody'), add(indexOf(variables('EmailBody'), 'Due Date:'), 10), 10),
  if(
    contains(variables('EmailBody'), 'Complete by:'),
    substring(variables('EmailBody'), add(indexOf(variables('EmailBody'), 'Complete by:'), 13), 10),
    formatDateTime(addDays(utcNow(), 30), 'yyyy-MM-dd')
  )
)
```

### Build HTML Email Body
```
concat(
  '<html><body>',
  '<h1>Training Reminder</h1>',
  '<p>Training: ', variables('TrainingName'), '</p>',
  '<p>Due Date: ', formatDateTime(variables('DueDate'), 'MMM dd, yyyy'), '</p>',
  '<a href="', variables('ActionLink'), '">Complete Training</a>',
  '</body></html>'
)
```

### Create Unique ID
```
concat(
  formatDateTime(utcNow(), 'yyyyMMddHHmmss'),
  '-',
  guid()
)
```

### Determine Priority Color
```
if(
  equals(variables('Status'), 'Overdue'),
  'Attention',
  if(
    equals(variables('Status'), 'Due Soon'),
    'Warning',
    'Good'
  )
)
```

---

## SharePoint Specific

### Filter Query Syntax
```
Status eq 'Pending' or Status eq 'Due Soon'
UserEmail eq 'john.doe@company.com'
DueDate le datetime'2026-04-15T00:00:00Z'
```

### Update Item Fields
```
{
  "Status": "@{variables('NewStatus')}",
  "LastReminderSent": "@{utcNow()}",
  "DaysRemaining": @{variables('DaysRemaining')}
}
```

---

## Teams Adaptive Card Variables

### Dynamic Card Content
```json
{
  "type": "TextBlock",
  "text": "${TrainingName}",
  "weight": "Bolder"
}
```

### Conditional Styling
```
"color": "${if(equals(Status, 'Overdue'), 'Attention', 'Default')}"
```

---

## Common Mistakes to Avoid

### ❌ Wrong: Using single quotes in expressions
```
'@{variables('TrainingName')}'
```

### ✅ Correct: No quotes around expression
```
@{variables('TrainingName')}
```

### ❌ Wrong: Accessing array without index
```
body('Get_items')['TrainingName']
```

### ✅ Correct: Access first item or use Apply to each
```
first(body('Get_items')?['value'])?['TrainingName']
```

### ❌ Wrong: Division without null check
```
div(variables('CompletedCount'), variables('TotalCount'))
```

### ✅ Correct: Check for division by zero
```
if(greater(variables('TotalCount'), 0), div(variables('CompletedCount'), variables('TotalCount')), 0)
```

---

## Testing Expressions

### Use Compose Action
```
1. Add "Compose" action
2. Enter expression in Inputs
3. Run flow
4. Check Outputs in run history
```

### Use Initialize Variable
```
1. Add "Initialize variable" action
2. Set value to expression
3. Use in subsequent actions
4. Check value in run history
```

---

## Performance Tips

1. **Cache values**: Store frequently used expressions in variables
2. **Minimize indexOf calls**: Extract once, reuse variable
3. **Use filter at source**: SharePoint filter query vs. filter in flow
4. **Avoid nested loops**: Flatten data structure when possible
5. **Batch operations**: Update multiple items in single action when possible

---

## Debugging Tips

1. **Add Compose actions**: Output intermediate values
2. **Use descriptive variable names**: Makes debugging easier
3. **Check run history**: View inputs/outputs of each action
4. **Test with sample data**: Use known values to verify logic
5. **Enable flow checker**: Catches syntax errors before running
