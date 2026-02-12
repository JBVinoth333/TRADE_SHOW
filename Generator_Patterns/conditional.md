# CONDITIONAL Generator - Logic-Based Values

## What It Does

Uses if-then-else logic to choose different generators based on conditions. Most complex generator type - implements branching logic.

---

## When to Use CONDITIONAL

Use CONDITIONAL when:
- Value depends on another field
- Different values for different scenarios
- Optional fields based on conditions
- Status-based variations
- Type-based routing

**Common Scenarios:**
- Different values based on ticket status
- Assign teams based on ticket type
- Escalation based on priority
- Optional fields based on conditions
- Complex nested branching logic

---

## Required Properties

| Property | Type | Purpose |
|----------|------|---------|
| `type` | string | Must be `"conditional"` |
| `condition` | string | What to check |
| `then` | object | Generator if true |

## Optional Properties

| Property | Type | Purpose |
|----------|------|---------|
| `else` | object | Generator if false (defaults to null) |
| `name` | string | Generator identifier |

---

## Condition Format

Simple equality check: `field == 'value'`

### Format Rules

- Use field name on left side
- Use `==` operator (only equality)
- Quote string values: `'value'`
- Don't quote booleans/numbers

### Condition Examples

| Condition | Type | Meaning |
|-----------|------|---------|
| `priority == 'HIGH'` | String | Priority equals HIGH |
| `ticketType == 'BUG'` | String | Type equals BUG |
| `isClosed == true` | Boolean | Closed flag is true |
| `severity == 'CRITICAL'` | String | Severity equals CRITICAL |

---

## Branch Types

Both `then` and `else` branches can use ANY generator:

| Type | Example | Use Case |
|------|---------|----------|
| Static | `{"type": "static", "value": "text"}` | Fixed values |
| Dynamic | `{"type": "dynamic", ...}` | Fetch from API |
| Remote | `{"type": "remote", ...}` | System functions |
| Reference | `{"type": "reference", ...}` | Request data |
| Conditional | `{"type": "conditional", ...}` | Nested conditions |

---

## Examples

### Simple Branching
If-then-else with static values:
```json
{
  "type": "conditional",
  "condition": "priority == 'HIGH'",
  "then": {"type": "static", "value": "ESCALATED"},
  "else": {"type": "static", "value": "NORMAL"}
}
```

### With Dynamic Generator
Fetch different data based on status:
```json
{
  "type": "conditional",
  "condition": "status == 'OPEN'",
  "then": {
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[0].id"
  },
  "else": {"type": "static", "value": null}
}
```

### With Remote Generator
Generate timestamp if closed:
```json
{
  "type": "conditional",
  "condition": "isClosed == true",
  "then": {
    "type": "remote",
    "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDateTime",
    "inputs": {"timeLine": "current", "format": "yyyy-MM-dd'T'HH:mm:ss'Z'"}
  },
  "else": {"type": "static", "value": null}
}
```

### With Reference Generator
Use request data if flag is set:
```json
{
  "type": "conditional",
  "condition": "useRequestId == true",
  "then": {"type": "reference", "ref": "$.input.body:$.preferredAgentId"},
  "else": {
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[0].id"
  }
}
```

### Without Else
Defaults to null if condition false:
```json
{
  "type": "conditional",
  "condition": "department == 'SUPPORT'",
  "then": {
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getSupportAgents",
    "dataPath": "$.response.body:$.agents[0].agentId"
  }
}
```

---

## Common Patterns

### Status-Based Branching
Different data for different statuses:
```json
"escalation_status": [{
  "type": "conditional",
  "condition": "priority == 'HIGH'",
  "then": {"type": "static", "value": "ESCALATED"},
  "else": {"type": "static", "value": "NORMAL"}
}]
```

### Type-Based Branching
Nested conditions for multiple types:
```json
"team_assignment": [{
  "type": "conditional",
  "condition": "ticketType == 'BUG'",
  "then": {"type": "static", "value": "TECHNICAL_TEAM"},
  "else": {
    "type": "conditional",
    "condition": "ticketType == 'FEATURE'",
    "then": {"type": "static", "value": "PRODUCT_TEAM"},
    "else": {"type": "static", "value": "SUPPORT_TEAM"}
  }
}]
```

### Flag-Based Branching
Auto-assign based on flag:
```json
"assigned_agent": [{
  "type": "conditional",
  "condition": "autoAssign == true",
  "then": {
    "type": "dynamic",
    "generatorOperationId": "support.Agent.getAgents",
    "dataPath": "$.response.body:$.data[0].id"
  },
  "else": {"type": "static", "value": null}
}]
```

---

## Nesting Guidelines

**Simple (Recommended):**
```json
"condition": "priority == 'HIGH'"
"then": { ... }
"else": { ... }
```

**Nested If-Else-If:**
Maximum 2-3 nesting levels only.

**Avoid:**
- Deeply nested conditions
- Complex logic (only `==` supported)
- Hard-to-understand branching

---

## Best Practices

**Do:**
- Keep conditions simple and readable
- Provide both `then` and `else` branches
- Use descriptive names showing condition
- Test both branches thoroughly
- Limit nesting to 2 levels maximum

**Avoid:**
- Complex conditions (only `==` works)
- Forgetting to test the `else` branch
- Deep nesting (hard to understand)
- Generic condition names
- Frequent condition changes

---

## When to Use CONDITIONAL

| Scenario | Use CONDITIONAL | Why |
|----------|---|---|
| Value depends on field | Yes | Logical branching |
| Multiple scenarios | Yes | More concise |
| Optional field | Yes | Clearer than null |
| Fixed value | No | Use STATIC |
| API fetch | No | Use DYNAMIC |
| Timestamp | No | Use REMOTE |
| Request data | No | Use REFERENCE |

---

## Troubleshooting

### Condition Not Evaluating
**Check:**
- Spelling of field name
- Quotes around string values
- Format: `field == 'value'`

### Wrong Branch Executing
**Verify:**
- Condition is correct
- Values match expected
- Both branches defined

### Nested Logic Wrong
**Remember:**
- Inner condition is `else` of outer
- Limit nesting (max 2 levels)
- Test each branch

### Null Values
**When:**
- No `else` branch provided
- `else` explicitly `null`
- Condition check is false

---

## Reference

- **Fetch Method:** Conditional logic
- **Data Type:** Any (from branch generator)
- **Scope:** Depends on condition
- **Updates:** Dynamic based on condition

---

*Last Updated: 11 February 2026*
