# STATIC Generator - Fixed Hardcoded Values

## What It Does

Uses a fixed, predetermined value that remains constant. Simplest generator type - directly assigns a hardcoded value.

---

## When to Use STATIC

Use STATIC when:
- Value never changes
- Fixed enum values
- Boolean flags
- Default values
- Testing invalid values
- Tags and categories
- Version numbers

**Common Scenarios:**
- Fixed status code values
- Constant priority levels
- Boolean active/deleted flags
- Default values
- Invalid values for error testing

---

## Required Properties

| Property | Type | Purpose |
|----------|------|---------|
| `type` | string | Must be `"static"` |
| `value` | any | The constant value |

## Optional Properties

| Property | Type | Purpose |
|----------|------|---------|
| `name` | string | Generator identifier |

---

## Supported Value Types

STATIC supports all JSON types:

| Type | Example | JSONPath |
|------|---------|----------|
| String | `"OPEN"` | `"value": "OPEN"` |
| Number | `42` | `"value": 42` |
| Decimal | `99.99` | `"value": 99.99` |
| Boolean | `true` | `"value": true` |
| Null | `null` | `"value": null` |
| Array | `["a", "b"]` | `"value": ["option1"]` |
| Object | `{"key": "val"}` | `"value": {"f_100": "val"}` |

---

## Examples

### String Value
```json
{
  "type": "static",
  "value": "OPEN"
}
```

### Number Value
```json
{
  "type": "static",
  "value": 42
}
```

### Boolean Value
```json
{
  "type": "static",
  "value": true
}
```

### Array Value
```json
{
  "type": "static",
  "value": ["option1", "option2"]
}
```

### Object Value
```json
{
  "type": "static",
  "value": {
    "field_100": "value",
    "field_101": "another"
  }
}
```

### Null Value
```json
{
  "type": "static",
  "value": null
}
```

---

## Common Patterns

### Status Variants
Different status values:
```json
"generators": {
  "status_open": [{"type": "static", "value": "OPEN"}],
  "status_closed": [{"type": "static", "value": "CLOSED"}],
  "status_pending": [{"type": "static", "value": "PENDING"}]
}
```

### Priority Levels
Different priority values:
```json
"generators": {
  "priority_critical": [{"type": "static", "value": "CRITICAL"}],
  "priority_high": [{"type": "static", "value": "HIGH"}],
  "priority_low": [{"type": "static", "value": "LOW"}]
}
```

### Boolean Flags
On/off values:
```json
"generators": {
  "is_active": [{"type": "static", "value": true}],
  "is_deleted": [{"type": "static", "value": false}]
}
```

### Tags/Arrays
Array of values:
```json
"generators": {
  "default_tags": [{"type": "static", "value": ["urgent", "escalated"]}],
  "permissions": [{"type": "static", "value": ["read", "write"]}]
}
```

### Error Testing
Valid vs invalid:
```json
"generators": {
  "valid_status": [{"type": "static", "value": "OPEN"}],
  "invalid_status": [{"type": "static", "value": "INVALID_STATUS"}]
}
```

### Custom Fields
Object structure:
```json
"generators": {
  "custom_fields": [{
    "type": "static",
    "value": {
      "field_100": "Support",
      "field_101": "North America"
    }
  }]
}
```

---

## Type Mapping

Match values to OpenAPI specification types:

| OpenAPI Type | STATIC Value | Example |
|--------------|---|---|
| string | Quoted text | `"OPEN"` |
| integer | Number (no quotes) | `42` |
| number | Decimal (no quotes) | `99.99` |
| boolean | true or false | `true` |
| array | [ ] brackets | `["a", "b"]` |
| object | { } braces | `{"key": "value"}` |
| null | null keyword | `null` |

---

## Best Practices

**Do:**
- Use descriptive names
- Match value type to OpenAPI spec
- Create valid AND invalid variants for testing
- Comment complex object structures

**Avoid:**
- Using STATIC for changing values (use DYNAMIC)
- Using STATIC for timestamps (use REMOTE)
- Using STATIC for request data (use REFERENCE)
- Generic names like "value"

---

## Troubleshooting

### Value Type Mismatch
**Ensure:**
- String values are quoted: `"OPEN"`
- Numbers not quoted: `42`
- Booleans lowercase: `true` not `True`

### Complex Objects Wrong
**Verify:**
- All braces and brackets matched
- Field names quoted: `"field_100"`
- Commas between properties

### Array Elements
**Remember:**
- Single item: `["single"]`
- Multiple: `["one", "two"]`
- Objects in array: `[{"id": 1}]`

---

## Reference

- **Fetch Method:** Hardcoded values
- **Data Type:** Any JSON type
- **Scope:** Fixed per generator
- **Updates:** Never changes

---

*Last Updated: 11 February 2026*
