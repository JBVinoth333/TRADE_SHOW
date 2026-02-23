# Dynamic Generator

## Purpose
Fetch real system data by calling an API operation and extracting values from the response.

## Use It When
- You need real, existing IDs or records.
- Values change frequently (agents, departments, tickets, contacts).

## Required Fields
- `type`: "dynamic"
- `generatorOperationId`
- `dataPath`

## Optional Fields
- `params`
- `name`

## DataPath Format
`$.response.body:$.path.to.value`

Common examples:
- `$.response.body:$.data[*].id`
- `$.response.body:$.data[0].id`
- `$.response.body:$.manager.id`

## Dependency Rule (Important)
When this generator depends on another generator, always reference the dependency as:
`$<name>.value`

## Example (Fetch Departments)
```json
{
  "type": "dynamic",
  "name": "departments",
  "generatorOperationId": "support.Department.getDepartments",
  "dataPath": "$.response.body:$.data[*].id",
  "params": {
    "isEnabled": true
  }
}
```

## Example (Create Ticket Using Dependencies)
```json
{
  "type": "dynamic",
  "name": "ticket",
  "generatorOperationId": "support.Ticket.createTicket",
  "dataPath": "$.response.body:$.id",
  "params": {
    "departmentId": "$departments.value",
    "contactId": "$contact.value",
    "subject": "Test ticket"
  }
}
```

## Checklist
- `generatorOperationId` exists in OpenAPI.
- `dataPath` matches real response shape.
- Dependency params use `$<name>.value`.
