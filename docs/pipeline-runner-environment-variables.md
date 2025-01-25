---
layout: page
order: 5
---

# Pipeline Runner - Environment Variables 

- [Pipeline Runner - Environment Variables](#pipeline-runner---environment-variables)
  - [Benefits of Using Environment Variables](#benefits-of-using-environment-variables)
  - [Java Example for Running Pipeline with Environment Variables](#java-example-for-running-pipeline-with-environment-variables)
    - [Windows Example](#windows-example)
    - [macOS/Linux Example](#macoslinux-example)
    - [Full Java Code Example](#full-java-code-example)
  - [Summary](#summary)


Environment variables allow you to dynamically configure and execute the pipeline runner without altering the JSON pipeline configuration. This approach provides flexibility, enabling you to set parameters like input and output file names dynamically.

This guide demonstrates how to integrate environment variables into the pipeline runner execution using Java, with examples for Windows, macOS, and Linux.

## Benefits of Using Environment Variables

- **Flexibility**: Configure parameters dynamically without modifying the core configuration.
- **Cross-Platform Compatibility**: Adapt commands for different operating systems.
- **Reusability**: Use the same pipeline configuration across multiple environments.

## Java Example for Running Pipeline with Environment Variables

Here is a detailed example of executing the pipeline runner from Java, setting up environment variables, and handling platform-specific commands.

The example demonstrates system-dependent methods to process files. The JSON configuration may require different commands for execution based on the operating system.

### Windows Example

The commands are executed using PowerShell. Below is a JSON configuration example for Windows:

```java
jsonContent = """
{
    "actions": [
        {
            "name": "env_check",
            "program": "Get-ChildItem Env:"
        },
        {
            "name": "print_message",
            "program": "Write-Host \"file size of %INPUT_PDF%\""
        },
        {
            "name": "print_file_size",
            "program": "powershell -Command \"(Get-Item '%INPUT_PDF%').Length\"",
            "args": []
        }
    ]
}
""";
```

### macOS/Linux Example

The commands are executed using Bash. Below is a JSON configuration example for macOS and Linux:

```java
jsonContent = """
{
    "actions": [
        {
            "name": "env_check",
            "program": "env"
        },
        {
            "name": "print_message",
            "program": "echo \"file size of ${INPUT_PDF}\""
        },
        {
            "name": "print_file_size",
            "program": "stat -f '%z' '${input_pdf}'",
            "args": [
                {
                    "name": "input_pdf",
                    "value": "$INPUT_PDF"
                }
            ]
        }
    ]
}
""";
```

### Full Java Code Example

```java
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Map;

public class App {
    public static void main(String[] args) {
        String jsonContent;
        File runnerFile = new File("pipeline_runner");
        String command = runnerFile.getAbsolutePath();

        String os = System.getProperty("os.name").toLowerCase();
        if (os.contains("win")) {
            command += ".exe ";
            jsonContent = /* Windows JSON content here */;
        } else {
            jsonContent = /* macOS/Linux JSON content here */;
        }

        File jsonFile = new File("data.json");
        try (FileWriter writer = new FileWriter(jsonFile)) {
            writer.write(jsonContent);
            System.out.println("JSON file created: " + jsonFile.getAbsolutePath());
        } catch (IOException e) {
            System.err.println("Error writing JSON to file: " + e.getMessage());
            return;
        }

        File logFile = new File("log.txt");
        String[] cmd_args = { command, "-p", jsonFile.getAbsolutePath(), "--log_level", "debug", "--log", logFile.getAbsolutePath() };

        ProcessBuilder processBuilder = new ProcessBuilder();
        processBuilder.command(cmd_args);
        Map<String, String> environment = processBuilder.environment();
        environment.put("MY_CUSTOM_VAR", "12345");
        environment.put("ANOTHER_VAR", "HelloWorld");
        environment.put("INPUT_PDF", new File("input.pdf").getAbsolutePath());

        try {
            Process process = processBuilder.start();
            new Thread(() -> {
                try (var reader = new java.io.BufferedReader(new java.io.InputStreamReader(process.getInputStream()))) {
                    reader.lines().forEach(System.out::println);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();

            int exitCode = process.waitFor();
            System.out.println("Process exited with code: " + exitCode);

        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## Summary

This example demonstrates how to use environment variables to configure and run the pipeline runner dynamically. By tailoring the configuration to the operating system, you can ensure smooth and efficient execution across different environments.
