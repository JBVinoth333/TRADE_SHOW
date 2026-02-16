# Data Generation Framework - Usage & Rules

## Overview

This framework generates test data for OpenAPI specifications. You can reuse existing generators and OAS files to quickly build new generators for your APIs.

### Where to Find Existing Generators

- **Generators:**
  - All generator configuration files are in:
    - `source/api-data-generators/support/{EntityName}/test_data_generation_configurations.json`
    - Example: `source/api-data-generators/support/Ticket/test_data_generation_configurations.json`

- **OpenAPI Specifications (OAS):**
  - OAS files are in:
    - `source/openapi-specifications/v1.0/support/{EntityName}.json`
    - Example: `source/openapi-specifications/v1.0/support/Ticket.json`

### How to Use Existing Generators

1. **Reference a generator** from another entity using the `$ref` syntax:
   - Example:
     ```json
     "contactId": "../Contact/test_data_generation_configurations.json#/generators/contact_id"
     ```

2. **Copy and modify** generator blocks from existing files to suit your new API or entity.

3. **Check the OAS file** for required fields and operations. Use the operationId and schema from the OAS to guide your generator structure.

### Basic Generator Structure

```json
{
  "generators": {
    "generator_id": [ { "type": "...", ...properties... } ]
  }
}
```

## Generator Types

| Type | Purpose | When to Use |
|------|---------|-------------|
| [Dynamic](./dynamic.md) | Fetch IDs from APIs | Get actual system data |
| [Remote](./remote.md) | Call system functions | Timestamps, enums, system values |
| [Static](./static.md) | Fixed constant values | Predetermined values |
| [Reference](./reference.md) | Use request data | Pass-through from incoming request |
| [Conditional](./conditional.md) | Logic-based branching | Value depends on conditions |

## Core Rules

1. **Specify "type"** - Declare which generator type
   ```json
   { "type": "static", "value": "OPEN" }
   ```
2. **Use arrays** - Wrap all generators in arrays
   ```json
   "status": [ { "type": "static", "value": "OPEN" } ]
   ```

3. **Use snake_case** - Lowercase with underscores, IDs end with `_id`
   ```json
   "agent_id", "ticket_status", "created_timestamp"
   ```

### 3. Generator Reference Format

| Type | Format | Example |
|------|--------|---------|
| Same file | `#/generators/name` | `"agentId": "#/generators/agent_id"` |
| Different file | `../Entity/file.json#/generators/name` | `"agentId": "../Agent/.../agent_id"` |
| Array elements | Use `[*]` suffix | `"agentIds[*]": "#/generators/agent_id"` |

### 4. Naming Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| Entity-Based | `agent_id`, `ticket_id` | Any ID from that entity |
| Status-Based | `active_agent_id`, `open_ticket_id` | Specific status |
| Operation-Based | `created_ticket_id`, `updated_contact_id` | Result of operation |

### 5. JSONPath Syntax

Format: `$.location:$.path`

Common patterns:
- `$.data[*].id` - All IDs from array
- `$.data[0].id` - First element only
- `$.manager.id` - Nested field

## Important File Restrictions

**NEVER edit the following files in existing entities:**
- OpenAPI Specification (OAS) files
- Generator files (test_data_generation_configurations.json)

These files are controlled by system processes. Any edits must be done through proper configuration management.

## Common Mistakes

| Mistake | Wrong | Correct |
|---------|-------|---------|
| Missing array | `"agent_id": { ... }` | `"agent_id": [ { ... } ]` |
| Forgetting `[*]` | `"agentIds": "#/generators/agent_id"` | `"agentIds[*]": "#/generators/agent_id"` |
| Missing type | `{ "value": "OPEN" }` | `{ "type": "static", "value": "OPEN" }` |
| Wrong path | `Agent/file.json` | `../Agent/file.json` |
| Missing generators key | `{ "agent_id": [ ] }` | `{ "generators": { "agent_id": [ ] } }` |

## Entity Relationships

Common entity dependencies:

