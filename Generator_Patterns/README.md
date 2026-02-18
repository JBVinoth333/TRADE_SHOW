
# Data Generation Framework

## Description

This project is a framework for generating test data for OpenAPI specifications. It allows you to reuse existing generators and OAS files to quickly build new generators for your APIs.

## Table of Contents

- [Description](#description)
- [Usage](#usage)
- [Features](#features)
- [Generator Types](#generator-types)
- [Reference](#reference)

## Usage

### Path Configurations

- **Generator configuration files:**
  - `source/api-data-generators/support/{EntityName}/test_data_generation_configurations.json`
  - Example: `source/api-data-generators/support/Ticket/test_data_generation_configurations.json`
- **OpenAPI Specifications (OAS) files:**
  - `source/openapi-specifications/v1.0/support/{EntityName}.json`
  - Example: `source/openapi-specifications/v1.0/support/Ticket.json`

Refer to these paths for the latest and correct locations of generator configuration files and OAS files.

### Basic Generator Structure

```json
{
  "generators": {
    "generator_id": [ { "type": "...", ...properties... } ]
  }
}
```


## Features

- Generate test data for OpenAPI specifications
- Reuse and reference generators across entities
- Support for multiple generator types (Dynamic, Remote, Static, Reference, Conditional)
- Flexible path and naming conventions

## Generator Types

| Type                            | Purpose               | When to Use                        |
| ------------------------------- | --------------------- | ---------------------------------- |
| [Dynamic](./dynamic.md)         | Fetch IDs from APIs   | Get actual system data             |
| [Remote](./remote.md)           | Call system functions | Timestamps, enums, system values   |
| [Static](./static.md)           | Fixed constant values | Predetermined values               |
| [Reference](./reference.md)     | Use request data      | Pass-through from incoming request |
| [Conditional](./conditional.md) | Logic-based branching | Value depends on conditions        |

### Naming Patterns

| Pattern         | Example                                   | Use Case                |
| --------------- | ----------------------------------------- | ----------------------- |
| Entity-Based    | `agent_id`, `ticket_id`                   | Any ID from that entity |
| Status-Based    | `active_agent_id`, `open_ticket_id`       | Specific status         |
| Operation-Based | `created_ticket_id`, `updated_contact_id` | Result of operation     |

### JSONPath Syntax

Format: `$.location:$.path`

Common patterns:
- `$.data[*].id` - All IDs from array
- `$.data[0].id` - First element only
- `$.manager.id` - Nested field
- `$.users[?(@.active == true)].id` - Filtered results

### Cross-File References


Reference generators from other entities:

```json
"assigned_agent_id": [
  { "ref": "../Agent/test_data_generation_configurations.json#/generators/agent_id" }
]
```
*Last Updated: 18 February 2026*
