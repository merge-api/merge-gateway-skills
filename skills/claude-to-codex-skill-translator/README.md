# skill-converter

A Claude skill that converts AI instruction files between **Claude `SKILL.md`** format and **OpenAI Codex `AGENTS.md`** format — in both directions.

---

## What it does

Claude skills and Codex agent files serve similar purposes (telling an AI system how to behave) but are written very differently:

- **Claude `SKILL.md`** files are *knowledge briefings* — prose written for Claude to read and internalize, explaining what to do and why
- **Codex `AGENTS.md`** files are *behavioral policies* — imperative directives telling an autonomous agent what commands to run, what rules to follow, and what it's prohibited from doing

This skill handles the translation between them, accounting for the fact that the conversion is inherently asymmetric: SKILL→Codex strips reasoning and surfaces concrete commands; Codex→SKILL adds context, wraps commands in explanation, and synthesizes trigger language.

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

## Usage

### SKILL.md → AGENTS.md

Paste or upload your `SKILL.md` and ask Claude to convert it. Claude will:

1. Strip the YAML frontmatter and use `name`/`description` as the Codex header
2. Rewrite instructional prose into imperative directives
3. Surface concrete shell commands for any workflow steps
4. Add Environment, Testing, Rules, and Prohibited Actions sections
5. Drop pure-knowledge content that doesn't translate to agent policy
6. Flag anything that was lossy or approximated

### AGENTS.md → SKILL.md

Paste or upload your `AGENTS.md` and ask Claude to convert it. Claude will:

1. Generate YAML frontmatter with a trigger-optimized `description`
2. Rewrite imperative rules into reasoned guidance (adding *why* where inferable)
3. Wrap shell commands in explanatory context
4. Expand terse rules into principles
5. Flag directory-scoped sections or environment-specific commands that don't translate cleanly

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
└── references/
    └── format-guide.md           # Deep reference: annotated examples, edge cases, idioms
```

The `format-guide.md` reference contains fully annotated examples of both file formats, a complete edge-case guide, and notes on the structural conventions each system expects.

---

## Installation

Install `skill-converter.skill` through your Claude skills interface. No additional dependencies required.
