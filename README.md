# gh-persona

A `gh` CLI extension that applies **personas** to Claude Code sessions via git worktree + CLAUDE.md injection.

## Setup

### 1. Install as gh extension

```bash
gh extension install A-hosoda/gh-persona
```

Or link locally for development:

```bash
gh extension install .
```

### 2. Install skills (optional)

Skills are Claude Code slash commands shipped with this repo. To use them, symlink the `skills/` directory into your Claude Code config:

```bash
# Symlink each skill into ~/.claude/skills/
ln -s "$(pwd)/skills/distill" ~/.claude/skills/distill
```

Or link all skills at once:

```bash
for skill_dir in skills/*/; do
  skill_name=$(basename "$skill_dir")
  ln -sf "$(cd "$skill_dir" && pwd)" ~/.claude/skills/"$skill_name"
done
```

Verify with `claude` — the skill should appear in the `/` command list.

### 3. Prerequisites

- `git` and `bash`
- `gh` CLI
- `claude` (Claude Code) for skills
- Optional: `fzf` for interactive persona selection

## Usage

### CLI commands

```bash
gh persona investor/macro        # Activate persona in a new worktree
gh persona list                  # List available personas
gh persona clean [name]          # Remove worktree(s)
```

### Skills (Claude Code slash commands)

```
/distill investor                # Interactively distill a persona from knowledge base
/distill investor/value          # Distill with a pre-determined perspective name
```

## Persona structure

```
personas/<domain>/
  base.md                    # Domain-level role, principles, thinking style
  knowledge-index.md         # What the persona knows / doesn't know
  perspectives/<name>.md     # Specific analytical lens within the domain
```

A persona is identified as `<domain>/<perspective>` (e.g. `investor/macro`).

## How it works

1. Creates an isolated git worktree
2. Builds a composite CLAUDE.md by concatenating:
   - Original CLAUDE.md from the target repo
   - `base.md` (domain role definition)
   - `perspectives/<name>.md` (perspective-specific focus)
   - `knowledge-index.md` (knowledge boundaries)
3. Drops the user into the worktree with the persona applied

## License

MIT
