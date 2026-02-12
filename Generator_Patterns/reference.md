# REFERENCE Generator - Use Request Input Data

## What It Does

Passes data directly from incoming request to response without modification. Extracts values from request body, query parameters, path parameters, or headers.

---

## When to Use REFERENCE

Use REFERENCE when:
- Using data from request body
- Getting values from query parameters
- Using path parameters
- Using header values
- Data comes directly from request

**Common Scenarios:**
- Pass ticket ID from request to response
- Use query parameters in operations
- Extract URL path parameters
- Forward authorization headers
- Use nested request objects

---

## Required Properties

| Property | Type | Purpose |
|----------|------|---------|
| `type` | string | Must be `"reference"` |
| `ref` | string | Path to request data |

## Optional Properties

| Property | Type | Purpose |
|----------|------|---------|
| `name` | string | Generator identifier |

---

## Reference Path Format

**Pattern:** `$.input.{location}:$.{jsonPath}`

### Location Types

| Location | Source | Example |
|----------|--------|---------|
| `$.input.body` | Request body JSON | `$.input.body:$.ticketId` |
| `$.input.query` | Query parameters | `$.input.query:$.departmentId` |
| `$.input.path` | URL path parameters | `$.input.path:$.ticketId` |
| `$.input.header` | HTTP headers | `$.input.header:$.Authorization` |

### Path Selector Examples

| Reference | Extracts |
|-----------|----------|
| `$.input.body:$.fieldName` | Simple field from body |
| `$.input.body:$.parent.child.fieldName` | Nested field |
| `$.input.body:$.items[*].id` | All array IDs |
| `$.input.body:$.items[0].id` | First array ID |
| `$.input.query:$.parameterName` | Query parameter |
| `$.input.path:$.pathParamName` | Path parameter |
| `$.input.header:$.HeaderName` | Header value |

---

## Examples

### From Request Body
Simple field:
```json
{
  "type": "reference",
  "ref": "$.input.body:$.ticketId"
}
```

### From Query Parameters
Query value:
```json
{
  "type": "reference",
  "ref": "$.input.query:$.departmentId"
}
```

### From URL Path
Path parameter:
```json
{
  "type": "reference",
  "ref": "$.input.path:$.ticketId"
}
```

### From HTTP Header
Header value:
```json
{
  "type": "reference",
  "ref": "$.input.header:$.Authorization"
}
```

### Nested Field
Extract from nested object:
```json
{
  "type": "reference",
  "ref": "$.input.body:$.owner.userId"
}
```

### Array Elements
All items in array:
```json
{
  "type": "reference",
  "ref": "$.input.body:$.attachmentIds[*]"
}
```

---

## Common Patterns

### Request Body Fields
Extract multiple fields from body:
```json
"generators": {
  "ticket_id": [{
    "type": "reference",
    "ref": "$.input.body:$.ticketId"
  }],
  "status": [{
    "type": "reference",
    "ref": "$.input.body:$.status"
  }],
  "priority": [{
    "type": "reference",
    "ref": "$.input.body:$.priority"
  }]
}
```

### Query & Path Parameters
Pagination and routing:
```json
"generators": {
  "department_id": [{
    "type": "reference",
    "ref": "$.input.path:$.departmentId"
  }],
  "limit": [{
    "type": "reference",
    "ref": "$.input.query:$.limit"
  }],
  "offset": [{
    "type": "reference",
    "ref": "$.input.query:$.offset"
  }]
}
```

### Mixed Sources
Combine body, path, query, and headers:
```json
"generators": {
  "ticket_id": [{
    "type": "reference",
    "ref": "$.input.body:$.ticketId"
  }],
  "department_id": [{
    "type": "reference",
    "ref": "$.input.path:$.departmentId"
  }],
  "api_key": [{
    "type": "reference",
    "ref": "$.input.header:$.X-API-Key"
  }],
  "include_comments": [{
    "type": "reference",
    "ref": "$.input.query:$.includeComments"
  }]
}
```

### Nested Objects
Extract deeply nested values:
```json
"generators": {
  "contact_id": [{
    "type": "reference",
    "ref": "$.input.body:$.contact.id"
  }],
  "contact_name": [{
    "type": "reference",
    "ref": "$.input.body:$.contact.name"
  }],
  "company_name": [{
    "type": "reference",
    "ref": "$.input.body:$.owner.company.name"
  }]
}
```

---

## Reference Path Cheat Sheet

```
Simple Field:
  $.input.body:$.fieldName

Nested (3 levels):
  $.input.body:$.level1.level2.fieldName

Array All:
  $.input.body:$.items[*].id

Array First:
  $.input.body:$.items[0].id

Query Parameter:
  $.input.query:$.parameterName

Path Parameter:
  $.input.path:$.pathParamName

Header:
  $.input.header:$.Authorization
  $.input.header:$.X-Custom-Header

Deep Nesting:
  $.input.body:$.a.b.c.d.e.fieldName
```

---

## Best Practices

**Do:**
- Verify request contains the fields you reference
- Use exact field names (case-sensitive)
- Test with actual request samples
- Use clear generator names matching source

**Avoid:**
- Referencing fields that don't exist
- Assuming request structure without docs
- Using REFERENCE for static values
- Ignoring case sensitivity

---

## When to Use REFERENCE

| Scenario | Use REFERENCE | Why |
|----------|---|---|
| Pass-through from request | Yes | Direct from request |
| Get data from request | Yes | No transformation |
| Fetch from database | No | Use DYNAMIC |
| Fixed constant | No | Use STATIC |
| System timestamp | No | Use REMOTE |
| Logic-based value | No | Use CONDITIONAL |

---

## Troubleshooting

### Field Not Found / Null
**Check:**
- Exact field name and spelling
- JSONPath is case-sensitive
- Field exists in request sample

### Array Handling Wrong
**Use:**
- `[*]` for all elements
- `[0]` for first element
- `[2]` for specific index

### Headers Not Working
**Check:**
- Headers are case-sensitive
- Format: `X-API-Key` not `x-api-key`
- Verify header exists in request

### Path Parameter Incorrect
**Remember:**
- Path params from URL
- Different from query params (?)
- Different from body params

---

## Reference

- **Fetch Method:** Request fields
- **Data Type:** Request data (pass-through)
- **Scope:** Single request scope
- **Updates:** From request content

---

*Last Updated: 11 February 2026*
