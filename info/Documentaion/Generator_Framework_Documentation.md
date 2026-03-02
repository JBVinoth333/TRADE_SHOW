# Generator Framework Documentation

**Version:** 3.0.0  
**Last Updated:** 02 March 2026  
**Audience:** Developers who create generator configurations

---

## 1) Generators Object Definition

The generator configuration must contain a top-level `generators` object.

### Structure

```json
{
  "generators": {
    "<generator_key>": [
      {
        "type": "<dynamic|remote|static|reference|conditional>",
        "...": "generator specific fields"
      }
    ]
  }
}
```

### Rules
- `generators` is required.
- Each key inside `generators` is your output field name (example: `ticket_id`).
- Each key value must be an array.
- Each object inside the array must have `type`.

---

## 2) Generator Type Definitions

### 2.1 Dynamic Generator (`dynamic`)
Fetch values from an API response.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `dynamic` |
| `generatorOperationId` | Yes | OpenAPI operation ID |
| `dataPath` | Yes | Response extraction path |
| `params` | No | Query/body filter values |

Example:

```json
{
  "type": "dynamic",
  "generatorOperationId": "support.Department.getDepartments",
  "dataPath": "$.response.body:$.data[*].id"
}
```

### 2.2 Remote Generator (`remote`)
Generate values through provider methods.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `remote` |
| `generatorMethod` | Yes | Full method path |
| `inputs` | Yes | Input object for method |

Example:

```json
{
  "type": "remote",
  "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDateTime",
  "inputs": {
    "timeLine": "current",
    "format": "yyyy-MM-dd'T'HH:mm:ss'Z'"
  }
}
```

### 2.3 Static Generator (`static`)
Return a fixed constant.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `static` |
| `value` | Yes | Constant value |

Example:

```json
{
  "type": "static",
  "value": "High"
}
```

### 2.4 Reference Generator (`reference`)
Reuse value from input request.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `reference` |
| `ref` | Yes | Input extraction path |

Example:

```json
{
  "type": "reference",
  "ref": "$.input.query:$.departmentId"
}
```

### 2.5 Conditional Generator (`conditional`)
Choose value by condition.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Must be `conditional` |
| `conditions` | Yes | List of `when` / `then` rules |
| `else` | No | Fallback value |

Example:

```json
{
  "type": "conditional",
  "conditions": [
    {
      "when": {
        "key": "$.input.query:$.modules",
        "equals": "tickets"
      },
      "then": {
        "value": "TICKET_FLOW"
      }
    }
  ],
  "else": "DEFAULT_FLOW"
}
```

---

## 3) How to Give JSONPath and Reference Paths

### 3.1 `dataPath` format for dynamic generators

Format:

```text
$.response.body:<jsonpath>
```

Examples:
- `$.response.body:$.data[*].id`
- `$.response.body:$.data[0].ticketNumber`
- `$.response.body:$.meta.count`

### 3.2 `ref` format for reference generators

Format:

```text
$.input.<source>:<jsonpath>
```

Allowed `<source>` values:
- `body`
- `query`
- `path`
- `header`

Examples:
- `$.input.body:$.ticket.id`
- `$.input.query:$.departmentId`
- `$.input.path:$.ticketId`
- `$.input.header:$.x_org_id`

### 3.3 Quick Path Tips
- Use exact key names from request/response payload.
- Use `[*]` when you need list values.
- Use `[0]` when you need only first item.
- If output is empty, first check path before checking logic.

---

## 4) Common Example with All Generator Types

```json
{
  "generators": {
    "department_id": [
      {
        "type": "dynamic",
        "generatorOperationId": "support.Department.getDepartments",
        "dataPath": "$.response.body:$.data[*].id"
      }
    ],
    "created_at": [
      {
        "type": "remote",
        "generatorMethod": "applicationDriver.rpc.desk.DynamicDataProvider.getDateTime",
        "inputs": {
          "timeLine": "current",
          "format": "yyyy-MM-dd'T'HH:mm:ss'Z'"
        }
      }
    ],
    "priority": [
      {
        "type": "static",
        "value": "High"
      }
    ],
    "input_department_id": [
      {
        "type": "reference",
        "ref": "$.input.query:$.departmentId"
      }
    ],
    "flow_name": [
      {
        "type": "conditional",
        "conditions": [
          {
            "when": {
              "key": "$.input.query:$.modules",
              "equals": "tickets"
            },
            "then": {
              "value": "TICKET_FLOW"
            }
          }
        ],
        "else": "DEFAULT_FLOW"
      }
    ]
  }
}
```

Expected output shape:

```json
{
  "department_id": ["220728000000123001", "220728000000123002"],
  "created_at": "2026-03-02T12:20:00Z",
  "priority": "High",
  "input_department_id": "220728000000009999",
  "flow_name": "TICKET_FLOW"
}
```

---

## 5) Quick Validation Checklist

- JSON syntax is valid (`jq empty <file>.json`).
- Each generator key contains array.
- `type` value is valid.
- `generatorOperationId` is correct.
- `dataPath` / `ref` paths are correct.
- `generatorMethod` path is correct.

---

## Version History

| Version | Date | Notes |
|---|---|---|
| 3.0.0 | 02 Mar 2026 | Rewritten with explicit generator object definition, per-type definitions, JSONPath/reference rules, and one complete all-types example |
| 2.0.0 | 02 Mar 2026 | Simple developer guide |
