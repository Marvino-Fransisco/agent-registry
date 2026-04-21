---
name: pe-template-format
description: >
  Design reusable prompt templates with variables, placeholders, and slot syntax
  for dynamic content injection. Use this skill whenever the user wants to
  templatise a prompt, parameterise it with variables, create a reusable prompt
  scaffold, or asks about Jinja2 / f-string / handlebars / mustache style prompt
  templates. Also use when the user has a working prompt they want to turn into
  a reusable template for a pipeline or product.
---

# Prompt Template Format

Turn prompts into reusable, parameterised templates ready for programmatic injection.

## Template Syntax Options

Choose based on the user's stack:

| Syntax | Use when |
|--------|----------|
| `{{variable}}` | Handlebars/Mustache, LangChain, most JS/Python frameworks |
| `{variable}` | Python f-strings, simple string formatting |
| `<variable>` | XML-style, good for complex nested content |
| `$VARIABLE` | Shell-style, config files |
| `[VARIABLE]` | Readable placeholders in docs/specs |

Default to `{{variable}}` unless the user specifies a stack.

## Variable Types

### Scalar variables
Simple string substitution.
```
Summarise the following {{content_type}} in {{max_words}} words or fewer.
```

### Block variables
Multi-line content (documents, code, conversations).
```
Here is the document to analyse:
<document>
{{document_content}}
</document>
```

### Optional blocks
Content that may or may not be present. Use conditional syntax:
```
{{#if context}}
Use the following context to inform your answer:
<context>{{context}}</context>
{{/if}}
```

### List variables
Repeated items (e.g. few-shot examples, list of rules):
```
{{#each examples}}
Input: {{this.input}}
Output: {{this.output}}

{{/each}}
```

## Template Anatomy

A well-structured template:

```
[1] METADATA BLOCK (comments, version, author)
[2] SYSTEM / ROLE FRAME
[3] TASK INSTRUCTION (with scalar variables)
[4] OPTIONAL CONTEXT BLOCK (conditional)
[5] EXAMPLES BLOCK (optional, loop or static)
[6] INPUT BLOCK (block variable)
[7] OUTPUT FORMAT
```

## Variable Registry

Always accompany a template with a variable registry:

```
## Template Variables

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `content_type` | string | yes | Type of content being summarised | "legal contract" |
| `max_words` | integer | yes | Word limit for the summary | 150 |
| `context` | string | no | Optional background context | "This is for a non-expert audience" |
| `document_content` | string | yes | The full text to summarise | [paste document] |
```

## Process

1. Take the user's existing prompt or task description.
2. Identify all dynamic values (things that change per invocation).
3. Classify each as scalar, block, optional, or list.
4. Select the appropriate syntax for the user's stack.
5. Rewrite the prompt as a template with all variables extracted.
6. Produce the variable registry.
7. Show a filled example (instantiated with sample values) so the user can validate it reads naturally.

## Output Template

```
## Prompt Template

### Template ({{syntax}})
[full template with variables]

### Variable Registry
[table of variables]

### Filled Example
[template instantiated with realistic sample values — shows how it reads in production]

### Integration Notes
[any framework-specific notes, e.g. how to load with LangChain PromptTemplate, Jinja2, etc.]
```
