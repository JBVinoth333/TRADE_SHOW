# Generator Types Quick Comparison

## Side-by-Side Comparison

### STATIC vs DYNAMIC vs REMOTE

```
┌──────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Aspect       │ STATIC           │ DYNAMIC          │ REMOTE           │
├──────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Type         │ "static"         │ "dynamic"        │ "remote"         │
│ Data Source  │ Hard-coded array │ REST API call    │ Java function    │
│ API Call     │ No               │ Yes              │ Maybe            │
│ Org-Aware    │ No               │ No               │ Yes (usually)    │
│ Dynamic      │ No               │ Yes              │ Yes              │
│ Complexity   │ Simple           │ Medium           │ High             │
│ Maintenance  │ Easy             │ Medium           │ Hard             │
│ Speed        │ Fast             │ Medium           │ Medium           │
└──────────────┴──────────────────┴──────────────────┴──────────────────┘
```

---

## Type Selection Decision Table

| Scenario | Use Type | Reason | Example |
|----------|----------|--------|---------|
| **Fixed list of values** | STATIC | Values don't change | `["Open", "Closed"]` |
| **System enum values** | STATIC | Predefined by system | `["High", "Medium", "Low"]` |
| **Need API data (IDs)** | DYNAMIC | Call REST API | Get agent ID from API |
| **Get specific field from API** | DYNAMIC | Extract from response | Extract ID from response body |
| **Org-specific enums** | REMOTE | Custom function call | Get organization's custom statuses |
| **Date computation** | REMOTE | Complex logic | Generate future date |
| **Use another generator** | REFERENCE | Reuse existing | Reference agent_id from Agent module |
| **Conditional values** | CONDITIONAL | Based on conditions | Choose based on request param |

---

## Code Examples

### STATIC: Fixed List
```json
{
  "article_permission": [{
    "type": "static",
    "value": ["ALL", "READ", "WRITE"]
  }]
}
```
✅ Use when: Values never change
❌ Don't use: For dynamic enum values

---

### DYNAMIC: API Call
```json
{
  "agent_id": [{
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getMyInfo",
    "dataPath": "$.response.body:$.id"
  }]
}
```
✅ Use when: Need to call REST API to get data
❌ Don't use: When custom function logic is needed

---

### REMOTE: Custom Function
```json
{
  "ticket_status": [{
    "type": "remote",
    "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDynamicEnumValues",
    "inputs": {
      "fieldName": "status",
      "moduleName": "Ticket",
      "orgId": "$.input.query:$.orgId"
    }
  }]
}
```
✅ Use when: Custom Java function needed (enums, dates, computed values)
❌ Don't use: When simple STATIC or DYNAMIC would work

---

### REFERENCE: Reuse Generator
```json
{
  "departmentId": [{
    "type": "reference",
    "value": "../Department/test_data_generation_configurations.json#/generators/department_id"
  }]
}
```
✅ Use when: Generator already exists in another module
❌ Don't use: For inline values (use STATIC instead)

---

### CONDITIONAL: Based on Conditions
```json
{
  "optional_field": [{
    "type": "conditional",
    "conditions": [
      {
        "if": "$.input.query:$.includeOptional == true",
        "then": { "type": "dynamic", "generatorOperationId": "..." },
        "else": { "type": "static", "value": null }
      }
    ]
  }]
}
```
✅ Use when: Value depends on request parameters or conditions
❌ Don't use: When unconditional generation is sufficient

---

## Real-World Scenario: Ticket Creation

### Scenario 1: Generate Ticket with Different Types

```json
{
  "apis": {
    "createTicket": {
      "201": {
        "subject": "Need help with order",
        "description": "I have a problem",
        "departmentId": "#/generators/department_id",      // DYNAMIC
        "agentId": "#/generators/agent_id",                // DYNAMIC
        "status": "#/generators/ticket_status",            // REMOTE
        "priority": "#/generators/ticket_priority",        // REMOTE
        "dueDate": "#/generators/ticket_due_date",        // REMOTE
        "permission": "#/generators/ticket_permission"     // STATIC
      }
    }
  },
  "generators": {
    // DYNAMIC: Call API to get department
    "department_id": [{
      "type": "dynamic",
      "generatorOperationId": "support.Department.getDepartments",
      "dataPath": "$.response.body:$.data[0].id",
      "name": "dept",
      "params": { "isEnabled": "true" }
    }],
    
    // DYNAMIC: Call API to get agent from the department
    "agent_id": [{
      "type": "dynamic",
      "generatorOperationId": "support.Agent.getAgents",
      "dataPath": "$.response.body:$.data[0].id",
      "params": { "departmentId": "$dept.value" }
    }],
    
    // REMOTE: Fetch organization's custom ticket statuses
    "ticket_status": [{
      "type": "remote",
      "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDynamicEnumValues",
      "inputs": {
        "fieldName": "status",
        "moduleName": "Ticket",
        "orgId": "$.input.query:$.orgId"
      }
    }],
    
    // REMOTE: Fetch organization's custom priorities
    "ticket_priority": [{
      "type": "remote",
      "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDynamicEnumValues",
      "inputs": {
        "fieldName": "priority",
        "moduleName": "Ticket",
        "orgId": "$.input.query:$.orgId"
      }
    }],
    
    // REMOTE: Generate a future due date
    "ticket_due_date": [{
      "type": "remote",
      "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDateTime",
      "inputs": {
        "timeLine": "future",
        "format": "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'"
      }
    }],
    
    // STATIC: Fixed permission value
    "ticket_permission": [{
      "type": "static",
      "value": ["ALL"]
    }]
  }
}
```

