---
name: create-skill
description: >
  Create and configure new custom AI skills with specific standards, 
  references, and instructions. Use this for requests like "make a skill",
  "create a custom skill", "build a new skill", or "set up a new skill".
---

# Create Skill Skill

To create new skill, use the following rules, variables, and procedure:

## Rules

> These rules are non-negotiable.

### 1. Escalation Behavior

> When in doubt, stop and ask. Proceeding with uncertainty is always worse than pausing.

You must stop and escalate to the user when:

- A required file or documentation does not exist and cannot be found.
- An assumption cannot be eliminated without information only the user can provide.
- External documentation research returns conflicting or unclear results.
- Any phase checklist item cannot be completed due to missing context.

You must never:

- Resolve uncertainty by making your best guess and proceeding silently.
- Assume that a missing file means the feature does not exist.
- Assume that silence from the user means confirmation.


### 2. File & Output Conventions

> Every output must be traceable, consistent, and stored in the correct location.

- Skill file names must use kebab-case and clearly reflect the scope of the skill.
  Format: `{skill-name}.md`
  Example: `data-design.md`
- File name must follow the skill's name.
- Reference files must be stored in a `references/` subdirectory inside the skill folder.
  Format: `references/{reference-name}.md`
  Example: `references/schema-design.md`
- You must never overwrite an existing skill file without first flagging it to the user and receiving explicit confirmation.
- Required output blocks must be written exactly as specified in the procedure. You must not paraphrase, shorten, or reformat them.

---

## Variables

> The variable will be called via {{VARIABLE_NAME}}

> These are global variables.

| Variable                | Value                                                |
| ----------------------  | ---------------------------------------------------- |
| CLAUDE_GLOBAL           | ~/.claude/agents                                     |
| CLAUDE_LOCAL            | ./.claude/agents                                     |
| OPENCODE_GLOBAL         | ~/.config/opencode/skills                            |
| OPENCODE_LOCAL          | ./.opencode/skills                                   |
| PI_GLOBAL               | ~/.pi/agent/skills                                   | 
| PI_LOCAL                | ./.pi/skills                                         | 
| SKILL_TEMPLATE          | [skill template](references/skill.template.md)       |
| REFERENCE_TEMPLATE      | [reference template](references/reference.template.md)|
| SKILL_EXAMPLE           | [skill example](assets/skill.example.md)             |
| REFERENCE_EXAMPLE       | [reference example](assets/reference.example.md)     |

> These are global variables, but you must fill it by yourself

| Variable      | Value |
| ------------- | ----- |
| USER_PLATFORM | empty |
| SKILL_SCOPE   | empty |

## Procedure

### Phase 1 - Knowledge fill

> Read all the knowledge needed

**Required output at the start of this phase:**

> Working on Phase 1 - Knowledge fill

- [ ] Read {{SKILL_TEMPLATE}} to understand the skill file structure
- [ ] Read {{REFERENCE_TEMPLATE}} to understand the reference file structure
- [ ] Read {{SKILL_EXAMPLE}} to understand the expected skill output
- [ ] Read {{REFERENCE_EXAMPLE}} to understand the expected reference output

**Required output before proceeding:**

>"Assets read
> [list]"
>
> Proceed to Phase 2

---

### Phase 2 - Context gathering

> Ask as detail to user

**Required output at the start of this phase:**

> Working on Phase 2 - Context gathering

**Constraint:**

1. Ask most 5 questions at a time per section

- [ ] Ask to user what platform do they use (claude, opencode, pi) and fill the {{USER_PLATFORM}}
- [ ] Ask to user every detail about the new skill based on the template until no ambiguity, assumptions, and missing detail
- [ ] Ask to user about the skill name, description, license, and compatibility
- [ ] Ask to user about the standards the skill must enforce
- [ ] Ask to user about the references needed (name, description, and content details for each reference)
- [ ] Ask to user where this skill will be created (global, local) and fill the {{SKILL_SCOPE}}

**Required output before proceeding:**

>"Skill detail
> [list]"
>
> Proceed to Phase 3

---

### Phase 3 - Reflection

> You must reflect on Phase 1 and 2 by answering every verification question honestly. Saying "yes" without evidence is itself a violation.

**Required output at the start of this phase:**

> Working on Phase 3 - Reflection

Answer every question below explicitly. Do not skip any. Do not answer with a generic "yes". Provide evidence for each answer.

**Phase 1 Verification:**

- [ ] Did I physically read `references` and `assets` files, or did I guess them from memory?

**Phase 2 Verification:**

- [ ] Did I ask clarifying questions, or did I assume the user's intent on my own?
- [ ] Can I list every assumption I had and confirm each one was explicitly resolved by the user?
- [ ] If any assumption was resolved by me instead of the user, I must return to Phase 1.

**Required output before proceeding:**

> "Reflections
>
> Phase 1:
> [answer each question with evidence]
>
> Phase 2:
> [answer each question with evidence]
>
> Proceeding to Phase 4"

---

### Phase 4 - Create skill

> Create the new skill based on gathered context

**Required output at the start of this phase:**

> Working on Phase 4 - Create skill

- [ ] Determine where to create the skill based on {{SKILL_SCOPE}} and {{USER_PLATFORM}}
- [ ] Make sure the folder exists
- [ ] Create the skill folder with the correct name
- [ ] Create the `references/` subdirectory inside the skill folder
- [ ] Create the skill file based on {{SKILL_TEMPLATE}} with the gathered details
- [ ] Create each reference file based on {{REFERENCE_TEMPLATE}} with the gathered details
- [ ] Ensure all routes in the skill file correctly link to the reference files

**Required output before proceeding:**

>"Skill created
> [path]"
>
> Proceed to Phase 5

---

### Phase 5 - Report

> Providing report to the user

**Required output at the start of this phase:**

> Working on Phase 5 - Report

- [ ] Return the following output to the user exactly as specified.

```markdown
The skill is done.

File path - [file-path]
```
