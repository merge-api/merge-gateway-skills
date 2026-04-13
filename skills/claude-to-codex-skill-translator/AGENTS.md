# AGENTS.md

## Purpose

Use this repository to convert agent instruction files written in `AGENTS.md` style into Claude-compatible skill bundles centered on a `SKILL.md` file.

The agent should treat this as a translation and restructuring task, not a literal file-format conversion. The goal is to preserve intent while adapting content to Claude skill conventions.

## Primary task

When asked to translate an agent file into a Claude skill:

1. Read the source `AGENTS.md` carefully.
2. Identify:
   - the skill's core purpose
   - when the skill should be triggered
   - expected inputs
   - expected outputs
   - workflow steps
   - constraints and guardrails
   - references to tools, commands, templates, or assets
3. Produce a Claude skill structure that includes:
   - `SKILL.md`
   - optional `references/` files when the source content is too long or conditional
   - optional `scripts/` files only when deterministic execution is clearly better than prose instructions
   - optional `assets/` only if explicitly requested or already available
4. Rewrite instructions to fit Claude skill patterns rather than copying the original wording line by line.
5. Keep the resulting skill compact, reusable, and portable.

## Output requirements

Unless the user asks for something else, generate:

1. A proposed skill name in lowercase kebab-case
2. A complete `SKILL.md`
3. A recommended folder structure
4. Any supporting reference files that should be split out from the main skill
5. A short explanation of what was preserved, what was transformed, and any ambiguities

## Claude skill rules to follow

### SKILL.md structure

`SKILL.md` must begin with YAML frontmatter containing exactly:

- `name`
- `description`

Do not add extra frontmatter fields unless the user explicitly asks for a nonstandard format.

### Description quality

The `description` must do double duty:
- describe what the skill does
- describe when it should be used

Put trigger conditions in the frontmatter description, not only in the body.

### Body style

The body of `SKILL.md` should:

- use direct operational instructions
- avoid long background exposition
- emphasize reusable behavior
- refer to supporting files explicitly when relevant
- keep the core file lean enough that an agent can load it efficiently

### Progressive disclosure

If the source `AGENTS.md` contains large sections such as:
- detailed command references
- domain-specific policies
- long checklists
- many workflow variants

then split those into files under `references/` and have `SKILL.md` point to them.

### Scripts

Only create `scripts/` files when:
- the task is deterministic
- repeatability matters
- the source clearly implies executable logic

Do not create scripts for text transformation work that Claude can reliably do directly.

## Translation rules

### Convert local repo instructions into reusable skill instructions

`AGENTS.md` often contains repo-scoped guidance such as:
- where code lives
- which tests to run
- naming conventions
- PR instructions
- directory-specific behavior

When converting to a Claude skill:

- preserve reusable workflow knowledge
- remove repo-specific details unless the user explicitly wants a repo-bound skill
- generalize hardcoded paths where possible
- convert environment-specific advice into portable instructions

### Preserve intent, not wording

Do not mirror the source file mechanically.
Rewrite for Claude skill semantics.

Examples:
- “Run `npm test` before finishing” may become “Run the project’s required validation commands before finalizing changes.”
- “Files in `frontend/` follow X pattern” may become a scoped convention section only if it matters to the skill’s reusable behavior.

### Separate triggers from execution details

If the source mixes:
- when the agent should apply behavior
- how the agent should perform the work

then move:
- trigger language into `description`
- execution details into the body
- long conditional details into `references/`

### Flag irreducible ambiguity

If the source `AGENTS.md` lacks enough detail to produce a high-quality Claude skill, do not invent specifics.
Instead:
- make the smallest reasonable assumptions
- clearly note open questions
- provide a best-effort draft

## Preferred output template

When generating a skill, use this structure unless the user requests otherwise:

- `skill-name/`
  - `SKILL.md`
  - `references/` (optional)
  - `scripts/` (optional)
  - `assets/` (optional)

## SKILL.md template

Use this pattern:

---
name: <lowercase-kebab-name>
description: <one sentence explaining what the skill does and when Claude should use it>
---

# Overview

State the skill’s job in a few lines.

## Instructions

Describe the required workflow in imperative language.

## Supporting resources

Point to files in `references/` or `scripts/` only when needed.

## Output expectations

Define what good output looks like.

## Constraints

List important guardrails and non-goals.

## Quality bar

State what the agent should verify before finishing.

## Conversion heuristics

When deciding whether content from `AGENTS.md` belongs in the Claude skill:

- Keep:
  - reusable workflows
  - durable conventions
  - decision criteria
  - output formats
  - validation expectations
- Transform:
  - repo-specific paths into general patterns
  - branch/commit/PR rules into optional workflow notes only if still relevant
  - environment setup into references when needed
- Drop:
  - instructions meaningful only inside one specific container or one evaluation harness
  - temporary operational notes
  - duplicated reminders with no reusable value

## Validation checklist

Before returning a translated skill, verify:

- frontmatter exists
- `name` is lowercase kebab-case
- `description` is lowercase and includes trigger conditions
- the body is concise and actionable
- extra files are included only when justified
- the result reads like a reusable Claude skill, not a repo README

## Default behavior

If the user provides only an `AGENTS.md` file and asks for conversion, return:

1. proposed skill name
2. full `SKILL.md`
3. any suggested supporting files
4. a short migration note explaining the structural changes
