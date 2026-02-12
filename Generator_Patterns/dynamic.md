# DYNAMIC Generator - Fetch from API Responses

## What It Does

Fetches IDs and references from actual API calls. Retrieves real system data that must exist in your test environment.

---

## When to Use DYNAMIC

Use DYNAMIC when you need:
- Real entity IDs from the system
- Dependent resource references
- Data that varies per test environment
- Actual data matching system state

**Common Scenarios:**
- Fetch agent IDs from Agent API
- Get ticket IDs from active tickets
- Retrieve department IDs for assignments
- Extract contact IDs for relationships

---

## Required Properties

| Property | Type | Purpose |
|----------|------|---------|
| `type` | string | Must be `"dynamic"` |
| `generatorOperationId` | string | Which API operation to call |
| `dataPath` | string | Where to extract data from response |

## Optional Properties

| Property | Type | Purpose |
|----------|------|---------|
| `params` | object | Filter parameters: `{"status": "ACTIVE"}` |
| `name` | string | Generator identifier |

---

## DataPath Format

**Pattern:** `$.response.body:$.path.to.data`

**Common Selectors:**
- `[*]` - Get all array elements
- `[0]` - Get first element only
- `.fieldName` - Get specific field

**Examples:**
- `$.response.body:$.data[0].id` - First item ID
- `$.response.body:$.data[*].id` - All item IDs
- `$.response.body:$.items[0].manager.id` - Nested field

---

## Examples

### Single Result
Fetch the first matching record:
```json
{
  "type": "dynamic",
  "generatorOperationId": "support.Agent.getAgents",
  "dataPath": "$.response.body:$.data[0].id"
}
```

### Multiple Results
Get all matching records:
```json
{
  "type": "dynamic",
  "generatorOperationId": "support.Agent.getAgents",
  "dataPath": "$.response.body:$.data[*].id"
}
```

### With Filter Parameters
Find specific records:
```json
{
  "type": "dynamic",
  "generatorOperationId": "support.Agent.getAgents",
  "dataPath": "$.response.body:$.data[*].id",
  "params": {"status": "ACTIVE"}
}
```

### Nested Field
Extract from object hierarchy:
```json
{
  "type": "dynamic",
  "generatorOperationId": "support.Department.getDepartment",
  "dataPath": "$.response.body:$.manager.id"
}
```

---

## Common Patterns

### Status Variants
Different generators for different statuses:
```json
"generators": {
  "active_agent_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[*].id",
    "params": {"status": "ACTIVE"}
  }],
  "inactive_agent_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[*].id",
    "params": {"status": "INACTIVE"}
  }]
}
```

### Error Testing
Valid vs invalid data:
```json
"generators": {
  "valid_agent_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[0].id"
  }],
  "invalid_agent_id": [{
    "type": "static",
    "value": 99999
  }]
}
```

### Multiple Entities
Reference different APIs:
```json
"generators": {
  "agent_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[0].id"
  }],
  "department_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Department.getDepartments",
    "dataPath": "$.response.body:$.data[0].id"
  }],
  "contact_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Contact.getContacts",
    "dataPath": "$.response.body:$.data[0].id"
  }]
}
```

---

## Cross-File References

### Reference from Different Entity

Reference a generator defined in another entity's file:
```json
"assigned_agent_id": [
  "#/generators/../Agent/test_data_generation_configurations.json#/generators/agent_id"
]
```

### Multiple Values (Array)
Use `[*]` suffix for array parameters:
```json
"agent_ids[*]": "#/generators/agent_id"
```

---

## Best Practices

**Do:**
- Use params to filter for specific scenarios
- Give descriptive names showing the filter/status
- Verify operationId exists in OpenAPI spec
- Use `[*]` for multiple results, `[0]` for single
- Test the API manually first

**Avoid:**
- Assuming first result is always valid
- Generic names like "id"
- Forgetting `[*]` on array parameters
- Hardcoding fallback without DYNAMIC

---

## Troubleshooting

### Getting Wrong Data or Empty Results
**Check:**
- operationId is spelled correctly
- dataPath matches actual response structure
- Test the API operation manually
- params filter isn't too strict

### Empty or No Results
**Try:**
- Relax filter parameters
- Check if API has matching data
- Use different dataPath expression
- Add STATIC fallback generator

### Wrong Number of Results
**Verify:**
- Using `[*]` for all vs `[0]` for first
- API returns expected array structure
- Check pagination limits

---

## Reference

- **Fetch Method:** API operations
- **Data Type:** Real system data
- **Scope:** Cross-entity references possible
- **Updates:** Auto-syncs with system data

---

*Last Updated: 11 February 2026*
