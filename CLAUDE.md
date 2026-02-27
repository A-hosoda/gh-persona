# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is gh-persona

A `gh` CLI extension that applies **personas** to Claude Code sessions via git worktree + CLAUDE.md injection. It creates an isolated worktree, builds a composite CLAUDE.md (original + persona base + perspective + knowledge index), and drops the user into it.

## Usage

```bash
gh persona investor/macro        # Activate persona in a new worktree
gh persona list                  # List available personas
gh persona clean [name]          # Remove worktree(s)
```

### Persona generation

```bash
/distill investor                # Interactively distill a persona from knowledge base
/distill investor/value          # Distill with a pre-determined perspective name
```

`/distill` is a Claude Code skill (not a gh subcommand). It reads knowledge files from the knowledge DB, conducts an interactive dialogue to capture thinking style, and generates `base.md`, `perspectives/<name>.md`, and `knowledge-index.md`.

The script is a standalone bash executable (`gh-persona` at project root). No build step or dependencies beyond `git` and `bash`. Optional: `fzf` for interactive selection.

## Architecture

### Persona structure

```
personas/<domain>/
  base.md                    # Domain-level role, principles, thinking style
  knowledge-index.md         # What the persona knows / doesn't know
  perspectives/<name>.md     # Specific analytical lens within the domain
```

A persona is identified as `<domain>/<perspective>` (e.g. `investor/macro`).

### CLAUDE.md composition order

`build_claude_md()` concatenates in this order:
1. Original CLAUDE.md from the target repo (if exists)
2. `---` separator
3. `base.md` — domain role definition
4. `perspectives/<name>.md` — perspective-specific focus
5. `knowledge-index.md` — knowledge boundary declaration (if exists)

### Worktree lifecycle

- Created at `${GH_PERSONA_WORKTREE_BASE:-/tmp/persona}-<domain>-<perspective>`
- Branch name: `persona-<domain>-<perspective>`
- `clean` subcommand handles removal (single or interactive bulk)

## Development notes

- The script uses `set -euo pipefail` — all errors are fatal
- `SCRIPT_DIR` is resolved to the script's location, so `personas/` is always found relative to the installed extension
- Test by running `./gh-persona list` or `./gh-persona investor/macro` from within any git repo
