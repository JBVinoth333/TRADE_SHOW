# REMOTE Generator

A Remote Generator calls Java methods (or other external code) to produce test data.

Fixed Fields
-------------

1) Field Name , Type , Description

2) `generatorMethod` , String , The fully-qualified Java method reference used to generate data (for example: `org.example.TestData.generateSnippetId`).

3) `params` , Map[String, Generator Group Reference | JSON Path Reference | String] , Input parameters passed to the method. Values can be static strings, references to other generators (`$generators:#/generators/...`) or JSON path references (`$.input.body:$.orgId`).

Example
-------

```json
{
  "generators": {
    "snippet_id": [
      {
        "type": "remote",
        "generatorMethod": "org.example.TestData.generateSnippetId",
        "params": {
          "name": "static_value"
        }
      }
    ]
  }
}
```

Inputs (parameter types)
------------------------

- Static string: `"value"`
- Generator group reference: `"$generators:#/generators/agent_id"`
- JSON path reference: `"$.input.body:$.orgId"` or `"$.response.body:$.id"`

Optional fields
---------------

- `name`: friendly identifier for the generator
- `timeout`: optional timeout for remote invocation (ms)

Best Practices
--------------

- Keep `generatorMethod` fully qualified and stable.
- Prefer generator-group or JSON-path references for inputs to ensure reproducibility.
- Validate method signatures and parameter types in the runtime that executes `remote` generators.
- Document expected return type (string, number, list, id) for consumers.

Troubleshooting
---------------

- If the system fails to invoke the method, verify the classpath and method name.
- If inputs are malformed, confirm JSON-path and generator references resolve correctly.
- For intermittent failures, add a `timeout` and retry logic where supported.

---

*Last Updated: 14 February 2026*
