# Test Data Generation Configuration Specification

## 1. Introduction

### 1.1 Overview
The Data Generation Framework generates test data based on OpenAPI specifications. Developers create generator configuration files that define how data is fetched, generated, or derived. These files must follow the strict patterns defined in Generator_Patterns and the generator creation agent rules.

## 2. Paths And Folder Layout
Paths are defined in PathConfig.properties and must be used as the source of truth:

- GENERATOR_CONFIG_PATH=../source/api-data-generators/support/
- OAS_FILE_PATH=../source/openapi-specifications/v1.0/support/

New generator files are always created under Created_Generators with a new subfolder per generator set. Each subfolder contains a test_data_generation_configurations.json file.

## 3. Configuration File Structure
Each generator file contains only one top-level key: "generators".

```json
{
  "generators": {
    "<generator_name>": [
      {}
    ]
  }
}
```

Rules:
- Only the "generators" object is allowed in the file.
- All dependencies for a generator must be included in the same array under that generator name.
- Generator names must be snake_case, unique, and meaningful.
- Use singular names for single values and plural names for lists.

## 4. Generator Naming And Ordering Rules

### 4.1 Generator Name Rules
- Snake_case only, lowercase with underscores.
- Use short, meaningful names tied to the entity and purpose.
- Use singular for single values (ticket_id), plural for lists (ticket_ids).

### 4.2 Ordering Rules
- If Generator A depends on Generator B, Generator B must appear earlier in the same array.

### 4.3 Name Field Inside Generators
- When a generator includes the "name" field, use the entity name in snake_case.
- Use plural for lists and singular for single values (departments, contact, tickets).

## 5. Generator Types
Only the following generator types are supported. Do not invent new types.

### 5.1 Dynamic Generator
Purpose: Fetch real system data by calling API operations.

Required properties:
- type: "dynamic"
- generatorOperationId
- dataPath

Optional properties:
- params
- name

DataPath format:
- $.response.body:$.path.to.data

Dependency rule:
- When a dynamic generator depends on another dynamic generator, always reference the dependency in params as $<generatorname>.value, regardless of whether the dataPath returns a single value or an array.

Operation ID rule:
- generatorOperationId must follow <service>.<Entity>.<operation> (for example, support.Agent.getAgents).

### 5.2 Remote Generator
Purpose: Call system functions for timestamps, enums, and configuration data.

Required properties:
- type: "remote"
- generatorMethod
- inputs

Optional properties:
- name

Supported method format:
- applicationDriver.rpc.desk.DynamicDataProvider.methodName

### 5.3 Static Generator
Purpose: Provide fixed, hardcoded values.

Required properties:
- type: "static"
- value

Optional properties:
- name

### 5.4 Reference Generator
Purpose: Read data from request inputs.

Required properties:
- type: "reference"
- ref

Optional properties:
- name

Reference path format:
- $.input.<location>:$.<jsonPath>

Supported locations:
- body
- query
- path
- header

### 5.5 Conditional Generator
Purpose: Select a generator based on input conditions.

Required properties:
- type: "conditional"
- conditions

Optional properties:
- else

Condition format:
- when: { key, equals }
- then: { use }

## 6. Reference Syntax Rules

### 6.1 JSONPath Syntax
Format:
- $.location:$.path

Common patterns:
- $.data[*].id
- $.data[0].id
- $.manager.id
- $.users[?(@.active == true)].id

### 6.2 Generator Reference Syntax
- Local generator reference: $<generatorname>.value
- Cross-file reference: use a relative path (for example ../Department/test_data_generation_configurations.json#/generators/department_id)

## 7. OpenAPI Alignment Requirements
- Always read the OpenAPI specification for the target operation before creating a generator.
- Use only parameters explicitly defined for that operation (body, query, path, header).
- Do not invent fields, wrapper keys, or placeholders.
- If required schema details are missing, request clarification instead of assuming.

## 8. Generator Creation Rules
- Always create a new generator file for new work. Do not modify existing generator files without explicit permission.
- Do not edit OpenAPI specification files.
- Do not output anything except the "generators" object in generator JSON files.
- Follow the exact structure and syntax in Generator_Patterns and README.md.

## 9. Examples

### 9.1 Dynamic Dependency Example
```json
{
  "generators": {
    "ticket_create": [
      {
        "type": "dynamic",
        "name": "departments",
        "generatorOperationId": "support.Department.getDepartments",
        "dataPath": "$.response.body:$.data[*].id"
      },
      {
        "type": "dynamic",
        "name": "ticket",
        "generatorOperationId": "support.Ticket.createTicket",
        "params": {
          "departmentId": "$departments.value"
        }
      }
    ]
  }
}
```

### 9.2 Reference Example
```json
{
  "generators": {
    "ticket_id": [
      {
        "type": "reference",
        "ref": "$.input.path:$.ticketId"
      }
    ]
  }
}
```

## 10. Maintainability Guidelines
- Keep generator arrays focused and short.
- Use descriptive generator names to avoid collisions.
- Keep conditional logic shallow and readable.
- Validate dataPath and reference paths against the actual response or input structures.
