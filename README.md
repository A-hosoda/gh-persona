# gh-persona

A GitHub CLI extension that applies **personas** to Claude Code sessions via git worktree isolation and CLAUDE.md injection.

## What it does

`gh-persona` gives Claude Code a specialized personality by:

1. Creating an isolated git worktree for the session
2. Building a custom `CLAUDE.md` = original project CLAUDE.md + persona definition
3. Injecting the persona into Claude Code's system prompt (compression-resistant)

This allows Claude Code to behave as a domain expert (e.g., systematic trader, security auditor) with consistent thinking style and knowledge context.

## Architecture

```
personas/
  investor/
    base.md                  # Domain-wide principles and thinking style
    knowledge-index.md       # What the persona knows / doesn't know
    perspectives/
      macro.md               # Specific analytical perspective
      quant.md               # Another perspective in the same domain
```

A persona is composed of:

- **Base** (`base.md`) — Core principles, thinking style, communication rules
- **Perspective** (`perspectives/*.md`) — Specific focus areas and analytical framework
- **Knowledge Index** (`knowledge-index.md`) — Declared knowledge boundaries (optional)

When activated, these are combined with the target repo's `CLAUDE.md` and written to the worktree.

## Installation

```bash
# Clone the repository
git clone https://github.com/A-hosoda/gh-persona.git

# Install as a gh extension
gh extension install .
```

Or install directly from GitHub:

```bash
gh extension install A-hosoda/gh-persona
```

### Requirements

- [GitHub CLI](https://cli.github.com/) (`gh`)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (`claude`)
- Git
- Bash
- Optional: [fzf](https://github.com/junegunn/fzf) for interactive persona selection

## Usage

### Activate a persona

```bash
# Navigate to any git repository
cd ~/your-project

# Activate a persona
gh persona investor/macro
```

This will:
- Create a worktree at `/tmp/persona-<repo>-investor-macro`
- Build `CLAUDE.md` with the persona injected
- Print the path to `cd` into

Then start Claude Code in the worktree:

```bash
cd /tmp/persona-<repo>-investor-macro && claude
```

### List available personas

```bash
gh persona list
```

### Clean up worktrees

```bash
# Remove a specific persona worktree
gh persona clean investor/macro

# Interactive cleanup (lists all persona worktrees)
gh persona clean
```

### Help

```bash
gh persona --help
```

## Configuration

### Worktree base path

By default, worktrees are created under `/tmp/`. Override with:

```bash
export GH_PERSONA_WORKTREE_BASE=/path/to/base
```

Worktree path format: `<base>-<repo>-<domain>-<perspective>`

## Creating a persona

### 1. Create the domain directory

```bash
mkdir -p personas/<domain>/perspectives
```

### 2. Write `base.md`

Define the domain-wide personality:

```markdown
# Persona: <Domain>

## Role
You are an experienced <description>. All responses should reflect this expertise.

## Core Principles
- ...

## Thinking Style
- ...

## Communication
- ...
```

### 3. Write perspective files

Create `personas/<domain>/perspectives/<name>.md`:

```markdown
# Perspective: <Name>

## Focus Areas
- ...

## Analytical Framework
- ...

## Biases (Intentional)
- ...
```

### 4. (Optional) Write `knowledge-index.md`

Declare knowledge boundaries:

```markdown
# Knowledge Index: <Domain>

## What I Know
- ...

## What I Don't Know
- ...
```

## How it works

```
┌─────────────────────┐     ┌──────────────────────┐
│   Source Repository  │     │   Persona Files       │
│                      │     │                       │
│   CLAUDE.md          │     │   base.md             │
│   (project rules)   │     │   perspectives/X.md   │
│                      │     │   knowledge-index.md  │
└─────────┬───────────┘     └──────────┬────────────┘
          │                            │
          └──────────┬─────────────────┘
                     ▼
          ┌──────────────────────┐
          │  git worktree        │
          │                      │
          │  CLAUDE.md =         │
          │    original          │
          │    + base.md         │
          │    + perspective.md  │
          │    + knowledge-index │
          │                      │
          │  $ claude             │
          └──────────────────────┘
```

Key design decisions:

- **CLAUDE.md injection** — Placed in system prompt, survives context compression (unlike skills or manual prompts)
- **Worktree isolation** — Each persona session has its own working directory, no conflicts between sessions
- **Repo-scoped paths** — Worktree paths include repo name, safe to use the same persona from different repos
- **Always rebuild** — CLAUDE.md is rebuilt on every activation, picking up persona file changes

## License

MIT
