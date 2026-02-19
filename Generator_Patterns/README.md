# Data Generation Framework

## Description

This project is a framework for generating test data for OpenAPI specifications. It allows you to reuse existing generators and OAS files to quickly build new generators for your APIs.

## Table of Contents

- [Data Generation Framework](#data-generation-framework)
  - [Description](#description)
  - [Table of Contents](#table-of-contents)
  - [Usage](#usage)
    - [Path Configurations](#path-configurations)
    - [Basic Generator Structure](#basic-generator-structure)
  - [Features](#features)
  - [Generator Types](#generator-types)
    - [JSONPath Syntax](#jsonpath-syntax)

## Usage

### Path Configurations

 **Generator configuration files:**
   See `PathConfig.properties` → `GENERATOR_CONFIG_PATH`
   Example: Replace `EntityName` with your entity, e.g., `Ticket`
 **OpenAPI Specifications (OAS) files:**
   See `PathConfig.properties` → `OAS_FILE_PATH`
   Example: Replace `EntityName` with your entity, e.g., `Ticket`

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

### JSONPath Syntax

Format: `$.location:$.path`

Common patterns:
- `$.data[*].id` - All IDs from array
- `$.data[0].id` - First element only
- `$.manager.id` - Nested field
- `$.users[?(@.active == true)].id` - Filtered results

*Last Updated: 18 February 2026**ss*