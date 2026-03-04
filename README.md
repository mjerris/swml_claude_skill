# SWML Claude Skill

A Claude Code skill for crafting SWML (SignalWire Markup Language) documents. Enables Claude to generate voice/video call flows, IVR menus, AI agents, and telephony automation using SWML.

## What's Included

- **SKILL.md** — Core skill definition with SWML overview, method summary, and common patterns
- **cheatsheet.md** — Quick reference for parameters, branching, and examples
- **swml_reference.md** — Full SWML documentation (~5000 lines, searchable)
- **schema.json** — JSON Schema for SWML validation (~12000 lines, searchable)

## Installation

### Personal (all projects)

```bash
mkdir -p ~/.claude/skills/swml
cp -r . ~/.claude/skills/swml/
```

### Project-specific

```bash
mkdir -p .claude/skills/swml
cp -r . .claude/skills/swml/
```

## Usage

Once installed, Claude will automatically use this skill when you ask it to create or modify SWML documents. For example:

- "Create a SWML document for an IVR menu"
- "Build a SWML AI agent that handles customer support"
- "Write a SWML call flow that connects to a phone number"
