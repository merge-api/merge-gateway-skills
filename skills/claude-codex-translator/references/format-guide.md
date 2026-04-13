# Format Guide: Claude SKILL.md vs Codex AGENTS.md

Deep reference for both formats — conventions, annotated examples, structural rules, and edge cases. Read this before performing any conversion.

---

## Claude SKILL.md Format

### Purpose

A SKILL.md is a **knowledge briefing** that loads into Claude's context when triggered. It tells Claude *what to know* and *how to think* about a class of tasks. The file is written **for Claude to read**, not for the user.

### Structural Rules

1. **YAML frontmatter is required.** It must contain:
   - `description` — a trigger-optimized summary of what the skill does and when it should activate. This is the most important field: Claude Code's skill-matching system uses it to decide when to load the skill. Be specific and "pushy" — list concrete phrases, task types, and contexts.
   - `allowed-tools` — comma-separated list of tools the skill may use (e.g., `Read, Grep, Glob, Edit, Write, Bash`).

2. **No `name` field in frontmatter.** The skill name comes from the directory name, not from YAML.

3. **Body is Markdown prose** — written as expert guidance, not a rulebook. Sections use `##` headings.

4. **Commands are examples, not directives.** Shell commands appear inside explanatory prose to illustrate *when* and *why* to run them, not as bare instructions.

5. **The "why" matters.** Every rule or recommendation should carry its rationale. "Always work on a feature branch" is weaker than "Always work on a feature branch — committing directly to `main` bypasses review gates and can break CI for the whole team."

6. **Progressive disclosure.** Keep the main SKILL.md lean (<300 lines is ideal). Split detailed references, long checklists, and domain-specific policies into `references/` files and point to them from the body.

### Frontmatter Examples

```yaml
---
description: Migrate from OpenRouter to Merge Gateway. Use when the user wants to switch from OpenRouter, replace OpenRouter, or move off OpenRouter.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---
```

```yaml
description: Convert between Claude SKILL.md files and OpenAI Codex AGENTS.md files. Use when the user wants to translate, convert, port, or adapt an AI skill or agent instruction file from one format to the other.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
```

### Annotated Example

```markdown
---
description: Set up database migrations for a Django project. Use when the user asks about migrations, schema changes, makemigrations, or migrate commands.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

# Django Database Migrations

Help developers create, review, and apply Django database migrations safely.

## Context & Prerequisites

Django migrations are Python files that describe schema changes. They live in
`<app>/migrations/` and are applied in dependency order. Before working with
migrations, verify the project uses Django by checking for `django` in
requirements and a `manage.py` at the project root.

## Workflow

### Creating a migration

After the user modifies a model, generate the migration:

    python manage.py makemigrations <app_name>

Review the generated file — Django's auto-detector is good but not perfect.
Watch for:

- **Renamed fields** being detected as remove + add (data loss). If you see
  a `RemoveField` paired with an `AddField` for what looks like a rename,
  suggest `RenameField` instead.
- **Default values** on new non-nullable columns. Django will prompt for a
  default; prefer setting `default=` on the model field explicitly.

### Applying migrations

    python manage.py migrate

If this fails, read the traceback carefully before suggesting `--fake` —
faking a migration marks it as applied without running it, which can leave
the schema out of sync.

## Key Principles

- Never edit a migration that has already been applied to a shared database.
  Create a new migration to alter or undo the change.
- Squash migrations only when the chain is long and all intermediate states
  have been fully deployed. Premature squashing blocks rollback.

## Common Pitfalls

- Running `makemigrations` without specifying the app can pick up unrelated
  model changes from third-party packages.
- Circular migration dependencies usually mean your apps have tightly coupled
  models — suggest extracting a shared app.
```

### What makes this good

- **Frontmatter** has a rich trigger description with specific phrases
- **Prose style** — reads like an expert colleague explaining, not a checklist
- **Commands are embedded in context** — `makemigrations` appears inside guidance about *what to watch for*, not as a bare step
- **"Why" is always present** — every warning explains the consequence
- **Concise** — the whole file is scannable without being shallow

---

## Codex AGENTS.md Format

### Purpose

An AGENTS.md is a **behavioral policy** for an autonomous Codex agent. It tells the agent *what to do*, *what commands to run*, and *what's prohibited*. The file is written **as rules to follow**, not knowledge to internalize.

### Structural Rules

1. **No YAML frontmatter.** The file starts with a Markdown `# Heading` or an opening paragraph.

2. **Imperative voice throughout.** Sentences are directives: "Run `npm test`", "Always check...", "Never commit to...". Avoid passive voice and explanatory prose.

3. **Shell commands are first-class.** Every workflow step should include the actual command to run. Don't say "run the linter" — say "Run `npm run lint`."

4. **Sections are functional**, not narrative. Common sections:
   - Environment Setup
   - Core Workflow / Steps
   - Rules / Conventions
   - Testing
   - Prohibited Actions

5. **Terse is better.** Agent files are policies, not tutorials. One sentence per rule is ideal. Don't explain *why* unless the rule would be misapplied without context.

6. **Directory scoping.** Codex supports placing `AGENTS.md` files in subdirectories to override or extend the root file. Rules in a subdirectory apply only to work in that subtree.

### Annotated Example

