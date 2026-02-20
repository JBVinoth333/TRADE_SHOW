Test Data Generation Configuration Specification


1. Introduction 
1.1 Overview

The Data Generation Framework is designed to generate test data based on the OpenAPI Specification. It extracts input parameter details from OpenAPI definitions and generates test data accordingly.

To control the data generation process, developers must define test data generation configuration files. These configuration files follow a structured format, as detailed in this specification.


2. Data Generation Configuration Structure
2.1 apis Object

The apis object defines the test data generation configuration. These configurations MAY be used to generate test data for API inputs.


2.1.1 Patterned Fields

