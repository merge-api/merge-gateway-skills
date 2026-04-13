# skill-converter

A translator skill for converting AI instruction artifacts between **Claude `SKILL.md`** skill bundles and **OpenAI Codex `AGENTS.md`** agent files, in both directions.

---

## What it does

Claude skills and Codex agent files serve similar purposes (telling an AI system how to behave) but are written very differently:

- **Claude `SKILL.md`** files are *knowledge briefings* — prose written for Claude to read and internalize, explaining what to do and why
- **Codex `AGENTS.md`** files are *behavioral policies* — imperative directives telling an autonomous agent what commands to run, what rules to follow, and what it's prohibited from doing

This repository handles translation between them as a restructuring task rather than a literal format conversion. The conversion is inherently asymmetric: `SKILL.md -> AGENTS.md` strips or compresses reasoning and surfaces concrete directives, while `AGENTS.md -> SKILL.md` adds context, wraps commands in explanation, and synthesizes trigger language.

---

## Trigger phrases

Install this skill and Claude will load it when you say things like:

- "Convert this skill to a Codex agent file"
- "Turn this AGENTS.md into a Claude skill"
- "Port my skill file for Codex"
- "Make this work with Claude Code"
- "Translate this AGENTS.md for Claude"
- "Make this skill cross-compatible"

---

## Invocation examples

You can invoke the skill by naming it directly or by making the conversion intent explicit.

Examples:

- `Use claude-codex-translator to convert this AGENTS.md into a Claude skill.`
- `Use claude-codex-translator to turn this SKILL.md into a Codex AGENTS.md file.`
- `Convert this skill to Codex.`
- `Turn this AGENTS.md into a Claude skill.`
- `Make this skill cross-compatible.`

You can also point it at a specific path:

- `Use claude-codex-translator on /path/to/AGENTS.md and generate a SKILL.md bundle.`
- `Use claude-codex-translator on /path/to/SKILL.md and produce an AGENTS.md version.`

---

## Usage

### SKILL.md → AGENTS.md

Paste or upload your `SKILL.md` and ask for a Codex-compatible agent file. The translator will:

1. Strip the YAML frontmatter and use `name`/`description` as the Codex header
2. Rewrite instructional prose into imperative directives
3. Surface concrete shell commands for any workflow steps
4. Add practical sections such as workflow, guardrails, and validation when needed
5. Drop pure-knowledge content that doesn't translate to agent policy
6. Flag anything that was lossy or approximated

### AGENTS.md → SKILL.md

Paste or upload your `AGENTS.md` and ask for a Claude skill. The translator will:

1. Generate YAML frontmatter with a trigger-optimized `description`
2. Propose a lowercase kebab-case skill name
3. Rewrite imperative rules into reasoned guidance, adding *why* where inferable
4. Wrap shell commands in explanatory context
5. Expand terse rules into principles
6. Suggest `references/` files when the source is too long or too conditional for a lean `SKILL.md`
7. Flag directory-scoped sections or environment-specific commands that don't translate cleanly

### Support files

The translator may recommend or generate supporting files when they improve fidelity without bloating the main artifact:

- `references/` for long checklists, command catalogs, workflow variants, or domain-specific policies
- `scripts/` only when deterministic execution is clearly better than prose instructions
- `assets/` only when explicitly requested or already part of the source material

---

## Lossy conversions

Not everything maps cleanly. Claude will tell you when something was dropped or approximated rather than silently omitting it. Common cases:

| Content | Direction | What happens |
|---|---|---|
| Bundled scripts | SKILL → Codex | Becomes a repo path reference; you'll need to commit the script |
| Conceptual/educational prose | SKILL → Codex | Summarized or omitted; can be restored as comment blocks |
| Directory-scoped overrides | Codex → SKILL | Flagged; skills don't have directory scoping |
| CI/CD-specific commands | Codex → SKILL | Kept, but noted as environment-dependent |

---

## File structure

```
skill-converter/
├── SKILL.md                      # Main skill — conversion logic and mappings
├── AGENTS.md                     # Codex-side instructions for the same bidirectional translator
└── references/
    └── format-guide.md           # Deep reference: annotated examples, edge cases, idioms
```

`references/format-guide.md` contains annotated examples of both file formats, edge cases, and notes on the structural conventions each system expects.

---

## Installation

Install `skill-converter.skill` through your Claude skills interface. No additional dependencies required.
