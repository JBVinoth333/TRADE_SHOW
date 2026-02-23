# Generator Framework Documentation

## 1) Purpose

This framework helps developers generate test data for APIs quickly and consistently. It supports:

- **Operational mapping configs** for API test execution (`apis` + `generators`)
- **Scenario configs** for chained setup flows (`generators` only)

---

## 2) Actual Framework Data (No Path Mapping)

The framework currently contains:

- **Support OAS modules:** 387
- **Portal OAS modules:** 55
- **Support generator config sets:** 284
- **Portal generator config sets:** 41
- **Scenario generator sets:** 6

Commonly used entities in generator chains:
- Ticket
- Department
- Contact
- Agent
- Product

Commonly used operation IDs in existing configs:
- `support.Department.getDepartments`
- `support.Contact.createContact`
- `support.Ticket.createTicket`
- `support.Ticket.mergeTicket`
- `portal.Product.listAllProducts`

---

## 3) Verified Runtime Facts (23 Feb 2026)

The documentation below is based on current files and counts in this repo:

- OpenAPI files for support services: **387**
- OpenAPI files for portal services: **55**
- Generator config files for support services: **284**
- Generator config files for portal services: **41**
- Scenario generator folders: **6**
- Scenario generator files: **6**

Concrete data points validated from current content:

- Ticket OpenAPI definitions include `operationId: "createTicket"`.
- Multiple scenario files call `support.Ticket.createTicket` in `generatorOperationId`.
- Existing runtime files use both local references (`#/generators/...`) and cross-file references (`../Entity/...#/generators/...`).
- Response-code-specific parameter mapping is actively used (`200`, `204`, `422`) in `apis` sections.

---

## 4) Configuration Models Used in This Repo

### 4.1 Full runtime model (`apis` + `generators`)

Used across current support and portal generator definitions for runtime API parameter mapping.

Typical structure:

```json
{
  "apis": {
    "operationName": {
      "200": {
        "paramName": "#/generators/some_generator"
      }
    }
  },
  "generators": {
    "some_generator": [
      {
        "type": "dynamic",
        "generatorOperationId": "support.Entity.operation",
        "dataPath": "$.response.body:$.data[*].id"
      }
    ]
  }
}
```

Meaning:
- `apis` maps **operation + status code + request parameter** to generator references.
- `generators` defines how values are fetched/generated.

Actual example from existing runtime style:

```json
{
  "apis": {
    "createTicket": {
      "200": {
        "departmentId": "../Department/test_data_generation_configurations.json#/generators/department_id",
        "contactId": "../Contact/test_data_generation_configurations.json#/generators/contact_id",
        "priority": "#/generators/ticket_priority"
      },
      "422": {
        "departmentId": "../Department/test_data_generation_configurations.json#/generators/department_id",
        "contactId": "../Contact/test_data_generation_configurations.json#/generators/contact_id",
        "priority": "#/generators/ticket_priority"
      }
    }
  }
}
```

### 4.2 Scenario chain model (`generators` only)

Used in `Created_Generators/*`, for example:
- `CreateTicket`
- `CreateTenTickets`
- `MergeTicket`
- `MergeThreeTickets`

Typical structure:

```json
{
  "generators": {
    "scenario_name": [
      { "type": "dynamic", "name": "departments", "generatorOperationId": "support.Department.getDepartments", "dataPath": "$.response.body:$.data[*].id" },
      { "type": "dynamic", "name": "ticket", "generatorOperationId": "support.Ticket.createTicket", "params": { "departmentId": "$departments.value" }, "dataPath": "$.response.body:$.id" }
    ]
  }
}
```

Meaning:
- One ordered generator pipeline, where later steps consume earlier outputs.

---

## 5) Developer Quickstart (Create a Generator in 5 Steps)

1. Pick the target API operation (`operationId`) in the OpenAPI spec.
2. List required inputs (query/path/body/header).
3. Add dependency generators first (department, contact, agent, etc.).
4. Use only OAS-defined fields in `params`.
5. Validate references and JSONPaths before running.

---

## 6) Generator Types (Easy-to-Read Definitions)

Below are the five supported generator types with clear purpose, required fields, and examples.

### 6.1 Dynamic

**Purpose:** Fetch real system data by calling an API operation.

**Required fields:**
- `type`: "dynamic"
- `generatorOperationId`
- `dataPath`

**Optional fields:**
- `params`
- `name`

**Example (fetch department IDs):**

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

**Dependency rule:**
When a dynamic generator depends on another generator, always reference the value as `$<name>.value`.

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

### 6.2 Remote

**Purpose:** Call system functions to generate timestamps, enums, and schema values.

**Required fields:**
- `type`: "remote"
- `generatorMethod`
- `inputs`

**Optional fields:**
- `name`

**Example (enum values):**

```json
{
  "type": "remote",
  "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDynamicEnumValues",
  "inputs": {
    "fieldName": "priority",
    "moduleName": "Ticket",
    "orgId": "16977187"
  }
}
```

### 6.3 Static

**Purpose:** Use fixed, constant values.

**Required fields:**
- `type`: "static"
- `value`

**Optional fields:**
- `name`

**Example:**

```json
{
  "type": "static",
  "value": "OPEN"
}
```

### 6.4 Reference

**Purpose:** Pass values directly from incoming request data.

**Required fields:**
- `type`: "reference"
- `ref`

**Optional fields:**
- `name`

**Example (path param):**

```json
{
  "type": "reference",
  "ref": "$.input.path:$.ticketId"
}
```

### 6.5 Conditional

**Purpose:** Choose a generator based on input conditions.

