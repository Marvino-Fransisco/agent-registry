---
name: create-agent
description: >
  Create and configure new custom AI agents with specific instructions, 
  behaviors, tools, and capabilities. Use this for requests like "make an agent",
  "create a custom agent", "build a new AI assistant", or "set up a new agent".
---

# Create Agent Skill

To create new agent, use the following rules, variables, and procedure:

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

- File names must use kebab-case and clearly reflect the scope of the design.
  Format: `{agent-name}.md`
  Example: `data-designer.md`
- File name must follow the agent's name.
- You must never overwrite an existing design file without first flagging it to the user and receiving explicit confirmation.
- Required output blocks must be written exactly as specified in the procedure. You must not paraphrase, shorten, or reformat them.

---

## Variables

> The variable will be called via {{VARIABLE_NAME}}

> These are global variables.

| Variable                | Value                                         |
| ----------------------  | --------------------------------------------- |
| CLAUDE_GLOBAL           | ~/.claude/agents                              |
| CLAUDE_LOCAL            | ./.claude/agents                              |
| OPENCODE_GLOBAL         | ~/.config/opencode/agents                     |
| OPENCODE_LOCAL          | ./.opencode/agents                            |
| TEMPLATE                | [template](assets/template.md)                |
| EXAMPLE                 | [example](assets/example.md)                  |
| CLAUDE_YAML_TEMPLATE    | [claude_template](assets/claude.yaml.md)      |
| OPENCODE_YAML_TEMPLATE  | [opencode_template](assets/opencode.yaml.md)  |
| REGISTRY                | ~/agent-registry/agents/{{USER_PLATFORM}}     |

> These are global variables, but you must fill it by yourself

| Variable      | Value |
| ------------- | ----- |
| USER_PLATFORM | empty |
| AGENT_SCOPE   | empty |

## Procedure

### Phase 1 - Knowledge fill

> Read all the knowledge needed

**Required output at the start of this phase:**

> Working on Phase 1 - Knowledge fill 

- [ ] Read {{TEMPLATE}} To understad what you need to fill
- [ ] Read {{EXAMPLE}} To understand the expected file output

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

- [ ] Ask to user every detail about the new agent based on the template until no ambiguity, assumptions, and missing detail
- [ ] Ask to user where this agent will be created (global, local, registry) and fill the {{AGENT_SCOPE}}
- [ ] Ask to user what platform do they use (claude, opencode) and fill the {{USER_PLATFORM}}

**Required output before proceeding:**

>"Agent detail
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

- [ ] Did I physically read `assets` files, or did I guess them from memory?

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

### Phase 4 - Create agent

> Create the new agent based on gathered context

**Required output at the start of this phase:**

> Working on Phase 4 - Create agent

- [ ] Determine where to create the agent based on {{AGENT_SCOPE}} and {{USER_PLATFORM}}
- [ ] Make sure the folder is exists
- [ ] Copy {{TEMPLATE}} to the designated folder
- [ ] Based on {{USER_PLATFORM}} copy the yaml template content
- [ ] Rename the file and update the file content

**Required output before proceeding:**

>"Agent created
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
The design is done.

File path - [file-path]
```
