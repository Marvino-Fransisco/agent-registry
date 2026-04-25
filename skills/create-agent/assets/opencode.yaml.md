# Opencode Yaml Template

```yaml
---
# Required/Optional Configuration Fields
description: "Brief description of the agent's role and when to use it"
temperature: 0.1             # Sampling temperature (optional)
steps: 100                   # Max steps for execution (optional)
disable: false
model: provider/model        # e.g., "anthropic/claude-sonnet-4-20250514"
mode: primary | subagent     # 'primary' for main interaction, 'subagent' for delegation
hidden: false                # Hide from default TUI list (optional)
color: "#FF5733"             # TUI display color (optional)
top_p: 0.95                  # Nucleus sampling parameter (optional)

# Permission Constraints (Optional granular control)
permissions:
  edit: ask                  # 'ask', 'allow', or 'deny'
  write: ask
  bash:
    "*": deny                # Deny all bash commands by default

task:
    "*": deny                # 'ask', 'allow', or 'deny'
    "orchestrator-*": allow
    "backend": ask

# Skill Loading (Optional)
skill: true                  # Enable dynamic loading of skills
---
```