**Required fields:**
- `type`: "conditional"
- `conditions`

**Optional fields:**
- `else`

**Example:**

```json
{
  "type": "conditional",
  "conditions": [
    {
      "when": {
        "key": "$.input.query:modules",
        "equals": "tickets"
      },
      "then": {
        "use": "$generators:#/generators/ticket_id"
      }
    }
  ],
  "else": "$generators:#/generators/contact_id"
}
```

---

## 7) Syntax Rules You Must Follow

### 6.1 JSONPath format

Use:
`$.location:$.path`

Examples:
- `$.response.body:$.data[*].id`
- `$.input.body:$.ticketId`
- `$.input.query:$.status`
- `$.input.path:$.ticketId`
- `$.input.header:$.Authorization`

### 6.2 Dynamic dependency references

For dependent dynamic params, use:
`$<name>.value`

Example from generated scenarios:
- `departmentId: "$departments.value"`
- `contactId: "$contact.value"`

### 6.3 Cross-file references

Runtime `apis` mappings commonly use:
- local: `#/generators/ticket_id`
- relative: `../Department/test_data_generation_configurations.json#/generators/department_id`

### 6.4 Operation ID format

Use:
`<service>.<Entity>.<operation>`

Examples:
- `support.Ticket.createTicket`
- `support.Department.getDepartments`
- `portal.Product.listAllProducts`

---

## 8) OAS-Driven Generation Process

The required workflow (from agent rules + current file patterns):

1. Identify target operation in OAS (`operationId`, parameter locations, request schema).
2. Determine dependencies required to satisfy mandatory fields.
3. Add dependency generators first (ordering matters).
4. Use only OAS-defined fields in generator `params`.
5. Map generated outputs either:
   - into scenario chain steps (`Created_Generators/*`), or
  - into `apis` parameter bindings for runtime operation/status mappings.

Validation checkpoint:
- Never invent parameters not defined by OAS.
- Match location type correctly (`body`, `query`, `path`, `header`).

---

## 9) Naming and Ordering Conventions

From agent and instruction files:
- Use **snake_case** names.
- Use singular names for single values and plural for arrays/lists.
- Keep names meaningful (`target_ticket`, `source_ticket_one`, `departments`).
- Place dependencies before dependents in each generator array.

Additional practical convention in core files:
- Generator keys may include array markers in runtime mappings (example: `secondary_department_id[*]`).

---

## 10) Practical Patterns Seen in Existing Files

### 9.1 Ticket creation chain
Observed in `Created_Generators/CreateTicket/*` and `CreateTenTickets/*`:
- fetch department
- create/fetch contact
- create one or more tickets using dependent values

### 9.2 Merge workflow
Observed in `Created_Generators/MergeTicket/*` and `MergeThreeTickets/*`:
- create target ticket
- create one or more source tickets
- call `support.Ticket.mergeTicket` with `ticketId` + `ids`

### 9.3 Enum and timestamp generation
Observed in current Ticket generator definitions:
- use `remote` + `getDynamicEnumValues`
- use `remote` + `getCustomFieldSchema`

### 9.4 API parameter binding matrix
Observed across support/portal runtime configs:
- `apis -> operation -> status -> param -> generator ref`
- supports both success and error response-code scenarios (e.g., `200`, `204`, `422`)

---

## 11) Agent Contract Summary

From `.github/agents/Generators.agent.md`:
- read patterns first, then OAS, then build generators
- create new generator files for new work
- output only generator JSON object during agent generation flow
- avoid editing OAS files
- ensure short-hand references resolve in same file unless explicit cross-file path is used

---

## 12) Common Pitfalls to Avoid

1. Missing `$` in dependency reference (must be `$name.value`).
2. Using params not present in OAS operation schema.
3. Wrong JSONPath location (`$.input.path` vs `$.input.query`).
4. Adding unsupported generator types.
5. Dependency steps placed after consumer step.
6. Mixing runtime mapping style and scenario style unintentionally.

---

## 13) Minimum Quality Checklist

Before finalizing any generator file:
- [ ] Correct top-level structure for target use case (`generators` only vs `apis` + `generators`)
- [ ] Valid generator type and required fields
- [ ] OAS operation/params verified
- [ ] Reference paths and JSONPath checked
- [ ] Naming is snake_case and meaningful
- [ ] Dependency ordering is correct

---

## 14) Canonical Example Snippets

### Scenario chain (create + comment)

```json
{
  "generators": {
    "create_ticket_with_comments": [
      {
        "type": "dynamic",
        "generatorOperationId": "support.Department.getDepartments",
        "dataPath": "$.response.body:$.data[*].id",
        "name": "departments",
        "params": { "isEnabled": true }
      },
      {
        "type": "dynamic",
        "generatorOperationId": "support.Contact.createContact",
        "dataPath": "$.response.body:$.id",
        "name": "contact"
      },
      {
        "type": "dynamic",
        "generatorOperationId": "support.Ticket.createTicket",
        "dataPath": "$.response.body:$.id",
        "name": "ticket",
        "params": {
          "departmentId": "$departments.value",
          "contactId": "$contact.value",
          "subject": "Test ticket"
        }
      },
      {
        "type": "dynamic",
        "generatorOperationId": "support.TicketComment.createTicketComment",
        "dataPath": "$.response.body:$.id",
        "params": {
          "ticketId": "$ticket.value",
          "content": "First comment"
        }
      }
    ]
  }
}
```

---

## 15) Final Notes

This documentation is derived from the current repository state across:
- all files in `Generator_Patterns`
- existing generator configs and created scenario generators
- current support and portal OpenAPI sets
- `.github/agents/Generators.agent.md`
