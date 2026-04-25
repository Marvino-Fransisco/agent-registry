# Claude Code Yaml Template

| Field            | Required | Description                                                                                                           |
|------------------|----------|-------------                                                                                                          |
| name             | Yes      | Unique identifier using lowercase letters and hyphens                                                                 |
| description      | Yes      | When Claude should delegate to this subagent                                                                          |
| tools            | No       | Tools the subagent can use. Inherits all tools if omitted                                                             |
| disallowedTools  | No       | Tools to deny, removed from inherited or specified list                                                               |
| model            | No       | Model to use: sonnet, opus, haiku, a full model ID (e.g., claude-opus-4-7), or inherit. Defaults to inherit           |
| permissionMode   | No       | Permission mode: default, acceptEdits, auto, dontAsk, bypassPermissions, or plan                                      |
| maxTurns         | No       | Maximum number of agentic turns before the subagent stops                                                             |
| skills           | No       | Skills to load into the subagent’s context at startup. Full skill content is injected; subagents don’t inherit skills |
| mcpServers       | No       | MCP servers available to this subagent. Can reference configured servers (e.g., "slack") or inline definitions        |
| hooks            | No       | Lifecycle hooks scoped to this subagent                                                                               |
| memory           | No       | Persistent memory scope: user, project, or local. Enables cross-session learning                                      |
| background       | No       | Set to true to always run as a background task. Default: false                                                        |
| effort           | No       | Effort level override: low, medium, high, xhigh, max. Defaults to inherit                                             |
| isolation        | No       | Set to worktree to run in a temporary git worktree with isolated repo copy                                            |
| color            | No       | Display color in task list/transcript: red, blue, green, yellow, purple, orange, pink, cyan                           |
| initialPrompt    | No       | Auto-submitted as first user turn when used as main agent. Prepended to user prompt                                   |

```yaml
---
name: code-reviewer
description: Reviews code for security issues, performance problems, and best practices
tools: Read, Glob, Grep, Bash
disallowedTools: Write, Edit
model: sonnet
permissionMode: acceptEdits
maxTurns: 10
skills: security, performance, testing
mcpServers:
  - github:
      type: http
      url: https://mcp.github.com/api
      headers:
        Authorization: "Bearer ${GITHUB_TOKEN}"
  - slack: # References pre-configured server
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "echo 'Validating bash command...'"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "npx prettier --write"
memory: project
background: false
effort: high
isolation: worktree
color: blue
initialPrompt: "Review the provided code and identify security vulnerabilities, performance issues, and suggest improvements."
---
```
