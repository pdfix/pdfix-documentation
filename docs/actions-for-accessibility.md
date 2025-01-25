---
layout: page
---

# PDFix Actions for Accessibility

- [PDFix Actions for Accessibility](#pdfix-actions-for-accessibility)
  - [Make Accessible for Non-Tagged Documents](#make-accessible-for-non-tagged-documents)
    - [Executing the Make Accessible Command](#executing-the-make-accessible-command)
    - [Customizing the Make Accessible Command](#customizing-the-make-accessible-command)
    - [Executing with a Custom Configuration](#executing-with-a-custom-configuration)
    - [Example of a Custom Command to Auto-Tag, Set Document Language, and Apply PDF/UA Identifier](#example-of-a-custom-command-to-auto-tag-set-document-language-and-apply-pdfua-identifier)
  - [Fixing Accessibility Issues in Tagged PDF Documents](#fixing-accessibility-issues-in-tagged-pdf-documents)
    - [Example: Fixing Missing Document Title, Annotation Contents, Lists, and Invalid MCIDs](#example-fixing-missing-document-title-annotation-contents-lists-and-invalid-mcids)
  - [Code Examples](#code-examples)


PDFix Actions for accessibility provide a flexible, no-code method for fixing accessibility issues using batch commands.
For more information about batch commands, please follow [actions]([actions.md](https://pdfix.net/pdfix-batch-commands/)).

Users can assemble a set of actions in a JSON file. These actions are executed sequentially on an opened PDF document.

## Make Accessible for Non-Tagged Documents

The **Make Accessible** command is a set of actions designed to fix common accessibility issues in non-tagged PDF documents.
By default, it uses a configuration that includes a sequence of actions such as cleaning up the previous structure, flattening form XObjects, embedding fonts, autotagging, updating metadata, creating bookmarks from headings, and applying other necessary fixes to ensure PDF/UA compliance.

### Executing the Make Accessible Command

**Using the Command-Line:**
```bash
./pdfix_app make-accessible -i "input.pdf" -o "output.pdf"
```

**Programmatically in Python:**
```python
pdfix = GetPdfix()
doc = pdfix.OpenDoc("path/to/doc.pdf", "")
cmd = doc.GetCommand()
cmdStm = pdfix.CreateMemStream()
command.SaveCommandsToStream(kActionMakeAccessible, cmdStm, kDataFormatJson, kSaveFull)
cmd.LoadParamsFromStream(cmdStm, kDataFormatJson)
cmdStm.Destroy()
command.Run()
doc.Save("path/to/out.pdf", kSaveFull)
```
**See the complete [code examples](#code-examples) below.**

### Customizing the Make Accessible Command

Some documents require a customized set of actions. For example, if the document:
- Contains transparent elements → flattening form XObjects may visually alter the document.
- Is a PDF 2.0 file → updating metadata must be tailored for this version.

Users can create a custom JSON file with appropriate actions and parameters. A list of all available actions is provided here [actions](https://pdfix.net/pdfix-batch-commands/).

### Executing with a Custom Configuration

**Using the Command-Line:**
```bash
./pdfix_app make-accessible -i "input.pdf" -o "output.pdf" -c "path/to/custom/command.json"
```

**Programmatically in Python:**
```python
pdfix = GetPdfix()
doc = pdfix.OpenDoc("path/to/doc.pdf", "")
cmd = doc.GetCommand()
cmdStm = pdfix.CreateFileStream("path/to/custom/command.json", kReadOnly)
cmd.LoadParamsFromStream(cmdStm, kDataFormatJson)
cmdStm.Destroy()
command.Run()
doc.Save("path/to/out.pdf", kSaveFull)
```
**See the complete [code examples](#code-examples) below.**

### Example of a Custom Command to Auto-Tag, Set Document Language, and Apply PDF/UA Identifier
```json
{
  "actions": [
    {
      "name": "add-tags"
    },
    {
      "name": "set_lang",
      "params": [
        {
          "name": "lang",
          "value": "en-US"
        }
      ]      
    },
    {
      "name": "set_pdf_ua_standard",
      "params": [
        {
          "name": "path",
          "value": "1"
        }
      ]
    }
  ]
}
```

## Fixing Accessibility Issues in Tagged PDF Documents

Tagged PDFs may require a different set of commands, addressing accessibility issues identified in a validation report.

These actions may include:
- Fixing headings
- Adding missing spaces
- Generating alt text or table summaries
- Fixing lists
- Deleting unnecessary tags
- Other necessary corrections

The approach remains the same as with the **Make Accessible** command, excluding methods that clear or re-tag the document structure.

### Example: Fixing Missing Document Title, Annotation Contents, Lists, and Invalid MCIDs
```json
{
  "actions": [
    {
      "name": "set_title",
      "params": [
        {
          "name": "title_type",
          "value": "2"              # Retrieve title from the file name
        }
      ]      
    },
    {
      "name": "set_annot_contents",
      "params": [
        {
          "name": "annot_types",
          "value": "Link|Widget"
        },
        {
          "name": "alt_type",
          "value": "1"              # Use text from annotation bounding box
        }
      ]
    },
    {
      "name": "fix_list_tag"
    },
    {
      "name": "artifact_content",
      "params": [
        {
          "name": "object_types",
          "value": {
            "template": {
              "object_update": [
                {
                  "query": {
                    "$and": [
                      {
                        "$0_artifact": "false"
                      },
                      {
                        "$0_mcid": "-1"
                      }
                    ],
                    "param": [
                      "pds_object"
                    ]
                  },
                  "statement": "$if"
                }
              ]
            }
          }      
        },
        {
          "name": "artifact_type",
          "value": "0"              # Mark as an artifact
        }
      ]
    },
    {
      "name": "remove_content_marks",
      "params": [
        {
          "name": "object_types",
          "value": ".*"             # All tag types
        },
        {
          "name": "flags",
          "value": "0"              # Invalid MCIDs
        }
      ]
    }
  ]
}
```

For more available actions and parameter options, check [actions](https://pdfix.net/pdfix-batch-commands/).

## Code Examples

Links to full code examples:
- [Python](https://github.com/pdfix/pdfix_sdk_example_python/blob/master/src/MakeAccessible.py)
- [C++](https://github.com/pdfix/pdfix_sdk_example_cpp/blob/master/src/MakeAccessible.cpp)
- [.NET](https://github.com/pdfix/pdfix_sdk_example_dotnet/blob/master/src/MakeAccessible.cs)
- [Java](https://github.com/pdfix/pdfix_sdk_example_java/blob/master/src/main/java/net/pdfix/samples/MakeAccessible.java)

