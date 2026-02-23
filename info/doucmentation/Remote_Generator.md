# Remote Generator

## Purpose
Call built-in system functions to generate timestamps, enum values, or schema metadata.

## Use It When
- You need current/past/future timestamps.
- You need system enum values (status, priority, channel).
- You need custom field schema data.

## Required Fields
- `type`: "remote"
- `generatorMethod`
- `inputs`

## Optional Fields
- `name`

## Common Methods
- `applicationDriver.rpc.desk.DynamicDataProvider.getDateTime`
- `applicationDriver.rpc.desk.DynamicDataProvider.getDynamicEnumValues`
- `applicationDriver.rpc.desk.DynamicDataProvider.getCustomFieldSchema`

## Example (Timestamp)
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

## Example (Enum Values)
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

## Checklist
- Method name is correct and fully qualified.
- Inputs match required parameters.
- Timestamp format uses correct syntax and quotes.
