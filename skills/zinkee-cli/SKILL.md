---
name: zinkee-cli
description: Quick operating guide for using the `zinkee` CLI safely and efficiently. Use when Codex needs to inspect, list, create, update, delete, upload, move, publish, or troubleshoot Zinkee resources through terminal commands, especially for schemas, records, files, displays, folders, teamspaces, comments, logs, config profiles, or JSON payload construction.
---

# Zinkee CLI

## Overview

Use this skill as a concise guide for the Zinkee CLI.
Start from the bundled command inventory, then load the operational guide only for auth,
payload, output, or shell-safety details.

## Quick Start

1. Read [references/command-inventory.md](references/command-inventory.md) to find the exact
   command path, required arguments, and supported options.
2. Read [references/operational-guide.md](references/operational-guide.md) before any write
   operation or any task that needs JSON payloads, field typing, profiles, or `jq`.
3. Prefer `zinkee --json --quiet ...` and post-process with `jq`.
4. If anything still looks unclear, inspect `zinkee --help`,
   `zinkee <group> --help`, or `zinkee <group> <command> --help`.

## Workflow

### Discover resources first

- Use `schema list` and `schema get` to identify schemas and fields before writing records.
- Remember that schema and field inputs accept UUIDs or slugs, but most record payloads and API
  responses use field UUIDs.
- Build UUID-to-slug maps from `schema get` instead of guessing field identifiers.

### Execute in machine-readable mode

- Default to `--json --quiet`.
- Use `jq`, not inline Python, for filtering and reshaping CLI output.
- Use `--data -` when the JSON payload is non-trivial or needs to come from a pipeline.
- Add `--yes` to destructive commands when the task is intentional and non-interactive.

### Respect local execution rules

If the environment where the CLI runs adds extra execution rules around `zinkee`, follow those
rules in addition to this skill.

### Escalate references only when needed

- Use [references/command-inventory.md](references/command-inventory.md) for command lookup.
- Use [references/operational-guide.md](references/operational-guide.md) for behavior and payload
  rules.
- Use the CLI's `--help` output if a bundled reference is insufficient or the installed
  CLI version looks newer than this skill snapshot.

## Resources

- [references/command-inventory.md](references/command-inventory.md): extracted command tree,
  arguments, and options bundled with the skill.
- [references/operational-guide.md](references/operational-guide.md): auth rules, shell safety,
  payload formats, and high-value workflow gotchas.
