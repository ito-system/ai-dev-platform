# Platform Architecture

This document explains the architecture of the AI development platform.

---

## Overview

The platform is a collection of reusable AI automation assets.
It has no runtime — it is executed by Claude Code interpreting markdown skill and playbook files.

```
Playbook
  └── reads shared-context/
  └── calls skills/
        └── skills produce artifacts
  └── calls validation skills/
        └── validation verifies artifacts
```

---

## Skill system

Skills are atomic, single-responsibility instructions written in Markdown.
Claude reads a skill file and executes its steps.

Key properties:
- Small: a skill does one thing
- Stateless: a skill takes inputs and produces outputs
- Composable: skills are assembled by playbooks

Directory: `skills/`

---

## Playbook orchestration

Playbooks are higher-level workflows that call skills in sequence.
A playbook reads shared-context, calls implementation skills, then runs validation skills.

Directory: `playbooks/`

---

## Shared context system

Shared context documents contain authoritative architectural decisions.
Playbooks reference them to ensure consistency across all generated code.

Directory: `shared-context/`

---

## Prompt system

Role prompts define Claude's persona and behavior for a given context.
They are referenced at the start of playbooks and skills rather than repeated inline.

Directory: `prompts/system/`

---

## Validation system

Validation skills verify that generated artifacts meet quality and consistency standards.
Every playbook must end with validation.
A FAIL result from any validation skill must block the playbook from completing.

Directory: `skills/validation/`

---

## Configuration

The `configs/` directory contains JSON and Markdown configuration files:
- `mcp.json`: MCP server configuration for Claude Code
- `github.json`: GitHub defaults and commit conventions
- `coding-rules.md`: Universal coding standards
