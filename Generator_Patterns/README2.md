# Data Generation Framework - Rules & Instructions

## Overview

A framework that generates test data for OpenAPI specifications. Define generator rules once and generate consistent test data for all parameters.

**File Location:** `api-data-generators/support/{EntityName}/test_data_generation_configurations.json`

**Basic Structure:**
```json
{
  "generators": {
    "generator_id": [ { "type": "...", ...properties... } ]
  }
}
```

## Rules & Instructions

### 1. Five Generator Types

| Type | Purpose | When to Use |
|------|---------|-------------|
| [Dynamic](./dynamic.md) | Fetch IDs from APIs | Get actual system data |
| [Remote](./remote.md) | Call system functions | Timestamps, enums, system values |
| [Static](./static.md) | Fixed constant values | Predetermined values |
| [Reference](./reference.md) | Use request data | Pass-through from incoming request |
| [Conditional](./conditional.md) | Logic-based branching | Value depends on conditions |

### 2. Three Core Rules

Every generator MUST follow these rules:

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
# Data Generation Framework — Common Instructions

This file is the single source of guidance for creating and using generator definitions across entities. Generators live here:

`source/api-data-generators/support/{Entity}/test_data_generation_configurations.json`

Minimal file shape:

```json
{
  "generators": {
    "generator_id": [ { "type": "...", "...": "..." } ]
  }
}
```

## Generator types (summary)

- Dynamic — invoke an API operation and extract fields (`generatorOperationId`, `dataPath`). See `dynamic.md`.
- Remote — call external/system code (Java) to produce values (`generatorMethod`, `params`). See `remote.md`.
- Conditional — choose a generator based on input conditions (`conditions`, `else`). See `conditional.md`.
- Static — return fixed values (`value`). See `static.md`.
- Reference — extract values from prior responses or inputs (`ref`). See `reference.md`.

## Core rules

1. Each generator entry MUST include `type`.
2. Generators are arrays: `"my_id": [ { ... } ]`.
3. Use `snake_case` for IDs and append `_id` for identifiers.
4. Cross-file references use: `"../Entity/test_data_generation_configurations.json#/generators/name"`.
5. For multi-value parameters append `[*]` to the parameter name (e.g. `agentIds[*]`).

## Quick examples

Dynamic example
```json
"generators": {
  "snippet_id": [{
    "type": "dynamic",
    "generatorOperationId": "Snippets.create",
    "dataPath": "$.response.body:$.snippetId",
    "params": { "name": "static_value" }
  }]
}
```

Remote example
```json
"generators": {
  "snippet_id": [{
    "type": "remote",
    "generatorMethod": "org.example.TestData.generateSnippetId",
    "params": { "name": "static_value" }
  }]
}
```

Conditional example
```json
"generators": {
  "snippet_id": [{
    "type": "conditional",
    "conditions": [
      { "when": { "key": "$.input.query:modules", "equals": "tickets" }, "then": { "use": "$generators:#/generators/ticket_id" } }
    ],
    "else": "$generators:#/generators/activity_id"
  }]
}
```

Reference (JSON path)
```json
"ref": "$.response.body:$.data[*].id"
```

Generator group ref
```json
"ref": "$generators:#/generators/agent_id"
```

## Best practices

- Validate `generatorOperationId` against the OpenAPI operation IDs.
- Prefer referencing existing generator groups to avoid duplication.
- Document which operation or request provides values for `reference` generators.
- Use `remote` generators for system-driven values (timestamps, sequence values, enums).

## How to create a generator (steps)

1. Inspect the OpenAPI parameter (type, location, description).
2. Choose generator type: dynamic, remote, static, reference, or conditional.
3. Create the generator entry under `generators` in the entity's `test_data_generation_configurations.json`.
4. Test by running the generation harness (or invoking the generator runner) and confirm outputs.

## Common mistakes

- Omitting `type` in generator objects.
- Using object instead of array for a generator (must be `[ { ... } ]`).
- Incorrect relative path in cross-file references (use `../Entity/...` and include the filename).
- Forgetting `[*]` on multi-value parameters.

---

*Last Updated: 14 February 2026*
