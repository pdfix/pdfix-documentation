---
layout: page
---

# Template Documentation

- [Template Documentation](#template-documentation)
  - [Key Features of Templates](#key-features-of-templates)
  - [Rules and Conditions](#rules-and-conditions)
    - [Object Identification](#object-identification)
    - [Applying Properties and Flags](#applying-properties-and-flags)
  - [Creating Objects in Advance](#creating-objects-in-advance)
  - [Workflow](#workflow)
  - [Creating a Template](#creating-a-template)
  - [Example JSON Configuration](#example-json-configuration)
  - [Links and Resources](#links-and-resources)

The template is a configuration file (JSON) that contains rules, flags, and thresholds to modify the recognition process.

## Key Features of Templates

With a template, you can:

- Define headings.
- Identify headers/footers.
- Modify table or list recognition.
- Define special list labels.
- Pre-define known elements on a specific position.
- And more.

## Rules and Conditions

### Object Identification

Templates use rules with conditions to identify objects based on their properties, such as:

- Position.
- Page number.
- Color.
- Font and font size.
- Content.
- And more.

For example:

- Define a font size to match with a specific heading level.
- Identify path objects by color to classify them as artifacts.
- Specify an image by position and page to define its alternate text.
- Define regular expressions to identify specific text to handle.
- And more.

### Applying Properties and Flags

Each rule may contain properties or set flags to be applied to an object in order to modify the recognition and tagging algorithm. For example:

- Set an object as an artifact.
- Isolate an object to prevent joining with other content.
- Define a custom label type.
- Prevent the creation of a certain structure (e.g., not a table).

## Creating Objects in Advance

Templates can define objects known before running the recognition. For example, if a document contains an address section in the top-left corner of the first page, the user can predefine a text object (P tag) by a bounding box.

This is a useful feature when using another layout recognition tool that generates partial results or when the application provides methods to define these predefined objects.

The information provided to the template can be for example **Page number**, **Bounding box**, **Object type** (image, table, text, list, etc.).

## Workflow

The **Configuration Process** of the templates are usually configured on a small set of documents. Once the configuration is finalized, it can be applied to a large number of similar documents.

Typical **Use Cases** are **Bank statements**, **Invoices**, Any **document type created from the same template** but containing different data.

## Creating a Template

To create a template, use **PDFix Desktop**. This application provides an interactive interface to design templates and view instant results.

## Example JSON Configuration

Following example defines a template rule for defining the H1 heading by font size, name and color.

```json
{
  "template": {
      "statement": "$if",
      "heading": "h1",
      "query": {
          "$and": [
              {
                  "$0_font_size": [
                      {
                          "$gte": "36"
                      }
                  ]
              },
              {
                  "$0_font_name": {
                      "$regex": "TimesNewRomanPS-BoldMT"
                  }
              },
              {
                  "$0_fill_color": [
                      "0",
                      "0",
                      "0"
                  ]
              }
          ],
          "param": [
              "pde_text"
          ]
    },
  }
}
```

## Links and Resources

- [PDF Template documentation](template-documentation.md)
- [PDF Template Examples](template-examples.md)