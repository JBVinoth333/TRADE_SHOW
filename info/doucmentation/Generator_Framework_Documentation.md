# Generator Framework Documentation

## 1) Purpose

This framework generates API test data from OpenAPI definitions by combining:
- OpenAPI specs in `source/openapi-specifications/v1.0/*`
- generator configs in `source/api-data-generators/*`
- generator pattern rules in `Generator_Patterns/*`
- creation rules in `.github/agents/Generators.agent.md`

It supports two practical outputs:
- **Operational mapping configs** for API test execution (`apis` + `generators`)
- **Scenario creation configs** under `Created_Generators/*` (`generators` only)

---

## 2) Repository Layout (Framework-Relevant)

### Rule and pattern sources
- `Generator_Patterns/README.md`
- `Generator_Patterns/Static.md`
- `Generator_Patterns/Dynamic.md`
- `Generator_Patterns/Reference.md`
- `Generator_Patterns/Remote.md`
- `Generator_Patterns/Conditional.md`
- `Generator_Patterns/PathConfig.properties`

### Agent behavior contract
- `.github/agents/Generators.agent.md`

### Runtime generator inventories (existing)
- `source/api-data-generators/support/*/test_data_generation_configurations.json`
- `source/api-data-generators/portal/*/test_data_generation_configurations.json`

### OpenAPI source-of-truth
- `source/openapi-specifications/v1.0/support/*.json`
- `source/openapi-specifications/v1.0/portal/*.json`
- `source/openapi-specifications/v1.0/common/Common.json`

### Scenario/sample generators
- `Created_Generators/*/test_data_generation_configurations.json`
- `info/Ishwarya/*/test_data_generation_configurations.json`
- `info/Vinoth/Generators/*/test_data_generation_configurations.json`

---

## 3) Path Configuration

Current configured paths from `Generator_Patterns/PathConfig.properties`:

```properties
GENERATOR_CONFIG_PATH=../source/api-data-generators/support/
OAS_FILE_PATH=../source/openapi-specifications/v1.0/support/
```

Notes:
- The repository also contains `portal` generators and OAS files; these are actively used in `source/api-data-generators/portal` and `source/openapi-specifications/v1.0/portal`.
- For support-entity generation, default to the configured `support` paths.

---

## 4) Configuration Models Used in This Repo

### 4.1 Full runtime model (`apis` + `generators`)

Used in core inventories such as:
- `source/api-data-generators/support/Ticket/test_data_generation_configurations.json`
- `source/api-data-generators/support/Department/test_data_generation_configurations.json`
- `source/api-data-generators/portal/Ticket/test_data_generation_configurations.json`

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

## 5) Generator Types (Authoritative Set)

From `Generator_Patterns/*`, exactly five types are supported:

1. **dynamic**
   - Fetches real data from API operations.
   - Required: `type`, `generatorOperationId`, `dataPath`
   - Optional: `params`, `name`

2. **remote**
   - Calls framework/system providers (timestamps, enums, schema values).
   - Required: `type`, `generatorMethod`, `inputs`
   - Optional: `name`

3. **static**
   - Fixed constant values.
   - Required: `type`, `value`
   - Optional: `name`

4. **reference**
   - Pulls values from incoming request (`body/query/path/header`).
   - Required: `type`, `ref`
   - Optional: `name`

5. **conditional**
   - Rule-based branch between values/generators.
   - Required: `type`, `conditions`
   - Optional: `else`

---

## 6) Syntax Rules You Must Follow

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

## 7) OAS-Driven Generation Process

The required workflow (from agent rules + current file patterns):

1. Identify target operation in OAS (`operationId`, parameter locations, request schema).
2. Determine dependencies required to satisfy mandatory fields.
3. Add dependency generators first (ordering matters).
4. Use only OAS-defined fields in generator `params`.
5. Map generated outputs either:
   - into scenario chain steps (`Created_Generators/*`), or
   - into `apis` parameter bindings (`source/api-data-generators/*`).

Validation checkpoint:
- Never invent parameters not defined by OAS.
- Match location type correctly (`body`, `query`, `path`, `header`).

---

## 8) Naming and Ordering Conventions

From agent and instruction files:
- Use **snake_case** names.
- Use singular names for single values and plural for arrays/lists.
- Keep names meaningful (`target_ticket`, `source_ticket_one`, `departments`).
- Place dependencies before dependents in each generator array.

Additional practical convention in core files:
- Generator keys may include array markers in runtime mappings (example: `secondary_department_id[*]`).

---

## 9) Practical Patterns Seen in Existing Files

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
Observed in `source/api-data-generators/portal/Ticket/*`:
- use `remote` + `getDynamicEnumValues`
- use `remote` + `getCustomFieldSchema`

### 9.4 API parameter binding matrix
Observed across support/portal runtime configs:
- `apis -> operation -> status -> param -> generator ref`
- supports both success and error response-code scenarios (e.g., `200`, `204`, `422`)

---

## 10) Agent Contract Summary

From `.github/agents/Generators.agent.md`:
- read patterns first, then OAS, then build generators
- respect `PathConfig.properties`
- create new generator files for new work
- output only generator JSON object during agent generation flow
- avoid editing OAS files
- ensure short-hand references resolve in same file unless explicit cross-file path is used

---

## 11) Common Pitfalls to Avoid

1. Missing `$` in dependency reference (must be `$name.value`).
2. Using params not present in OAS operation schema.
3. Wrong JSONPath location (`$.input.path` vs `$.input.query`).
4. Adding unsupported generator types.
5. Dependency steps placed after consumer step.
6. Mixing runtime mapping style and scenario style unintentionally.

---

## 12) Minimum Quality Checklist

Before finalizing any generator file:
- [ ] Correct top-level structure for target use case (`generators` only vs `apis` + `generators`)
- [ ] Valid generator type and required fields
- [ ] OAS operation/params verified
- [ ] Reference paths and JSONPath checked
- [ ] Naming is snake_case and meaningful
- [ ] Dependency ordering is correct

---

## 13) Canonical Example Snippets

### Dynamic dependency chain

```json
{
  "type": "dynamic",
  "name": "ticket",
  "generatorOperationId": "support.Ticket.createTicket",
  "dataPath": "$.response.body:$.id",
  "params": {
    "departmentId": "$departments.value",
    "contactId": "$contact.value"
  }
}
```

### Remote enum value

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

### Reference from path param

```json
{
  "type": "reference",
  "ref": "$.input.path:$.ticketId"
}
```

---

## 14) Final Notes

This documentation is derived from the current repository state across:
- all files in `Generator_Patterns`
- existing generator configs under `source/api-data-generators` and `Created_Generators`
- OAS files in `source/openapi-specifications/v1.0`
- `.github/agents/Generators.agent.md`

If path config defaults change (for example, support vs portal), update `Generator_Patterns/PathConfig.properties` and this document together.
