# Conditional Generator

## Purpose
Choose a generator or value based on input conditions.

## Use It When
- Value depends on request input.
- You need branching logic for different scenarios.

## Required Fields
- `type`: "conditional"
- `conditions`

## Optional Fields
- `else`

## Condition Structure
Each condition has:
- `when`: { `key`, `equals` }
- `then`: { `use` }

## Example
```json
{
  "type": "conditional",
  "conditions": [
    {
      "when": {
        "key": "$.input.query:modules",
        "equals": "tickets"
      },
      "then": {
        "use": "$generators:#/generators/ticket_id"
      }
    },
    {
      "when": {
        "key": "$.input.query:modules",
        "equals": "contacts"
      },
      "then": {
        "use": "$generators:#/generators/contact_id"
      }
    }
  ],
  "else": "$generators:#/generators/activity_id"
}
```

## Checklist
- All referenced generators exist.
- Conditions are mutually exclusive.
- Provide an `else` when possible.
