# Data Generation Framework - Usage & Rules

## Overview

This framework generates test data for OpenAPI specifications. You can reuse existing generators and OAS files to quickly build new generators for your APIs.

### Path Configurations

#### Path Configurations

All generator and OpenAPI specification path configurations are as follows:

- **Generator configuration files:**
  - `source/api-data-generators/support/{EntityName}/test_data_generation_configurations.json`
  - Example: `source/api-data-generators/support/Ticket/test_data_generation_configurations.json`

- **OpenAPI Specifications (OAS) files:**
  - `source/openapi-specifications/v1.0/support/{EntityName}.json`
  - Example: `source/openapi-specifications/v1.0/support/Ticket.json`

Refer to these paths for the latest and correct locations of generator configuration files and OAS files.

### How to Use Existing Generators

1. **Reference a generator** from another entity using the `$ref` syntax:
   - Example:
     ```json
     "contactId": "../Contact/test_data_generation_configurations.json#/generators/contact_id"
     ```

### Basic Generator Structure

```json
{
  "generators": {
    "generator_id": [ { "type": "...", ...properties... } ]
  }
}
```

## Generator Types

| Type                            | Purpose               | When to Use                        |
| ------------------------------- | --------------------- | ---------------------------------- |
| [Dynamic](./dynamic.md)         | Fetch IDs from APIs   | Get actual system data             |
| [Remote](./remote.md)           | Call system functions | Timestamps, enums, system values   |
| [Static](./static.md)           | Fixed constant values | Predetermined values               |
| [Reference](./reference.md)     | Use request data      | Pass-through from incoming request |
| [Conditional](./conditional.md) | Logic-based branching | Value depends on conditions        |


### 3. Generator Reference Format

| Type           | Format                                 | Example                                  |
| -------------- | -------------------------------------- | ---------------------------------------- |
| Same file      | `#/generators/name`                    | `"agentId": "#/generators/agent_id"`     |
| Different file | `../Entity/file.json#/generators/name` | `"agentId": "../Agent/.../agent_id"`     |
| Array elements | Use `[*]` suffix                       | `"agentIds[*]": "#/generators/agent_id"` |

### 4. Naming Patterns

| Pattern         | Example                                   | Use Case                |
| --------------- | ----------------------------------------- | ----------------------- |
| Entity-Based    | `agent_id`, `ticket_id`                   | Any ID from that entity |
| Status-Based    | `active_agent_id`, `open_ticket_id`       | Specific status         |
| Operation-Based | `created_ticket_id`, `updated_contact_id` | Result of operation     |

### 5. JSONPath Syntax

Format: `$.location:$.path`

Common patterns:
- `$.data[*].id` - All IDs from array
- `$.data[0].id` - First element only
- `$.manager.id` - Nested field

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
- 

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

---

*Last Updated: 17 February 2026*