```markdown
# Django Migration Agent

Manage Django database migrations safely for this project.

## Environment Setup

- Python 3.11+
- Django is installed in the project virtualenv
- Activate the virtualenv before running any commands: `source .venv/bin/activate`

## Core Workflow

1. After modifying any model, run:
   ```
   python manage.py makemigrations <app_name>
   ```
2. Review the generated migration file for correctness.
3. Apply the migration:
   ```
   python manage.py migrate
   ```
4. Run the test suite to verify nothing broke:
   ```
   python manage.py test
   ```

## Rules

- Always specify the app name when running `makemigrations`.
- Never edit a migration that has been applied to a shared database.
- Never use `--fake` without explicit user approval.
- If `makemigrations` detects a field removal paired with a field addition, check if it should be a `RenameField` instead.
- Always set an explicit `default=` on new non-nullable fields rather than relying on the interactive prompt.

## Testing

Run the full test suite after every migration:
```
python manage.py test
```

A passing suite means the migration is safe to commit.

## Prohibited Actions

- Do not run `migrate --fake` automatically.
- Do not squash migrations unless all intermediate migrations are deployed.
- Do not modify migrations in `main` branch history.
- Do not run `makemigrations` without specifying the app name.
```

### What makes this good

- **No frontmatter** — starts with a heading
- **Every step has a runnable command** — no vague instructions
- **Rules are one-liners** — scannable, unambiguous
- **Prohibited Actions are explicit** — clear guardrails
- **No "why" prose** — the agent doesn't need motivation, just policy

---

## Side-by-Side Comparison

| Dimension | SKILL.md | AGENTS.md |
|---|---|---|
| Audience | Claude (reasoning assistant) | Codex (autonomous agent) |
| Voice | Expert colleague explaining | Manager issuing directives |
| Commands | Examples inside explanatory prose | First-class steps to execute |
| Rationale ("why") | Always included | Rarely included |
| Frontmatter | Required (`description`, `allowed-tools`) | None |
| Naming | Directory name is the skill name | H1 heading is the agent name |
| Structure | Narrative sections (Context, Workflow, Principles, Pitfalls) | Functional sections (Setup, Steps, Rules, Testing, Prohibitions) |
| Tone | "Consider...", "Watch out for...", "This matters because..." | "Always...", "Never...", "Run..." |
| Length | Moderate prose (~100-300 lines) | Compact directives (~50-150 lines) |
| Supporting files | `references/`, `scripts/`, `assets/` dirs | Directory-scoped override files |
| Scoping | Single file per skill, no directory scoping | Can have root + subdirectory overrides |

---

## Edge Cases & Conversion Notes

### Content that doesn't translate cleanly

**SKILL.md -> AGENTS.md:**

- **Educational prose** — background explanations that help Claude reason don't belong in an agent policy. Summarize into a one-line comment or drop entirely. Flag what you dropped.
- **Bundled scripts** — SKILL.md can reference scripts in `scripts/` that Claude runs. Codex expects commands, not script paths. Either inline the script's logic as commands or tell the user to commit the script and reference its repo path.
- **Conditional reasoning** — "If the project uses X, consider Y because Z" becomes "If X is detected, do Y." The reasoning gets stripped.
- **Tool references** — SKILL.md may reference Claude-specific tools (`Read`, `Grep`, `Edit`). These have no Codex equivalent — translate to shell commands (`cat`, `grep`, `sed`/manual edit).
- **`allowed-tools` field** — has no Codex equivalent. Drop it silently.

**AGENTS.md -> SKILL.md:**

- **Directory-scoped overrides** — SKILL.md has no directory scoping mechanism. Merge subdirectory rules into the main file as conditional sections ("When working in the `frontend/` directory...").
- **Environment setup** — agent files often list exact setup commands. In a SKILL.md, these become prerequisites context ("This project uses Python 3.11+ with Django in a virtualenv").
- **Bare commands without context** — every command in a SKILL.md needs surrounding prose explaining when to run it and what to look for. Don't just list commands.
- **Implicit environment assumptions** — agent files assume a specific runtime. SKILL.md should make these explicit as prerequisites.

### Ambiguous inputs

If a file has both YAML frontmatter AND imperative directives, it may be a hybrid. Look for:
- If it has `description:` and `allowed-tools:` in frontmatter — treat as SKILL.md
- If it has `name:` and `description:` in frontmatter but imperative body — likely a SKILL.md with an unusual style
- If frontmatter is informal comments, not YAML — treat as AGENTS.md
- When truly ambiguous, ask the user which direction they want

### Length guidelines

- A typical SKILL.md conversion should land between 100-300 lines
- A typical AGENTS.md conversion should land between 50-150 lines
- If the output exceeds these ranges significantly, consider whether content should be split (for SKILL.md, into `references/`) or trimmed (for AGENTS.md, dropping non-essential context)

### Model/tool name mapping

When converting, watch for references to platform-specific tooling:

| Claude Code concept | Codex equivalent |
|---|---|
| `Read` tool | `cat` / `head` / `tail` |
| `Grep` tool | `grep` / `rg` |
| `Glob` tool | `find` / `ls` |
| `Edit` tool | manual file edit / `sed` |
| `Write` tool | write file / `cat <<EOF` |
| `Bash` tool | shell commands directly |
| `Agent` tool | no direct equivalent — flatten into inline steps |
| Skill references (`/skill-name`) | no equivalent — inline the relevant instructions |

### Preserving fidelity

The goal is **intent preservation**, not mechanical translation. A good conversion:
- Keeps every actionable piece of information from the source
- Adapts the *form* to match the target format's conventions
- Flags anything that was dropped, approximated, or ambiguous
- Doesn't invent information that wasn't in the source
- Doesn't pad with boilerplate just to fill expected sections
