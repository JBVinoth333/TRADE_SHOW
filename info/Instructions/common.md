any doubts ping me.

1. **Check the OAS file** for required fields and operations. Use the operationId and schema from the OAS to guide your generator structure.

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

## 5. How to Create a Generator (Step-by-Step)

### Step 1: Understand Your Parameters
Open your OpenAPI spec and list all parameters with their types and descriptions.

### Step 2: Choose Generator Type for Each Parameter

Examples:
- `departmentId` (integer) → **Dynamic** (fetch from API)
- `status` (string) → **Static** ("OPEN")
- `createdAt` (timestamp) → **Remote** (getDateTime)
- `agentId` (from another entity) → **Dynamic** (fetch from Agent API)


### Step 4: Create Your Generator File


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