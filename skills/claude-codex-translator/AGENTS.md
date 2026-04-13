# AGENTS.md

## Purpose

Use this repository to translate between OpenAI Codex-style `AGENTS.md` files and Claude-compatible skill bundles centered on a `SKILL.md` file.

Treat this as a translation and restructuring task, not a literal file-format conversion. The goal is to preserve intent while adapting content to the destination format's conventions, strengths, and constraints.

## Primary task

When asked to convert an instruction file or skill bundle:

1. Identify the source format and target format.
2. Read the source material carefully, including any directly referenced support files.
3. Identify:
   - the artifact's core purpose
   - when it should be triggered or applied
   - expected inputs
   - expected outputs
   - workflow steps
   - constraints and guardrails
   - references to tools, commands, templates, scripts, or assets
4. Rewrite the content to fit the destination format instead of copying line by line.
5. Keep the result compact, reusable, and structurally correct for the target system.
6. Clearly note anything that was dropped, generalized, or approximated.

## Supported conversion directions

This repository should support both directions:

1. `AGENTS.md` -> Claude skill bundle
2. Claude skill bundle -> `AGENTS.md`

If the user provides ambiguous input or does not clearly state the desired output, infer it from context when reasonable. If direction is still unclear, ask a short clarifying question.

## Direction-specific workflow

### Converting `AGENTS.md` into a Claude skill

When the source is an `AGENTS.md` file:

1. Read the source `AGENTS.md` carefully.
2. Extract the reusable behavior from repo-scoped or environment-scoped instructions.
3. Produce a Claude skill structure that may include:
   - `SKILL.md`
   - optional `references/` files when source content is too long, too conditional, or too detailed for the main skill file
   - optional `scripts/` files only when deterministic execution is clearly better than prose instructions
   - optional `assets/` only if explicitly requested or already available
4. Rewrite imperative repo-agent rules into Claude skill guidance.
5. Preserve concrete conventions that matter to the workflow, but generalize hardcoded repo details unless the user explicitly wants a repo-bound skill.

### Converting a Claude skill into `AGENTS.md`

When the source is a Claude skill bundle:

1. Read `SKILL.md` first, then inspect only the support files that are necessary to preserve the workflow.
2. Identify which parts are:
   - trigger language
   - behavioral instructions
   - reusable reference material
   - deterministic logic that should remain as script or command references
3. Produce an `AGENTS.md` file that turns the source into clear, imperative agent instructions.
4. Convert explanatory guidance into direct operational rules, checks, and workflows.
5. Preserve important references to scripts, commands, templates, and assets when they are still required by the Codex-side workflow.

## Output requirements

Unless the user asks for something else, generate the destination artifact plus a short migration note.

### For `AGENTS.md` -> Claude skill conversions

Return:

1. A proposed skill name in lowercase kebab-case
2. A complete `SKILL.md`
3. A recommended folder structure
4. Any supporting reference files that should be split out from the main skill
5. A short explanation of what was preserved, what was transformed, and any ambiguities

### For Claude skill -> `AGENTS.md` conversions

Return:

1. A complete `AGENTS.md`
2. Any recommended supporting files or repo structure changes needed to keep the workflow usable
3. A short explanation of what was preserved, what was transformed, and any ambiguities

## Format rules to follow

### When producing `SKILL.md`

`SKILL.md` must begin with YAML frontmatter containing exactly:

- `name`
- `description`

Do not add extra frontmatter fields unless the user explicitly asks for a nonstandard format.

The `description` must do double duty:

- describe what the skill does
- describe when it should be used

Put trigger conditions in the frontmatter description, not only in the body.

The body of `SKILL.md` should:

- use direct operational instructions
- avoid long background exposition
- emphasize reusable behavior
- refer to supporting files explicitly when relevant
- keep the core file lean enough that an agent can load it efficiently

If the source contains large sections such as:

- detailed command references
- domain-specific policies
- long checklists
- many workflow variants

split those into files under `references/` and have `SKILL.md` point to them.

Only create `scripts/` files when:

- the task is deterministic
- repeatability matters
- the source clearly implies executable logic

Do not create scripts for text transformation work that the destination agent can reliably do directly.

### When producing `AGENTS.md`

`AGENTS.md` should read like a practical Codex instruction file:

