# REFERENCE Generator - Extract Data from Previous API Responses

## What It Does

A Reference Generator extracts data from previous API responses and maps it to current request parameters. It captures values generated or returned by earlier operations and reuses them across sequential API calls.

---

## When to Use REFERENCE

Use REFERENCE when:
A Reference Generator extracts data from previous API responses.- You need data from a prior API response
- Extracting IDs from create operations
- Passing response values to dependent operations
- Chaining API calls with response data
- Building sequences where one response feeds into the next request

**Common Scenarios:**
- After creating a ticket, use the returned ticket ID in subsequent operations
- Extract agent ID from a get operation and use it in an update
- Use contact ID from response to create related entities
- Forward resource IDs between related API chains
- Build dependent request flows

---

## Required Properties

| Property | Type | Purpose |
|----------|------|---------|
| `type` | string | Must be `"reference"` |
| `ref` | string | Path to response data to extract |

## Optional Properties

| Property | Type | Purpose |
|----------|------|---------|
| `name` | string | Generator identifier |

---

## Reference Path Format

**Pattern:** `$.response.body:$.{jsonPath}` or `$.input.body:$.{jsonPath}`

### Source Locations

| Source | Pattern | Example |
|--------|---------|---------|
| Previous response body | `$.response.body:$.path.to.field` | `$.response.body:$.id` |
| Current request body | `$.input.body:$.path.to.field` | `$.input.body:$.ticketId` |
| Request query params | `$.input.query:$.parameterName` | `$.input.query:$.departmentId` |
| Request path params | `$.input.path:$.parameterName` | `$.input.path:$.ticketId` |

### Path Selector Examples

| Reference | Extracts |
|-----------|----------|
| `$.response.body:$.id` | Simple ID from response |
| `$.response.body:$.data[0].id` | First item ID |
| `$.response.body:$.data[*].id` | All item IDs |
| `$.response.body:$.owner.id` | Nested field |
| `$.input.body:$.fieldName` | From current request body |

---

## Examples

### Extract ID from Response
Capture created resource ID:
```json
{
  "type": "reference",
  "ref": "$.response.body:$.id"
}
```

### Extract First Item from Array
Get first element from response list:
```json
{
  "type": "reference",
  "ref": "$.response.body:$.data[0].id"
}
```

### Extract Nested Field
Get nested value from response:
```json
{
  "type": "reference",
  "ref": "$.response.body:$.owner.id"
}
```

### From Request Body
Pass through request field:
```json
{
  "type": "reference",
  "ref": "$.input.body:$.ticketId"
}
```

### All Array Items
Extract all IDs from response array:
```json
{
  "type": "reference",
  "ref": "$.response.body:$.data[*].id"
}
```

---

## Common Patterns

### Sequential API Calls
Chain operations using response data:
```json
"generators": {
  "created_ticket_id": [{
    "type": "reference",
    "ref": "$.response.body:$.id"
  }],
  "created_contact_id": [{
    "type": "reference",
    "ref": "$.response.body:$.contactId"
  }],
  "created_agent_id": [{
    "type": "reference",
    "ref": "$.response.body:$.assigneeId"
  }]
}
```

### Response to Request Forward
Use previous response in current request:
```json
"generators": {
  "ticket_id": [{
    "type": "reference",
    "ref": "$.response.body:$.data[0].id"
  }],
  "status": [{
    "type": "reference",
    "ref": "$.response.body:$.status"
  }]
}
```

### Mixed Response and Request
Combine both response and request data:
```json
"generators": {
  "department_id": [{
    "type": "reference",
    "ref": "$.response.body:$.departmentId"
  }],
  "query_limit": [{
    "type": "reference",
    "ref": "$.input.query:$.limit"
  }],
  "request_priority": [{
    "type": "reference",
    "ref": "$.input.body:$.priority"
  }]
}
```

---

## Best Practices

**Do:**
- Verify the path exists in the API response
- Use exact JSONPath (case-sensitive)
- Test with actual response samples
- Document which operation feeds the reference
- Use descriptive generator names

**Avoid:**
- Referencing non-existent response fields
- Assuming response structure without testing
- Missing array indices (use `[0]` or `[*]`)
- Hard-coding values that should be referenced

---

## When to Use REFERENCE

| Scenario | Use REFERENCE | Why |
|----------|---|---|
| Extract ID from API response | Yes | Direct response data |
| Use created resource ID | Yes | Sequential dependency |
| Pull request body field | Yes | Pass-through value |
| Fixed constant value | No | Use STATIC |
| Fetch dynamic system data | No | Use DYNAMIC |
| Logic-based transformation | No | Use CONDITIONAL |
| RPC/remote data | No | Use REMOTE |

---

## Troubleshooting

### Field Not Found / Null
**Check:**
- JSONPath matches actual response structure
- Field exists in response sample
- Case sensitivity is exact

### Array Index Issues
**Use:**
- `[0]` for first element
- `[*]` for all elements
- `[2]` for specific index

### Response vs Request Confusion
**Remember:**
- Previous/current response: `$.response.body:...`
- Request body: `$.input.body:...`
- Query params: `$.input.query:...`
- Path params: `$.input.path:...`

---

## Reference

- **Fetch Method:** API response extraction
- **Data Type:** Real response data (captured)
- **Scope:** Dependent on prior operation
- **Updates:** Changes with each new response

---

*Last Updated: 13 February 2026*