```
Account ─ Department, Contact, Contract
Agent ─ Department, used in Ticket, Call, Activity
Ticket ─ Contact, Agent, Department
```

Reference related entities using cross-file references.
- `$.users[?(@.active == true)].id` - Filtered results

### 2.9 Cross-File References

Reference generators from other entities:

```json
"assigned_agent_id": [
  { "ref": "../Agent/test_data_generation_configurations.json#/generators/agent_id" }
]
```

**Rules:**
- Use `../` for relative paths
- Include full filename
- Include `#/generators/` prefix
- Entity folder name must match exactly

### 2.10 Common Mistakes to Avoid

| Error | Fix |
|-------|-----|
| `"agent_id": { ... }` | Use array: `[ { ... } ]` |
| `"agentIds": ...` | Add `[*]`: `"agentIds[*]"` |
| Including "apis" section | Remove - only generators |
| Missing `../` in references | Use `../Agent/file.json#/...` |
| Missing `type` property | Add type to every generator |

## 3. Generator Types Quick Reference

### 3.1 Dynamic Generator

Fetch IDs from real API responses. Required: type, generatorOperationId, dataPath. Optional: params,name.

See [dynamic.md](./dynamic.md) for details.

### 3.2 Remote Generator

Call system functions (timestamps, enums, configuration). Required: type, generatorMethod, inputs.

See [remote.md](./remote.md) for details.

### 3.3 Reference Generator

Use data from request (body, query, path, headers). Required: type, ref.

See [reference.md](./reference.md) for details.

### 3.4 Static Generator

Use fixed constant values. Required: type, value.

See [static.md](./static.md) for details.

### 3.5 Conditional Generator

Choose value based on conditions. Required: type, condition, then. Optional: else.

See [conditional.md](./conditional.md) for details.

## 4. Real-World PatternsInputs

### 4.1 Entity Relationships

```
Account
├─ Department
├─ Contact
└─ Contract

Agent
├─ Department
├─ Used in Ticket, Call, Activity
└─ Used in many other entities

Ticket
├─ Contact
├─ Agent
└─ Department

Contact
├─ Account
└─ Used in Ticket, Call
```

When creating generators, reference related entities using cross-file references.

### 4.2 Testing Different Status Codes

Create generators for success and error scenarios:

```json
"generators": {
  "valid_ticket_id": [ { "type": "dynamic", ... } ],
  "invalid_ticket_id": [ { "type": "static", "value": 99999 } ],
  "valid_status": [ { "type": "static", "value": "OPEN" } ],
  "invalid_status": [ { "type": "static", "value": "INVALID_STATUS" } ]
}
```

Use in APIs:
- 200/201 responses: use valid generators
- 404 responses: use invalid_id generators
- 422 responses: use invalid_value generators

### 4.3 Array Parameters

When API accepts multiple values:

```json
"agentIds[*]": "../Agent/test_data_generation_configurations.json#/generators/agent_id"
```

The `[*]` suffix means "use multiple values from this generator".

## 5. How to Create a Generator (Step-by-Step)

### Step 1: Understand Your Parameters
Open your OpenAPI spec and list all parameters with their types and descriptions.

### Step 2: Choose Generator Type for Each Parameter

Examples:
- `departmentId` (integer) → **Dynamic** (fetch from API)
- `status` (string) → **Static** ("OPEN")
- `createdAt` (timestamp) → **Remote** (getDateTime)
- `agentId` (from another entity) → **Dynamic** (fetch from Agent API)

### Step 3: Read the Appropriate Guide

Click the guide link for your generator type (Section 3). Find examples that match your needs.

### Step 4: Create Your Generator File

Create file at: `Generators/Real_Generators/MyEntity/test_data_generation_configurations.json`

```json
{
  "generators": {
    "your_generator_id": [
      {
        "type": "...",
        "property1": "value1",
        "property2": "value2"
      }
    ]
  }
}
```
## 7. Key Principles

- Consistency: Same rules everywhere
- Reusability: Reference generators across APIs
- Clarity: Good names help understanding
---

*Last Updated: 14 February 2026*
