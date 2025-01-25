---
layout: page
order: 4
---

# PDFix Pipeline Runner

- [PDFix Pipeline Runner](#pdfix-pipeline-runner)
  - [Key Features](#key-features)
  - [Download](#download)
  - [Pipeline Execution](#pipeline-execution)
    - [Parameter Description](#parameter-description)
  - [Pipeline Configuration](#pipeline-configuration)
    - [Defining Actions](#defining-actions)
  - [Action Definition](#action-definition)
    - [Action Example](#action-example)
    - [Action Parameters](#action-parameters)
    - [Arguments](#arguments)
      - [Example:](#example)
    - [Using Environment Variables](#using-environment-variables)
  - [Examples](#examples)
    - [Pipeline Execution Workflow](#pipeline-execution-workflow)
      - [Notes:](#notes)
      - [Full Pipeline Configuration:](#full-pipeline-configuration)
    - [Pipeline Runner Execution from Java](#pipeline-runner-execution-from-java)


The **PDFix Pipeline Runner** is a command-line tool designed to execute automated workflows, or "pipelines," for processing PDF documents. It leverages modular actions, each defined in a configuration JSON file, to perform various tasks such as OCR, language detection, and compliance checks (e.g., PDF/UA standards). These actions can either run locally or inside Docker containers, making the tool flexible and adaptable across different environments.

Typical use cases include:
- Automating repetitive PDF processing tasks.
- Ensuring compliance with PDF standards (e.g., PDF/UA).
- Integrating PDF workflows into enterprise systems.

The application supports custom pipelines by chaining actions with input/output dependencies, allowing for seamless file transformation and metadata updates. It also includes logging capabilities and version display options for better control and debugging.

## Key Features
- **Flexible Pipelines:** Define and execute workflows using a JSON-based configuration.
- **Modular Actions:** Support for both local CLI tools and Docker-based operations.
- **Dynamic Arguments:** Use macros for referencing outputs between actions.
- **Cross-Platform Support:** Compatible with Windows, macOS, and Linux.
- **Error Handling:** Customizable acceptable return codes for robust error management.
- **Logging:** Configurable directories for storing execution logs.

## Download
Please contact support for the download link.

## Pipeline Execution

Minimal pipeline-runner execution:
```bash
./pipeline-runner -p "path/to/pipeline.json"
```

### Parameter Description
```
-p, --pipeline <pipeline_config>   Path to the pipeline configuration JSON
-w, --workingdir <working-dir>     A directory to store any input files and files generated during the 
                                   execution of a pipeline. A system temporary folder is used 
                                   if no folder is provided. (optional)
--log <log-location>               A directory or file path where execution log files will be written.
--log_level <level>                A level of logged information [none, critical, error, warning, info, debug]
--version                          Display the application version only
```

## Pipeline Configuration

The configuration is a JSON document with an array of actions to be executed.

### Defining Actions

The structure of this JSON is as follows:
```json
{
  "title": "The command name",
  "actions": [
    {
      "name": "action-1",
      "path": "<path to program>",
      "program": "<program CLI>",
      "args": []
    },
    {
      "name": "action-2",
      "path": "<path to program>",
      "program": "<program CLI>",
      "args": []
    }
  ]
}
```

## Action Definition

Each action is identified by a JSON node with necessary instructions for execution. Supported action types are:
- **local**: A command-line application installed on a system.
- **docker**: Embedded in a Docker image with the support of a command-line interface (*Docker must be installed on the system to execute such actions*).

Available actions are listed on [PDFix Actions Marketplace](https://pdfix.net/products/actions-marketplace/).

### Action Example

```json
{
  "name": "action-1",
  "id": "action-1-id",
  "path": "/path/to/application/",
  "program": "${action_path}/my_cli_app -i ${input_pdf} -o ${output_pdf}",
  "platform": [ "windows", "darwin" ],
  "returnCodes": [ 0 ],
  "args": []
}
```

### Action Parameters

| Parameter      | Description                                                                                 |
|----------------|---------------------------------------------------------------------------------------------|
| **name**       | A string identifier of an action. The action name can be referenced from argument values.   |
| **id**         | A unique identifier for the action within the pipeline. Defaults to the action name.       |
| **path**       | Path to the executable of an action. Optional for system-wide commands (e.g., `docker`).    |
| **program**    | Full command for execution, including input/output parameters. Macros are recommended.      |
| **platform**   | Supported platforms (`windows`, `darwin`, `linux`).                                         |
| **returnCodes**| Acceptable return codes. Default is `[ 0 ]`.                                                |
| **stdout**     | Handles application output (e.g., save to `${output_txt}`).                                 |
| **stderr**     | Handles application errors (e.g., save to `${error_txt}`).                                  |
| **args**       | Arguments passed to the program.                                                           |
| **title**      | Optional user-friendly name of an action.                                                  |

### Arguments

`args` is an array of user-defined arguments used for execution, replacing macros in the `program`. Each argument has required properties:

| Property       | Description                                                                                 |
|----------------|---------------------------------------------------------------------------------------------|
| **name**       | Defines the macro name for replacement in the `program` string.                             |
| **value**      | Value of the argument.                                                                      |
| **flags**      | Argument flags (e.g., `0x2` for input files, `0x4` for output files).                       |
| **ext**        | File extension for values representing file names.                                          |
| **type**       | Type of argument value (`string`, `int`, `file_path`, `json`). Default is `string`.         |

#### Example:
```json
{
  "name": "input_pdf",
  "desc": "Path to the PDF document you want to process",
  "flags": 2,
  "ext": "pdf",
  "type": "file_path",
  "value": "/usr/tmp/input.pdf"
}
```

### Using Environment Variables

Environment variables allow dynamic configuration of the pipeline runner without altering the JSON pipeline configuration. This provides flexibility for setting parameters like input and output file names.

[Learn more about configuring Environment Variables](https://github.com/pdfix/pdfix-documentation/blob/main/docs/pipeline-runner-environment-variables.md).

## Examples

### Pipeline Execution Workflow

This pipeline performs the following steps:
1. **OCR Document** using the Docker image `pdfix/ocr-tesseract`.
2. **Detect Document Language** with the Docker image `pdfix/detect-language`.
3. **Autotag PDF** using the locally installed PDFix SDK.
4. **Set PDF/UA Standard** in the document metadata with the locally installed PDFix SDK.

#### Notes:
- The input file is `/usr/tmp/this_is_input.pdf`.
- The output file is `/usr/tmp/this_is_output.pdf`.
- Intermediate steps reference outputs of previous actions using macros (e.g., `${action-id.output_pdf}`).
- Macros `${license_name}` and `${license_key}` are automatically provided by the pipeline-runner when a license is active on the system.

#### Full Pipeline Configuration:
```json
{
  "actions": [
    {
      "args": [
        {
          "name": "input_pdf",
          "value": "/usr/tmp/this_is_input.pdf"
        },
        {
          "name": "output_pdf",
          "value": ""
        },
        {
          "name": "language",
          "value": "eng"
        }
      ],
      "path": "",
      "program": "docker run --platform linux/amd64 -v \"${working_directory}:/data\" --rm pdfix/ocr-tesseract:v0.4.4 --name \"${license_name}\" --key \"${license_key}\" ocr -i \"/data/${input_pdf}\" -o \"/data/${output_pdf}\" --lang \"${language}\"",
      "returnCodes": [ 0 ],
      "id": "ocr_tesseract",
      "name": "ocr_tesseract",
      "title": "OCR Tesseract"
    },
    {
      "args": [
        {
          "name": "input_pdf",
          "value": "${ocr_tesseract.output_pdf}"
        },
        {
          "name": "output_pdf",
          "value": ""
        }
      ],
      "path": "",
      "program": "docker run --platform linux/amd64 -v ${working_directory}:/data -w /data --rm pdfix/lang-detect:v0.4.4 --name \"${license_name}\" --key \"${license_key}\" detect-language -i \"/data/${input_pdf}\" -o \"/data/${output_pdf}\"",
      "returnCodes": [ 0 ],
      "id": "language_detection",
      "name": "language_detection",
      "title": "Language Detection"
    }
  ]
}
```

### Pipeline Runner Execution from Java

[Learn more about using Pipeline Runner in Java applications](https://github.com/pdfix/pdfix-documentation/blob/main/docs/pipeline-runner-environment-variables.md).
