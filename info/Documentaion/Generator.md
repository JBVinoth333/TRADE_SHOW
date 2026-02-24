1. Dynamic Generator -Example

```json
     {
         "generators": {
             "snippet_id": [
                 {
                     "type": "dynamic",
                     "generatorOperationId": "Snippets.create",
                     "name": "snippetId",
                     "dataPath": "$.response.body:$.snippetId",
                     "params": {
                         "key": "value"
                     }
                 }
             ]
         }
     }
```

2. Remote Generator
   parems -> inputs

Example:

```json
     {
         "generators": {
             "snippet_id": [
                 {
                     "type": "remote",
                     "generatorMethod": "org.example.TestData.generateSnippetId",
                     "inputs": {
                         "key": "value"
                     }
                 }
             ]
         }
     }
```
3. Reference Generator
   
   A Reference Generator extracts data from previous API responses and requests. 