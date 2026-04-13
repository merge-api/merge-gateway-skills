---
name: skill-converter
description: Convert between Claude SKILL.md files and OpenAI Codex AGENTS.md files (and vice versa). Use this skill whenever the user wants to translate, convert, port, or adapt an AI skill or agent instruction file from one format to the other — e.g. "convert this skill to Codex", "turn this AGENTS.md into a Claude skill", "port my skill file", "make this work with Codex", "translate this for Claude Code". Also triggers when the user asks to make a skill "cross-compatible" or wants to run the same instructions in both systems.
---

# Skill Converter: Claude SKILL.md ↔ Codex AGENTS.md

Converts AI instruction files between two formats:
- **Claude skills** — `SKILL.md` files that brief Claude with expert knowledge and workflows
- **Codex agent files** — `AGENTS.md` files that configure autonomous Codex agents with rules, commands, and policies

Before converting, read `references/format-guide.md` for a deep reference on both formats and their idioms. The conversion mappings in this file are the core logic; the reference explains the *why*.

---

## Identify the Input

First, determine what you've been given:

**It's a Claude SKILL.md if:**
- Has YAML frontmatter with `name:` and `description:` fields
- Written as expert briefing prose ("Here's how to...", "When you encounter X...")
- Contains instructional sections for Claude's reasoning
- May reference bundled scripts or asset files

**It's a Codex AGENTS.md if:**
- Little or no frontmatter (or informal comments)
- Contains imperative directives ("Run `npm test`", "Never commit to `main`")
- Specifies environment setup, shell commands, test/lint invocations
- May have a directory hierarchy with local overrides

If the format is ambiguous, ask the user which direction they want to convert.

---

## Conversion: SKILL.md → AGENTS.md

Claude skills are **knowledge briefings**. Codex agent files are **behavioral policies**. The translation is a shift from "what Claude should know" to "what the agent should do."

### Mapping Table

| SKILL.md element | AGENTS.md equivalent |
|---|---|
| YAML `name` | H1 heading or comment header |
| YAML `description` | Opening paragraph: when/why this agent config applies |
| Instructional prose sections | Imperative rule blocks ("Always...", "Never...", "When X, do Y") |
| Step-by-step workflows | Numbered command sequences with actual shell invocations |
| "Claude should consider..." | "Before making changes, check..." |
| Conditional logic in prose | Explicit if/then blocks in plain English |
| Bundled scripts references | Inline command examples or references to repo paths |
| Edge case callouts | Explicit prohibitions or guardrails section |

### Process

1. **Strip the frontmatter** — pull `name` and `description` out; use `name` as a H1 header and `description` as an opening orientation paragraph.

2. **Rewrite prose as directives** — scan every instructional paragraph and convert it into imperative sentences. "Claude should verify the file exists before proceeding" → "Always verify the file exists before running commands."

3. **Make commands concrete** — anywhere the skill references a tool, script, or workflow step, surface the actual shell command. If the skill says "run the linter," write `npm run lint` or `ruff check .` depending on context.

4. **Add an Environment section** (if the skill implies dependencies) — list setup steps, required tools, or env vars the agent needs available.

5. **Add a Testing section** (if applicable) — specify what command(s) to run to verify work is correct.

6. **Add a Prohibitions/Guardrails section** — surface any "don't do X" guidance from the skill as an explicit list.

7. **Drop pure-knowledge content** — background explanations, conceptual context, and "why" prose that exists to educate Claude don't translate to agent policy. Summarize briefly or omit.

### Output Structure

```markdown
# [Skill Name]

[1–2 sentence orientation: when this config applies and what the agent's job is]

## Environment Setup
[Commands to set up the environment, install deps, configure tools]

## Core Workflow
[Numbered steps with real shell commands]

## Rules
- Always [X]
- Never [Y]
- When [condition], [action]

## Testing
[How to verify the work — what to run and what passing looks like]

## Prohibited Actions
- [Guardrails from the original skill]
```

---

## Conversion: AGENTS.md → SKILL.md

