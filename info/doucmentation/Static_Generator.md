# Static Generator

## Purpose
Return a constant, fixed value.

## Use It When
- Value never changes.
- You need known values for testing.

## Required Fields
- `type`: "static"
- `value`

## Optional Fields
- `name`

## Example
```json
{
  "type": "static",
  "value": "OPEN"
}
```

## Supported Value Types
- string
- number
- boolean
- array
- object
- null

## Checklist
- Value type matches the API schema.
- Do not use for timestamps or system enums.
