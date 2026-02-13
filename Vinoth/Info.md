

Dynamic Generator:
==================

A Dynamic Generator generates data by invoking other APIs.

Fixed Fields:
------------

0)Field Name , Type , Description

1)generatorOperationId , String , The OpenAPI operation ID used for test data generation

2)dataPath , JSON Path Reference , Specifies how to extract generated data from the API response

3)name , String , A unique identifier to reference the generator.

4)params , Map[String , Generator Group Reference | JSON Path Reference | String] , Input parameters for API calls.

Example)

     {

         "generators": {

             "snippet_id": [

                 {

                     "type": "dynamic",

                     "generatorOperationId": "Snippets.create",

                     "dataPath": "$.response.body:$.snippetId",

                     "params": {

                         "name": "static_value"

                     }

                 }

             ]

         }

     }

==========================================================================

Remote Generator:
=================

A Remote Generator calls Java methods to generate data.

Fixed Fields:
-------------

1) Field Name , Type , Description

2) generatorMethod , String , The Java method reference for generating data.

3)params , Map[String,Generator Group Reference | JSON Path Reference | String] , Input parameters for the method.

Example) 

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
 ==========================================================================

Conditional Generator:
======================

A Conditional Generator selects a data generator based on predefined conditions.

Fixed Fields:
--------------

0)Field Name , Type , Description

1)conditions , List[Condition Object] , A list of conditions mapping inputs to generators.

2)elseCondition , Generator Group Reference | JSON Path Reference | String , Fallback generator if no conditions match

Example)
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
==========================================================================

Static Generator:
=================

A Static Generator selects data from predefined values.

Fixed Fields:
------------

1) value , List[Any] , A list of predefined values.

Example)

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
===============================================================

 Reference Generator:
=====================

A Reference Generator extracts data from previous API responses.

Fixed Fields:
-------------

0) Field Nam , Type , Description

1) ref , JSON Path Reference , Reference to extract data from previous responses.

Example)

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
===========================================================================
===========================================================================
Reference Object:
=================

A Reference Object provides a way to reuse data across different API calls.

JSON Path Reference;
--------------------

1)A JSON Path Reference extracts data from API inputs or responses using JSON Path syntax.The reference string consists of four parts:

2)Source: This refers to which stored variable name we need to get the data from. If we need to generate data from the current APIs, we simply mention $.

3)IO: This part refers to the data to retrieve from the input or response.

4)Location: This refers to the location of the data (e.g., body, query, headers).

5)JSON Path: This query is used to filter the data based on the query.

Example)

     "ref" : "$agents.responses.body:$.data[*].name"   

=============================================================================

 Generator Group Reference:
--------------------------

A Generator Group Reference is used to refer to an already defined data generation group, allowing us to avoid duplicating data generation logic. It can refer to the current module's or another module's data generator group.



Example)

     "ref" : "$generators:#/generators/agent_id"   

============================================================================
Conclusion:
----------
This structured documentation ensures consistency, scalability, and reusability in test data generation.




































