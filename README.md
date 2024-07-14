

# Hud's Elastic Suite for Onboarding Data Sources

## Overview
Hud's Elastic Suite is an integrated set of tools designed to streamline the process of onboarding data sources into Elasticsearch. This suite empowers users to create and manage GROK parsers, generate Elasticsearch ingest pipelines, and map field types efficiently. The following documentation provides a comprehensive guide on utilizing each component within the suite.

## Diagram
![Hud's Elastic Suite for Onboarding Data Sources](https://github.com/huddaannaa/HudsElasticSuite/blob/main/hudsElasticSuite_.jpg)


## Table of Contents
1. [Introduction](#introduction)
2. [Creating the GROK Parser](#creating-the-grok-parser)
3. [Step 1: Extract Fields from GROK](#step-1-extract-fields-from-grok)
4. [Step 2: Build Elasticsearch Pipelines](#step-2-build-elasticsearch-pipelines)
5. [Step 3: Read Fields and Map Field Types](#step-3-read-fields-and-map-field-types)
6. [Elastic Ingest Sample Log Generator](#elastic-ingest-sample-log-generator)
7. [Manual Elasticsearch Loader](#manual-elasticsearch-loader)
8. [Ingest Pipelines and Index Templates in Elasticsearch](#ingest-pipelines-and-index-templates-in-elasticsearch)
9. [Parameters](#parameters)
10. [Example Usage](#example-usage)

## Introduction
Hud's Elastic Suite simplifies the data onboarding process for Elasticsearch by providing a structured approach to log parsing, field extraction, pipeline generation, and field mapping. This suite ensures that data is ingested accurately and efficiently, minimizing the time and effort required to set up and maintain Elasticsearch data sources.

These set of tools are designed to make the life of analysts much easier when onboarding data sources into Elasticsearch. The two main components are:
- Index templates
- Ingest pipelines (using the ingest node)

**Note:** Pipelines can also be created using Logstash, but this documentation focuses on the ingest node.

## Benefits of the Tools

### Summary

Hud's Elastic Suite for Onboarding Data Sources provides a comprehensive set of tools designed to streamline and automate the process of integrating various log data sources into Elasticsearch. The suite covers all essential steps from GROK parsing, field extraction, pipeline generation, to field type mapping, ensuring data is accurately and efficiently ingested. 

For Security Operations Centers (SOC), this means enhanced threat detection, improved incident response, and consistent, accurate log analysis. For data engineers, the suite offers automated pipeline creation, efficient field mapping, scalability, and customization, reducing the time and effort required for setting up and maintaining Elasticsearch data sources.

By utilizing Hud's Elastic Suite, organizations can significantly enhance their data onboarding process, ensuring that log data is processed and indexed effectively, thereby supporting better data analysis and security operations.

### For Security Operations Center (SOC)
1. **Streamlined Log Ingestion:** The suite automates the ingestion of logs from various sources, ensuring that all logs are parsed and indexed correctly for analysis.
2. **Enhanced Threat Detection:** By accurately mapping and processing log data, the suite facilitates better detection of security threats and anomalies.
3. **Improved Incident Response:** SOC analysts can quickly onboard new log sources and generate actionable insights, speeding up the incident response process.
4. **Consistency and Accuracy:** The tools ensure consistent field extraction and data processing, reducing the chances of errors in log analysis.

### For Data Engineers
1. **Automated Pipeline Creation:** The suite automates the creation of Elasticsearch ingest pipelines, saving time and effort in setting up data ingestion workflows.
2. **Efficient Field Mapping:** By automating the field mapping process, data engineers can ensure that all data is correctly typed and indexed, optimizing search and analysis performance.
3. **Scalability:** The tools support the onboarding of multiple data sources, making it easier to scale data ingestion as the organization grows.
4. **Customization:** Engineers can customize GROK patterns and pipeline configurations to meet specific data ingestion requirements, providing flexibility and control over the data onboarding process.


**Note:** Pipelines can also be created using Logstash, but this documentation focuses on the ingest node.

## Creating the GROK Parser
Create the GROK parser using the tool available at [Grok Debugger](https://grokdebug.herokuapp.com/). This parser is essential for extracting custom-named fields from log data, which serves as the foundation for subsequent data processing steps.

### Importance
The GROK parser translates raw log data into structured data, allowing for easier manipulation and analysis within Elasticsearch. It acts as the first step in the data pipeline, ensuring that logs are parsed correctly before further processing.

### Steps to Create the GROK Parser
1. **Develop GROK from Grok Debugger Website or Kibana Dev Tools:**
   - Visit [Grok Debugger](https://grokdebug.herokuapp.com/).
   - Create your GROK pattern to parse the log data.
   - Example:
     ```
     %{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:log_level} %{GREEDYDATA:message}
     ```
   - Save the GROK pattern for use in Step 1.

## Step 1: Extract Fields from GROK
**Tool:** `extract_fields_from_grok.py`

**Usage:**
```sh
extract_fields_from_grok.py [-h] -G grok_pattern_file -m mapping_file [-s output_file]
```

- **Purpose:** Extracts custom-named fields from the GROK parser and generates a default mapping.
- **Input:** GROK parser configuration.
- **Output:** Extracted fields.

### Importance
Extracting fields from the GROK parser ensures that the relevant data is identified and made available for further processing. This step is crucial as it lays the groundwork for building the ingest pipeline by providing the necessary field mappings.

### Steps to Extract Fields
1. Save the GROK pattern in a file (e.g., `grok_pattern.txt`).
2. Run `extract_fields_from_grok.py` with the saved GROK pattern.
   ```sh
   extract_fields_from_grok.py -G grok_pattern.txt -m mapping_file.json -s output_file.json
   ```
3. Review the extracted fields in `output_file.json`.

## Step 2: Build Elasticsearch Pipelines
**Tool:** `generate_pipes.py`

**Usage:**
```sh
generate_pipes.py [-h] [-s parameters_file] [-x pipeline_config_file]
```

- **Purpose:** Generates the ingest pipeline for Elasticsearch. It can also generate the fields used in the pipeline, which is useful for creating templates.
- **Input:** Extracted fields from Step 1, parameters.
- **Output:** Elasticsearch ingest pipeline configuration.

### Importance
Building Elasticsearch pipelines allows for the transformation and enrichment of data as it is ingested. This step is essential for defining how the data should be processed, indexed, and stored within Elasticsearch, ensuring that it is searchable and analyzable.

### Steps to Build Pipelines
1. Run `generate_pipes.py` to generate a parameter file (`parameter.py`).
   ```sh
   generate_pipes.py -s parameter.py
   ```
2. Edit `parameter.py` to map the original fields to ECS fields and configure pipeline settings:
   ```python
   # parameter.py
   fds = """
   # specify event fields here
   """
   event_dataset = "my_dataset"
   event_type = "connection"
   event_category = "network"
   event_kind = "event"
   datetime_fields = "timestamp"
   remove_fields = ["host", "agent", "elastic_agent"]

   pipeline_name = f"logs-{event_dataset}"
   grok_field = "message"

   grok_list = [
       # List of GROK patterns
   ]
   ```
3. Generate the pipeline configuration file.
   ```sh
   generate_pipes.py -x pipeline_config.json
   ```

## Step 3: Read Fields and Map Field Types
**Tool:** `read_fields_and_map2field_types.py`

- **Purpose:** Reads the fields from the pipeline and assigns default field types for template creation.
- **Input:** Extracted fields from Step 2.
- **Output:** Field types mapped for use in the Elasticsearch template.

### Importance
Mapping field types is critical for data integrity and query performance. This step ensures that each field is assigned the correct type (e.g., string, integer, date), which optimizes storage and retrieval in Elasticsearch.

### Steps to Read and Map Field Types
1. Use `read_fields_and_map2field_types.py` to read and map fields.
   ```sh
   read_fields_and_map2field_types.py -i pipeline_config.json -o field_mapping.json
   ```

## Elastic Ingest Sample Log Generator
**Tool:** `elastic_ingest_sample_log_generator.py`

- **Purpose:** Used for testing ingest pipelines by generating sample logs in an Elasticsearch format (e.g., `_id`, `message`, `index`).
- **Output:** Sample logs for testing.

### Importance
Generating sample logs allows for thorough testing of the ingest pipeline. This step ensures that the pipeline is functioning as expected and that data is ingested correctly, preventing issues in production.

### Steps to Generate Sample Logs
1. Use `elastic_ingest_sample_log_generator.py` to create sample logs.
   ```sh
   elastic_ingest_sample_log_generator.py -o sample_logs.json
   ```

## Manual Elasticsearch Loader
**Tool:** `manual_es_loader.py`

- **Purpose:** Manually sends sample logs to Elasticsearch, useful for testing pipelines and templates.
- **Output:** Sample logs ingested into Elasticsearch.

### Importance
The manual loader provides a way to test and validate pipeline configurations before automating the data ingestion process. This step is essential for ensuring that the pipeline and template configurations are accurate and effective.

### Steps to Manually Load Logs
1. Use `manual_es_loader.py` to send sample logs to Elasticsearch.
   ```sh
   manual_es_loader.py -i sample_logs.json -e elasticsearch_url
   ```

## Ingest Pipelines and Index Templates in Elasticsearch
- **Purpose:** The ingest pipelines and index templates are loaded into Elasticsearch to process and store the incoming log data.
- **Input:** Ingest pipeline configuration, index template configuration.
- **Output:** Configured ingest pipelines and index templates within Elasticsearch.

### Importance
Configuring ingest pipelines and index templates in Elasticsearch is the final step in the onboarding process. This ensures that data is processed according to the defined rules and stored in a structured manner, ready for analysis.

### Steps to Configure Elasticsearch
1. Load the ingest pipeline configuration into Elasticsearch.
   ```sh
   curl -X PUT "elasticsearch_url/_ingest/pipeline/my_pipeline" -H 'Content-Type: application/json' -d @pipeline_config.json
   ```
2. Load the index template configuration into Elasticsearch.
   ```sh
   curl -X PUT "elasticsearch_url/_template/my_template" -H 'Content-Type: application/json' -d @index_template.json
   ```

## Parameters
- **Purpose:** Stores parameters required for generating the pipeline and templates.
- **Input:** Extracted fields from Step 1 and generated fields from `generate_pipes.py`.
- **Usage:** Used as input to `generate_pipes.py` if the parameters do not exist.

### Importance
Parameters provide the necessary context and configuration for pipeline and template generation. This step ensures that all required settings are available, making the pipeline creation process smooth and consistent.

## Example Usage

### Creating the GROK Parser
1. Visit [Grok Debugger](https://grokdebug.herokuapp.com/).
2. Create your GROK pattern to parse the log data.
3. Save the GROK pattern for use in Step 1.

### Step 1: Extract Fields from GROK
1. Run `extract_fields_from_grok.py` with the saved GROK pattern.
   ```sh
   extract_fields_from_grok.py -G grok_pattern.txt -m mapping_file.json -s output_file.json
   ```
2. Review the extracted

 fields.

### Step 2: Build Elasticsearch Pipelines
1. Provide the extracted fields and necessary parameters to `generate_pipes.py`.
   ```sh
   generate_pipes.py -s parameter.py -x pipeline_config.json
   ```
2. Generate the ingest pipeline configuration.
3. Optionally generate fields for creating templates.

### Step 3: Read Fields and Map Field Types
1. Use `read_fields_and_map2field_types.py` to read the fields from the pipeline.
2. Map the fields to their respective types for the Elasticsearch template.

### Testing with Elastic Ingest Sample Log Generator
1. Use the `elastic_ingest_sample_log_generator.py` to generate sample logs.
2. Test the ingest pipeline by ingesting these sample logs into Elasticsearch.

### Manual Testing with Manual Elasticsearch Loader
1. Use `manual_es_loader.py` to manually send sample logs to Elasticsearch.
2. Validate the correctness of the pipeline and template configurations.

### Configuring Elasticsearch
1. Load the ingest pipeline configuration into Elasticsearch.
   ```sh
   curl -X PUT "elasticsearch_url/_ingest/pipeline/my_pipeline" -H 'Content-Type: application/json' -d @pipeline_config.json
   ```
2. Load the index template configuration into Elasticsearch.
   ```sh
   curl -X PUT "elasticsearch_url/_template/my_template" -H 'Content-Type: application/json' -d @index_template.json
   ```
3. Verify that the configurations are correctly set up.
