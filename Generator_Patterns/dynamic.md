# DYNAMIC Generator - Fetch from API Responses

## What It Does
A DYNAMIC Generator fetches data from real API responses.
 <!-- It allows you to extract values like IDs, names, or other fields directly from the system under test, ensuring that your tests use valid and up-to-date data. -->
---

## When to Use DYNAMIC
- When you need real system data that changes frequently (e.g., active agent IDs, current ticket IDs).
- When there is an API available to fetch the required data.
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

<!-- **Common Selectors:**
- `[*]` - Get all array elements
- `[0]` - Get first element only
- `.fieldName` - Get specific field -->

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
- Verify operationId exists in OpenAPI specification
- Test the API manually first to confirm dataPath is correct
- 
**Avoid:**
- Assuming first result is always valid
- Generic names like "id"
- Forgetting `[*]` on array parameters
- Hardcoding fallback without DYNAMIC
- 
---

## Check
- operationId is spelled correctly
- dataPath matches actual response structure
- params filter isn't too strict

## Reference
- **Fetch Method:** API operations
- **Data Type:** Real system data
- **Scope:** Cross-entity references possible
- **Updates:** Auto-syncs with system data

---

*Last Updated: 11 February 2026*
