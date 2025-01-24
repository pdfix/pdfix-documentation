---
layout: page
---

# PDFix Actions for Accessibility

PDFix Actions for Accessibility provide a flexible no-code method for fixing accessibility issues using batch commands.
For more information about batch commands please follow [here](#).

The user can assemble a set of actions in a JSON file. These actions are executed in sequence on an opened PDF document.

## Make Accessible for non-tagged documents

The Make Accessible command is a set of actions tailored for fixing the most common accessibility issues in non-tagged PDF documents.
By default it uses a configuration containing a sequence of actions that include cleanup of previous structure, flatten form XObjects, embed fonts, autotag, update metadata, create bookmarks from headings, and other fixes required to make the PDF as compliant with PDF/UA as possible.

Executing the make-accessible command:

Using the Command-Line:
```bash
./pdfix_app make-accessible -i "input.pdf" -o "output.pdf"
```

or programatically in Python:
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

**See the complete [code examples](#code-examples) below.***

### Customizing the Make Accessible command

Some documents require a customized set of actions. For example if the document:
- contains transparent elements -> flattening form XObject would destroy the document visually
- is PDF 2.0 and updating the medatada must be tailored for this PDF version

The user can assemble a custom JSON with appropriate actions and parameters. The list of all available actions is available on this link [pdf-actions.md](pdf-actions.md)


The Execution with custom configuration:

Using the Command-Line:
```bash
./pdfix_app make-accessible -i "input.pdf" -o "output.pdf" -c "path/to/custom/command.json"
```

or programatically in Python:
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

**See the complete [code examples](#code-examples) below.***


An example of custom command to auto-tag, set document landuage, and the PDF/UA identifier:

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

## Fix Accessibility Issues in tagged PDF documents

Tagged PDFs may require another set of commands consisting of actions that address accessibility issues from a PDF validation report.

These could include actions to fix headings, add missing spaces, generate alt texts or table summaries, fix lists, delete tags, and more.

The approach is the same as the customized Make Accessible command, excluding methods to clear or re-tag the document's tag structure.

Following is an example of a custom command addressing missing document title, annotation contents, fix lists, artifact untagged content, and remove content marks with an invalid MCID.

```json
{
  "actions": [
    {
      "name": "set_title",
      "params": [
        {
          "name": "title_type",
          "value": "2"              # Retrieve the title from the file name
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
          "value": "1"              # Use the text from annotation bounding box
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
          "value": {             # non-tagged content
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
          "value": "0"              # mark as an artifact
        }
      ]
    },
    {
      "name": "remove_content_marks",
      "params": [
        {
          "name": "object_types",
          "value": ".*"             # all tag types
        },
        {
          "name": "flags",
          "value": "0"              # invalud MCIS
        }
      ]
    }
  ]
}
```

For more actions and parameter options check [pdf-actions.md](pdf-actions.md)

## Code examples

Links to the full code examples: [Python](https://github.com/pdfix/pdfix_sdk_example_python/blob/master/src/MakeAccessible.py), [c++](https://github.com/pdfix/pdfix_sdk_example_cpp/blob/master/src/MakeAccessible.cpp), [.NET](https://github.com/pdfix/pdfix_sdk_example_dotnet/blob/master/src/MakeAccessible.cs), [Java](https://github.com/pdfix/pdfix_sdk_example_java/blob/master/src/main/java/net/pdfix/samples/MakeAccessible.java)