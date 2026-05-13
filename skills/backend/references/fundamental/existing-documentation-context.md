# Existing Documentation Context

## Existing Documentation Context Standards

- Documentation must be read before making any design or code decisions
- Documentation gaps must be flagged explicitly — never silently assumed
- If documentation does not exist, that fact must be reported to the user

---

## Instructions

### Step 1 - Identification

> Identify what documentation exists in the project

- Check for `README.md` at the root
- Check for `docs/` directory
- Check for `ARCHITECTURE.md`, `CONTRIBUTING.md`, `AGENTS.md`, `DESIGN.md`
- Check for inline documentation (JSDoc, docstrings, comments)
- Check for API documentation (OpenAPI, Swagger, Postman collections)
- Check for design documents in `designs/` folder
- Check for ADRs (Architecture Decision Records)

---

### Step 2 - Read and Extract

> Read all relevant documentation and extract key information

- Extract the project description and purpose
- Extract the technology stack and version constraints
- Extract the architecture decisions and their rationale
- Extract the coding conventions and standards
- Extract the deployment and environment requirements
- Extract any known limitations or technical debt

---

### Step 3 - Assess Gaps

> Identify what is missing from the documentation

- Is the architecture documented?
- Are the API contracts documented?
- Are the environment variables documented?
- Are the coding conventions documented?
- Is the deployment process documented?

---

### Step 4 - Output

> Summarize the documentation context

- List all documentation files found and their key takeaways
- List any gaps in documentation
- State how the documentation context influences the current task

---

## Edge Cases

### 1. Outdated Documentation

> Documentation exists but does not match the current codebase

**Example:**

- README says the project uses MongoDB but the codebase has migrated to PostgreSQL

**Rule of Thumb:**

- Trust the code over the documentation
- Flag the inconsistency to the user

**Practical Trade-off:**

- Outdated docs are worse than no docs — they mislead
- Flag for update but do not block current work

---

### 2. No Documentation at All

> The project has zero documentation

**Example:**

- No README, no docs folder, no inline comments

**Rule of Thumb:**

- Report the absence to the user
- Derive all context from reading the actual code
- Recommend creating documentation as a follow-up task

**Practical Trade-off:**

- Reading code is slower but more accurate than reading outdated docs
- Documentation creation should not block the current task

---

## Bad Example

### 1. Assuming Documentation Accuracy Without Verification

> Reading `README.md` that says "All endpoints require authentication" and assuming it is true without checking the actual middleware registration

### 2. Proceeding Without Checking Documentation

> Making architecture decisions without reading existing `ARCHITECTURE.md` that already defines the conventions

---

## Good Example

### 1. Honest Documentation Assessment

> Documentation scan results:
>
> - `README.md` — Found. Contains setup instructions. Stack description is outdated (lists MongoDB, codebase uses PostgreSQL).
> - `docs/api.md` — Found. API contracts documented for v1 endpoints only. v2 endpoints are undocumented.
> - `AGENTS.md` — Found. Defines coding conventions and test commands.
> - `docs/architecture.md` — Not found.
> - ADRs — Not found.
>
> Gaps: Architecture documentation missing. v2 API undocumented. README outdated.
>
> Action: Proceeding with code-based context. Flagging documentation gaps for user awareness.
