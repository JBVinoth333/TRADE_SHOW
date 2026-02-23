# Reference Generator

## Purpose
Pass data directly from the incoming request.

## Use It When
- Input value should flow into response or another operation.
- You need query/path/body/header values without changes.

## Required Fields
- `type`: "reference"
- `ref`

## Optional Fields
- `name`

## Reference Format
`$.input.<location>:$.<jsonPath>`

Supported locations:
- `body`
- `query`
- `path`
- `header`

## Example (Path Param)
```json
{
  "type": "reference",
  "ref": "$.input.path:$.ticketId"
}
```

## Example (Body Field)
```json
{
  "type": "reference",
  "ref": "$.input.body:$.contactId"
}
```

## Checklist
- The field exists in the request.
- The location type is correct.
