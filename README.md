# Salesforce Concise Documentation in Markdown

A collection of concise, LLM-friendly documentation for Salesforce development, focusing on Apex, Lightning Web Components (LWC), and Report Metadata. Inspired by official Salesforce documentation but reorganized for clarity and quick reference.

## Overview

This repository contains markdown documentation covering:

- **Apex Core Language** (`/apex/`)

  - Language fundamentals (data types, SOQL, DML)
  - Asynchronous Apex (Future, Batch)
  - Triggers and best practices
  - Code samples and patterns

- **Lightning Web Components** (`/lwc/`)

  - Core concepts and lifecycle
  - JavaScript patterns and decorators
  - Events and communication
  - Data services and UI patterns
  - Base components guide
  - CSS and styling

- **Report Metadata** (`/report/`)
  - Core metadata structure
  - Data shaping and filtering
  - Grouping and summarization
  - Charts and advanced features
  - Full examples

## Use Cases

1. **LLM Integration**

   - Feed these docs to LLMs for more accurate code generation
   - Use as context for LLM prompts
   - Train custom models on Salesforce development

2. **Quick Reference**
   - Concise explanations of key concepts
   - Common patterns and best practices
   - Metadata structure examples

## Structure

```
salesforce-concise-markdown-docs/
├── apex/
│   ├── topics/
│   │   ├── data_types.md
│   │   ├── soql_basics.md
│   │   ├── dml_operations.md
│   │   └── ...
│   └── manifest.json
├── lwc/
│   ├── topics/
│   │   ├── lwc_overview.md
│   │   ├── lwc_lifecycle_hooks.md
│   │   └── ...
│   └── manifest.json
└── report/
    ├── topics/
    │   ├── report_metadata_core.md
    │   ├── report_data_shaping.md
    │   └── ...
    └── manifest.json
```

## Features

- **Markdown Format**: Easy to read, edit, and process
- **Code Examples**: Practical implementation samples
- **Metadata Examples**: Configuration patterns
- **Topic Organization**: Logical grouping of related concepts
- **Cross-References**: Links between related topics
- **Manifest Files**: Topic categorization and metadata

## Contributing

1. Fork this repository
2. Add or improve documentation
3. Maintain the established format:
   - Clear topic organization
   - Concise explanations
   - Practical code examples
   - Proper metadata structure

## License

Public Domain - This work is dedicated to the public domain. You can copy, modify, distribute and perform the work, even for commercial purposes, all without asking permission.

## Acknowledgments

Based on official Salesforce documentation and community best practices, reorganized for accessibility and LLM compatibility.