- lead with a short statement of purpose and applicability
- prefer direct, imperative guidance
- include concrete workflow steps when actions are expected
- preserve guardrails and validation expectations
- mention scripts, commands, or files explicitly when they matter to execution

Do not preserve Claude-specific frontmatter or other destination-irrelevant metadata unless the user explicitly asks for a hybrid format.

Favor concise operational structure such as:

- purpose or applicability
- core workflow
- rules and guardrails
- validation or quality checks
- references to supporting files when needed

## Translation rules

### Preserve intent, not wording

Do not mirror the source file mechanically.
Rewrite for the destination format's semantics.

Examples:

- `Run npm test before finishing` may become `Run the project's required validation commands before finalizing changes` in a skill, or remain a concrete testing instruction in `AGENTS.md` if the repo-specific command is essential.
- `Claude should verify the file exists before proceeding` may become `Always verify the file exists before running commands` in `AGENTS.md`.

### Separate triggers from execution details

If the source mixes:

- when the artifact should be applied
- how the agent should perform the work

then move:

- trigger language into `description` when producing `SKILL.md`
- applicability language into the opening section when producing `AGENTS.md`
- long conditional details into `references/` when that keeps the main file leaner

### Convert local repo instructions into reusable guidance when appropriate

Source files often contain repo-scoped details such as:

- where code lives
- which tests to run
- naming conventions
- PR instructions
- directory-specific behavior

When converting:

- preserve reusable workflow knowledge
- keep repo-specific details only when they are essential or the user wants a repo-bound result
- generalize hardcoded paths where possible
- convert environment-specific advice into portable instructions when the destination format benefits from that abstraction

### Preserve concrete operational details when they matter

Do not over-generalize away information that makes the workflow usable.

Keep concrete details such as:

- important validation commands
- required tools or dependencies
- explicit prohibitions
- key file paths, scripts, templates, or assets

if removing them would make the translated output materially weaker.

### Flag irreducible ambiguity

If the source lacks enough detail to produce a high-quality conversion, do not invent specifics.
Instead:

- make the smallest reasonable assumptions
- clearly note open questions
- provide a best-effort draft

## Preferred output templates

### For Claude skill outputs

Use this structure unless the user requests otherwise:

- `skill-name/`
  - `SKILL.md`
  - `references/` (optional)
  - `scripts/` (optional)
  - `assets/` (optional)

Use this `SKILL.md` pattern:

---
name: <lowercase-kebab-name>
description: <one sentence explaining what the skill does and when Claude should use it>
---

# Overview

State the skill's job in a few lines.

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

### For `AGENTS.md` outputs

Use this pattern unless the user requests otherwise:

# <title>

Briefly state what this instruction file is for and when it applies.

## Core workflow

Describe the expected execution sequence in direct, operational language.

## Rules and guardrails

List imperative requirements, prohibitions, and decision criteria.

## Validation

State what the agent should verify before finishing.

## Supporting files

Reference scripts, templates, assets, or other files only when needed.

## Conversion heuristics

When deciding whether content belongs in the translated output:

- Keep:
  - reusable workflows
  - durable conventions
  - decision criteria
  - output formats
  - validation expectations
- Transform:
  - repo-specific paths into general patterns when portability matters more than fidelity
  - branch, commit, or PR rules into optional notes if they are not central to the destination artifact
  - environment setup into references when that keeps the main file focused
- Drop:
  - instructions meaningful only inside one temporary container, harness, or one-off evaluation context
  - temporary operational notes
  - duplicated reminders with no reusable value

## Validation checklist

Before returning a translated result, verify the appropriate checklist.

### For `SKILL.md` outputs

- frontmatter exists
- `name` is lowercase kebab-case
- `description` is lowercase and includes trigger conditions
- the body is concise and actionable
- extra files are included only when justified
- the result reads like a reusable Claude skill, not a repo README

### For `AGENTS.md` outputs

- the file reads as direct operational guidance
- workflows are concrete enough to execute
- rules are imperative and unambiguous
- validation expectations are present
- Claude-specific frontmatter and metadata are removed unless explicitly requested
- the result could plausibly be used as a Codex-side agent instruction file

## Default behavior

If the user provides only source instructions and asks for conversion, return the best-fit translated artifact for the requested direction plus a short migration note explaining the structural changes.
