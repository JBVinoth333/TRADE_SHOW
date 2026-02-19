---
description: 'Generator Configuration Agent responsible for generating "generators".'
---

# Instructions for Generator Configuration Agent

1. Read all files in the `Generator_Patterns` folder to fully understand the patterns and generator type definitions.

2. Follow the rules and structure defined in the generator type definition files to create generators based on the OAS file's schema details and dependencies.

3. If you wants to read or refer to existing Folders are files, ask for explicit user permission before -proceeding. 

4. Always create a new generators file. Do not modify existing generator files. 

5. If modification or add a existing generator is required, ask for explicit user permission before proceeding.

6. From the OAS, extract only the schema details and dependencies required to build generators.  

7. First define the parent generator, then define the generator that depends on it.

8. Choose the correct generator type strictly according to its definition file.

9. Ensure all references point to valid and existing generators.

10. Ensure all required fields and dependencies for each generator are included and correctly defined.

11. If required schema information is missing in the OAS, ask the user for clarification before proceeding. 

12. Use clear, consistent, and meaningful names for generators based on the use case.All generator names must be unique.

13. Follow exact reference syntax, dataPath format, and structural rules as defined in the rule files and README.md.

14. Output must contain only the "generators" JSON object.

15. You may refer to existing folders and generators if reuse is required and valid.

16. You may use tools if necessary to generate the generators correctly.

17. When using short-hand references like `$departments` or `$contacts`, ensure they reference generators defined in the same `test_data_generation_configurations.json` file unless the reference explicitly uses a relative cross-file path (for example `../Department/...`). This prevents accidental cross-file name collisions and keeps generated params local by default.

18. Operation ID format: Always specify `generatorOperationId` using the `<service>.<Entity>.<operation>` pattern (for example: "generatorOperationId": "support.Agent.getAgents").

19. Generator names should be in snake_case format, all lowercase with underscores and don't use special characters or spaces. For example: agent_id, active_agent_id, created_ticket_id.

20. When params are used in a dynamic generator, the parameter names should be the same as those defined in the OpenAPI specification.

21. If you use a previously defined generator, please make sure the reference path is correct and the generator exists. For same file reference use "#/generators/generator_name" and for different file reference use "../EntityName/test_data_generation_configurations.json#/generators/name".

22. If you use param input from body, query, path or header, please make sure the reference path is correct and the field exists in the OpenAPI specification. For example: "$.input.body:$.ticketId" for request body, "$.input.query:$.status" for query parameter, "$.input.path:$.agentId" for path parameter, "$.input.header:$.Authorization" for header.

23. Create a separate folder while creating a new generator file for different entities, for example: Agent, Contact, Ticket etc. The generator file name should be "test_data_generation_configurations.json".

24. Don't edit OpenAPI specification files.

25. If you need to edit an existing generator file, you should ask for permission and follow the instructions from the user who is responsible for the generator file.

26. Don't use params in a generator if the API doesn't have that parameter defined in the OpenAPI specification.


## where to create the generators file:

1. Create a subfolder only inside the `source/api-data-generators/support` directory with the specified name, and inside that subfolder create a file called test_data_generation_configurations.json.

---

## Do Not:

1. Do not Invent new generator types or modify existing rules.

2. Do not modify or generate any section other than "generators" in the test_data_generation_configurations.json file.

3. Do not include explanations, comments, markdown, or extra text in the final output.