---

## Why Different Types?

1. **Department & Agent (DYNAMIC)**
   - Need actual IDs from database
   - API endpoints available
   - Real entities required

2. **Status & Priority (REMOTE)**
   - Organization has custom values
   - Can't hardcode (changes per org)
   - Java function retrieves org-specific enums

3. **Due Date (REMOTE)**
   - Computed value (future date)
   - Can't use STATIC (would be past after day 1)
   - Computed by function

4. **Permission (STATIC)**
   - Fixed value
   - Doesn't change
   - No computation needed

---

## Common Mistakes & Fixes

### ❌ Mistake 1: Using STATIC for Org-Specific Values
```json
// WRONG - Hard-coded status
"status": { "type": "static", "value": ["Open"] }
```

```json
// CORRECT - Fetch org-specific status
"status": [{
  "type": "remote",
  "generatorMethod": "...getDynamicEnumValues",
  "inputs": { "fieldName": "status", "moduleName": "Ticket", "orgId": "..." }
}]
```

### ❌ Mistake 2: Using DYNAMIC for Fixed Values
```json
// WRONG - Unnecessary API call
"locale": [{
  "type": "dynamic",
  "generatorOperationId": "Solution.listLocales",
  "dataPath": "$.response.body:$.locales[*].locale"
}]
```

```json
// CORRECT - Fixed value
"locale": [{
  "type": "static",
  "value": ["en"]
}]
```

### ❌ Mistake 3: Using REMOTE when DYNAMIC would work
```json
// WRONG - Unnecessary complexity
"agent_id": [{
  "type": "remote",
  "generatorMethod": "...getAgentId"
}]
```

```json
// CORRECT - Use REST API
"agent_id": [{
  "type": "dynamic",
  "generatorOperationId": "support.Agent.getMyInfo",
  "dataPath": "$.response.body:$.id"
}]
```

---

## Performance Implications

| Type | Speed | Notes |
|------|-------|-------|
| STATIC | ⚡⚡⚡ Fastest | No network calls |
| DYNAMIC | ⚡⚡ Medium | REST API call required |
| REMOTE | ⚡⚡ Medium | RPC function call required |
| REFERENCE | ⚡⚡⚡ Fast | Just reference lookup |
| CONDITIONAL | ⚡⚡ Medium | Depends on branch taken |

**Note**: Use STATIC first, then DYNAMIC/REMOTE only when needed for data.

---

## Dependency Flow Example

```
START
   │
   ├─ STATIC locale = ["en"]
   │
   ├─ DYNAMIC department_id = API call (uses locale)
   │
   ├─ DYNAMIC agent_id = API call (uses department_id)
   │
   ├─ REMOTE ticket_status = Java function (uses orgId)
   │
   ├─ DYNAMIC ticket = API call (uses all above)
   │
   └─ REMOTE feedback_date = Java function (computed)
```

Each layer depends on previous layers, forming a clean dependency chain.

---

## Summary

| Type | Best For | Effort | Impact |
|------|----------|--------|--------|
| **STATIC** | Fixed values | ⭐ Low | ⚡ High speed |
| **DYNAMIC** | API data | ⭐⭐ Medium | ⚡⚡ Good balance |
| **REMOTE** | Custom logic | ⭐⭐⭐ High | ⚡⚡ Flexible |
| **REFERENCE** | Reuse | ⭐ Low | ⚡⚡⚡ Very efficient |
| **CONDITIONAL** | Smart logic | ⭐⭐⭐ High | ⚡⚡ Context-aware |

Choose based on **where your data comes from** and **how often it changes**.