Codex agent files are **policies for autonomous agents**. Claude skill files are **briefings for an intelligent assistant**. The translation is a shift from "rules to follow" to "knowledge to apply."

### Mapping Table

| AGENTS.md element | SKILL.md equivalent |
|---|---|
| H1 heading | YAML `name` field |
| Opening orientation paragraph | YAML `description` (with trigger language added) |
| Imperative rules ("Always X") | Contextual guidance ("When doing X, make sure to...") |
| Shell command sequences | Workflow steps with explanation of *why* each step |
| Environment setup section | Dependencies/prerequisites callout |
| Testing commands | Verification section — what to check and why |
| Prohibited actions | "Avoid..." or "Watch out for..." guidance |
| Directory-scoped overrides | Noted as context-specific variants in the skill body |

### Process

1. **Create frontmatter** — synthesize `name` from the H1 or file context. Write `description` as a trigger-optimized summary: what Claude should do *and* the specific phrases/contexts that should load this skill. Make the description "pushy" — lean toward over-triggering rather than under.

2. **Rewrite directives as guidance** — convert imperative rules into contextual wisdom. "Never commit to main" → "Always work on a feature branch; committing directly to `main` bypasses review gates and should be avoided." Add the *why* where you can infer it.

3. **Wrap commands in context** — don't just list shell commands; explain when to run them, what they verify, and how to interpret results. Commands become examples inside explanatory prose.

4. **Expand terse rules into principles** — agent files are often very terse. Claude skills benefit from richer explanation. A one-line rule may become a short paragraph.

5. **Add a "When to use this skill" orientation** — Claude benefits from knowing the shape of problems this skill addresses. Add a short section orienting Claude to the kinds of tasks it will encounter.

6. **Preserve domain conventions** — language-specific patterns, repo conventions, and tool names should be kept as concrete as possible.

### Output Structure

```markdown
---
name: [kebab-case-name]
description: [What this skill does + when to trigger it. Include specific phrases, contexts, file types, or task patterns. Be pushy about triggering.]
---

# [Skill Name]

[Orientation: what kinds of tasks this skill addresses and what good outcomes look like]

## Context & Prerequisites
[What Claude should know about the environment, stack, or project before acting]

## Workflow
[Step-by-step guidance with commands as examples, not just directives]

## Key Principles
[The "always/never" rules, rewritten as reasoned guidance with explanations]

## Common Pitfalls
[Guardrails from the original agent file, explained with context]

## Verification
[How to check that work is correct — what to run and what to look for]
```

---

## Quality Checks

After generating the output, verify:

**For SKILL.md outputs:**
- [ ] Frontmatter has both `name` and `description`
- [ ] Description includes trigger language (phrases, contexts, task types)
- [ ] No naked shell commands without surrounding explanation
- [ ] Reads naturally as a briefing, not a rulebook
- [ ] "Why" is present alongside "what"

**For AGENTS.md outputs:**
- [ ] No YAML frontmatter (unless the user's repo convention requires it)
- [ ] Every workflow step has a concrete, runnable command
- [ ] Rules are imperative and unambiguous
- [ ] Has a testing/verification section
- [ ] File could plausibly sit at repo root or in a subdirectory

---

## Handling Lossy Conversions

Some content doesn't translate cleanly — flag these to the user rather than silently dropping them:

- **Bundled scripts** (SKILL → Codex): scripts become repo path references; tell the user they'll need to commit those scripts to the repo
- **Conceptual/educational prose** (SKILL → Codex): summarize that background was omitted; offer to include it as a comment block
- **Directory-scoped overrides** (Codex → SKILL): Claude skills don't have directory scoping; note this and suggest the user create variant sections instead
- **CI/CD or environment-specific commands** (Codex → SKILL): keep them but note they're environment-dependent

Always tell the user what was dropped or approximated, and offer to restore it in a different form.

---

## Reference Files

- `references/format-guide.md` — Deep reference on both formats: idioms, conventions, full examples, edge cases
