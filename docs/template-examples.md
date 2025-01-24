---
layout: page
---

# Template Configuration Examples

- [Template Configuration Examples](#template-configuration-examples)
  - [Thresholds and Triggers](#thresholds-and-triggers)
    - [Create Tables Only with Table Borders](#create-tables-only-with-table-borders)
    - [Tag Area with Forms (Widgets) in the Table Layout](#tag-area-with-forms-widgets-in-the-table-layout)
    - [Set Reading Order Detection Algorithm](#set-reading-order-detection-algorithm)
  - [Regular Expressions and Text Functions](#regular-expressions-and-text-functions)
    - [Define a Custom Bullet](#define-a-custom-bullet)
    - [Set Heading by Font Name and Size](#set-heading-by-font-name-and-size)
  - [Page Object Functions](#page-object-functions)
    - [Artifact Any Path Object with White Color](#artifact-any-path-object-with-white-color)
  - [Object Operators](#object-operators)
    - [Define Header by Position and Type](#define-header-by-position-and-type)

This page describes use cases and code examples for creating a template configuration.

For more information about the template configuration, click [here](template.md) to access detailed documentation.

## Thresholds and Triggers

The complete set of thresholds and triggers is documented on [GitHub](https://github.com/pdfix/pdfix_sdk_builds/blob/main/pdf_template.md#threshold-values).

Here is a set of the most common triggers:

### Create Tables Only with Table Borders

`on/off`

```json
{
  "template": {
    "pagemap": [
      {
        "table_detect_sect": 0
      }
    ]
  }
}
```

### Tag Area with Forms (Widgets) in the Table Layout

`on/off`

```json
{
  "template": {
    "pagemap": [
      {
        "table_detect_sect": 0
      }
    ]
  }
}
```

### Set Reading Order Detection Algorithm

`0 - built-in`
`1 - content-based`
`2 - x-y sort`

```json
{
  "template": {
    "pagemap": [
      {
        "rd_sort": 2
      }
    ]
  }
}
```

## Regular Expressions and Text Functions

Regular expressions and text functions are used to identify and mark specific text content, assigning properties and flags as needed.

### Define a Custom Bullet

Define a custom bullet of the Unicode "✸" (\u2738).

```json
{
  "template": {
    "pagemap_regex": [
      {
        "regex_bullet": "^[\\u2022\\u...\\u2738]$"
      }
    ]
  }
}
```

### Set Heading by Font Name and Size

Define conditions for text on page *1* with a minimum font size of *36* and the font name matching the regular expression *TimesNewRomanPS-BoldMT*.

```json
{
  "template": {
    "text_update": [
      {
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
              "$_page_num": {
                "$eq": 1
              }
            }
          ],
          "param": [
            "pde_text"
          ]
        }
      }
    ]
  }
}
```

## Page Object Functions

Page Object Functions are executed on low-level PDF page objects, such as paths, images, and text chunks, prior to any layout recognition.

### Artifact Any Path Object with White Color

```json
{
  "template": {
    "path_object_process": [
      {
        "query": {
          "$and": [
            {
              "$0_fill_color": [
                "255",
                "255",
                "255"
              ]
            }
          ],
          "param": [
            "pds_path"
          ]
        },
        "flag": "artifact"
      }
    ]
  }
}
```

## Object Operators

### Define Header by Position and Type

- Any object with the bottom coordinate larger than 698.32 will be marked as an artifact (header).

```json
{
  "template": {
    "element_update": [
      {
        "flag": "header",
        "query": {
          "$and": [
            {
              "$0_bottom": {
                "$gte": "698.32"
              }
            }
          ],
          "param": [
            "pde_element"
          ]
        },
        "statement": "$if"
      }
    ]
  }
}
```
