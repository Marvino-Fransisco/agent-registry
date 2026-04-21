# Docs read

efficiently extract useful design constraints from project documentation, without reading everything.

## Goal

Answer these questions from docs alone:

- What endpoints/data does the feature have access to?
- What design tokens / component library must be used?
- Are there stated constraints (auth, permissions, feature flags)?
- Are there recent breaking changes relevant to this work?

## Step 1 — Locate Documentation Files

```bash
( \
  find . -maxdepth 2 -type f -name "README.md" -not -path "*/node_modules/*" ; \
  find ./docs ./researches ./designs -type f -name "*.md" -not -path "*/node_modules/*" \
)
```

## Step 2 — Read README (Skim, Don't Read All)

Focus on these sections only:

- **Getting started / setup** → confirms assumed stack
- **Architecture / structure** → high-level decisions already made
- **Environment variables** → what API keys / feature flags exist
- **Constraints / limitations** → anything explicitly off-limits

Skip: badges, license sections, long installation instructions, CI/CD pipeline details.

## Step 3 — Read API Contracts

Priority order:

1. `openapi.yaml` / `swagger.json` → machine-readable source of truth
2. Handwritten `API.md` → human descriptions
3. TypeScript service files (`*.api.ts`, `*.service.ts`) → infer from types

For each relevant endpoint, extract:

```markdown
## Endpoint: [method] [path]
- Description: [what it does]
- Request body: [key fields]
- Response shape: [key fields]
- Auth required: [yes/no]
- Errors: [known error codes/messages]
```

Only extract endpoints relevant to the feature being designed.

## Step 4 — Read Design Tokens / Theme

```bash
cat tailwind.config.ts        # Tailwind custom tokens
cat src/styles/tokens.ts      # Manual tokens
cat src/theme/index.ts        # Theme object
```

Extract and summarize:

```markdown
## Design Tokens Available
- Colors: [primary, secondary, destructive, muted, etc — list with values]
- Typography: [font families, size scale]
- Spacing: [scale reference]
- Border radius: [sm, md, lg values]
- Shadows: [available levels]
- Breakpoints: [sm, md, lg, xl values]
```

If a component library is in use (shadcn/ui, Radix, MUI, Ant Design), note which components are already installed and available.

## Step 5 — Check CHANGELOG for Recent Breaks

```bash
head -80 CHANGELOG.md
```

Look for entries in the last 2–3 versions that mention:

- Breaking changes to APIs the feature will use
- Deprecated patterns the new feature must avoid
- New utilities or components now available

## Step 6 — Output the Documentation Summary

```markdown
## Documentation Summary

### Project Overview
[1–2 sentences from README]

### Relevant API Endpoints
[Table or list of endpoints the feature will need]

| Method | Path | Purpose | Auth |
| -------- | ------ | --------- | ------ |
| GET    | /api/... | ... | Yes |

### Design System
- Component library: [shadcn/ui / MUI / custom / none]
- Available components relevant to this feature: [list]
- Token reference: [key colors, spacing, typography]

### Constraints Noted
- [Any explicit constraints from docs that affect the design]

### Recent Changes to Know
- [CHANGELOG entries relevant to this feature]

### Missing Documentation
- [Anything that should be documented but isn't — flag for the design]
```

## What to Skip

Never read or summarize:

- License files
- CI/CD pipeline configs (`.github/workflows/`)
- Deployment configs (`Dockerfile`, `docker-compose.yml`) unless the feature has SSR/SSG implications
- Test setup files
- Long lists of contributors

## Edge Cases

- **No README**: Check if there's a `docs/` folder or a Notion/Confluence link in `package.json` homepage field.
- **OpenAPI too large**: Only parse the paths relevant to the feature. Ignore unrelated endpoints entirely.
- **No design tokens**: Note this in the summary — the design will need to make token decisions from scratch.
- **Conflicting docs**: If README says one thing and the code says another, trust the code. Flag the conflict.
