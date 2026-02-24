================================================================================
                    DATA GENERATION FRAMEWORK DOCUMENTATION
================================================================================

TABLE OF CONTENTS
1. Introduction
2. Generator Fundamentals
3. Generator Types
   3.1 Static Generator
   3.2 Dynamic Generator
   3.3 Reference Generator
   3.4 Remote Generator
   3.5 Conditional Generator
4.Reference Object
5. Conclusion

================================================================================
1. INTRODUCTION
================================================================================

The Data Generation Framework enables developers to create test data for APIs
defined in OpenAPI Specifications (OAS). It provides a structured JSON-based
approach to generate, reference, and chain data dependencies.

Key Concepts:
- Generators are JSON configurations that produce test data
- Five generator types handle different data scenarios
- Generators can depend on other generators
- All data must come from defined OAS operations or static values

Framework Benefits:
- Reusable test data across multiple API operations
- Consistent data generation reducing manual effort
- Clear dependency management between generators
- Maintains data integrity and type safety

================================================================================
2. GENERATOR FUNDAMENTALS
================================================================================

2.1 BASIC STRUCTURE
=====================

All generators must follow this JSON structure:

{
  "generators": {
    "generator_name": [
      {
        "type": "generator_type",
        // additional properties
      }
    ]
  }
}

================================================================================
3. GENERATOR TYPES
================================================================================

3.1 STATIC GENERATOR
======================

PURPOSE: A Static Generator selects data from predefined values.

REQUIRED PROPERTIES:
  - type: Must be "static"
  - value: A single JSON value (string, number, boolean, array, object, null)

EXAMPLES:

    {
        "generators": {
            "snippet_id": [
                {
                    "type": "static",
                    "value": [
                        "value1",
                        "value2"
                    ]
                }
            ]
        }
    }
    

3.2 DYNAMIC GENERATOR
=======================

PURPOSE: A Dynamic Generator generates data by invoking other APIs.

REQUIRED PROPERTIES:
  - type: Must be "dynamic"
  - generatorOperationId: Target API operation ID (format: service.Entity.operation)
  - dataPath: JSONPath to extract value from response

OPTIONAL PROPERTIES:
  - params: Filter parameters (object) - e.g., {"status": "ACTIVE"}
  - name: Unique name for generator (used in references)

EXAMPLES:

    {
         "generators": {
             "snippet_id": [
                {
                    "type": "dynamic",
                    "generatorOperationId": "Snippets.create",
                    "dataPath": "$.response.body:$.snippetId",
                    "name": "snippetId",
                    "params": {
                        "name": "static_value"
                    }
                }
            ]
        }
    }
    

3.3 REFERENCE GENERATOR
==========================

PURPOSE: A Reference Generator extracts data from previous API responses or request parameters.

REQUIRED PROPERTIES:
  - type: Must be "reference"
  - ref: Path to request data or response data (format: $.input.body:$.field or $.response.body:$.field)

EXAMPLES:

    {
        "generators": {
            "agent_name": [
                {
                    "type": "reference",
                    "ref": "$agents.responses.body:$.data[*].name"
                }
            ]
        }
    }

    
3.4 REMOTE GENERATOR
======================

PURPOSE: A Remote Generator calls Java methods to generate data.

REQUIRED PROPERTIES:
  - type: Must be "remote"
  - generatorMethod: System function path

OPTIONAL PROPERTIES:
  - inputs: Object containing input parameters for the method

EXAMPLES:

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
    

3.5 CONDITIONAL GENERATOR
===========================

PURPOSE: A Conditional Generator selects a data generator based on predefined conditions.

REQUIRED PROPERTIES:
  - type: Must be "conditional"
  - conditions: Array of condition objects
  - else: (optional but recommended) default value/generator

CONDITION STRUCTURE:

Each condition object contains:
  - when: Condition to evaluate (key, operator, value)
    - key: Path to value being tested (e.g., $.input.query:modules)
    - equals: Value to match (string comparison)
  - then: Generator or value if condition matches
    - use: Reference to another generator

EXAMPLES:

    {
        "generators": {
            "snippet_id": [
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
                        },
                        {
                            "when": {
                                "key": "$.input.query:modules",
                                "equals": "contacts"
                            },
                            "then": {
                                "use": "$generators:#/generators/contact_id"
                            }
                        }
                    ],
                    "else": "$generators:#/generators/activity_id"
                }
            ]
        }
    }

================================================================================
4. REFERENCE OBJECT
================================================================================

PURPOSE: A Reference Object provides a way to reuse data across different API calls.

4.1 JSON Path Reference
=======================

PURPOSE: A JSON Path Reference extracts data from API inputs or responses using JSON Path syntax.The reference string consists of four parts:

    1. Source: This refers to which stored variable name we need to get the data from. If we need to generate data from the current APIs, we simply mention $.
    2. IO: This part refers to the data to retrieve from the input or response.
    3. Location: This refers to the location of the data (e.g., body, query, headers).
    4. JSON Path: This query is used to filter the data based on the query.

EXAMPLE:

    "ref" : "$agents.responses.body:$.data[*].name"

4.2 GENERATOR GROUP REFERENCE
===============================

PURPOSE: A Generator Group Reference is used to refer to an already defined data generation group, allowing us to avoid duplicating data generation logic. It can refer to the current module's or another module's data generator group.

EXAMPLE:
    
    "ref" : "$generators:#/generators/agent_id"
    
5. Conclusion
================

This specification outlines the configuration structure for the Data Generation Framework. By following this format, developers can efficiently generate and control test data using OpenAPI specifications.

This structured documentation ensures consistency, scalability, and reusability in test data generation.