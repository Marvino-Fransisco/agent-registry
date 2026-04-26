---
name: data-design
description: >
  Use this skill when working on tasks, answering questions, or brainstorming related to data structure, schema design, and data flow. Focus on keeping designs simple, consistent, and scalable as complexity grows.
license: MIT
compatibility: Designed for AI Agent
metadata:
  author: Marvino Fransisco
  version: "1.0"
---

# Data Design Skill

This skill contains routes to references that should be read **before** dealing with data-related problems.

## Standards

- Never expose Database column names directly
- `snake_case` for Database columns
- `camelCase` for API responses
- plural table names
- soft delete approach
- slug when needed

## Routes

| Reference | Description |
| --------- | ----------- |
| [schema-design](references/schema-design.md) | read this reference when handling database schema design |
| [sql](references/sql.md) | read this reference when handling query |
| [storage-guide](references/storage-guide.md) | read this reference when handling storage |
| [api-validation](references/api-validation.md) | read this reference when handling api and api contracts |
