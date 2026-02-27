---
name: distill
description: >-
  知見ベースからペルソナを対話的に蒸留する。
  knowledge-index.md は知見ファイルから自動生成し、
  base.md と perspectives/ は対話を通じて形成する。
allowed-tools: Read, Glob, Grep, Write, Bash
model: claude-sonnet-4-6
argument-hint: "<domain> [domain/perspective]"
context: conversation
---

# Distill - Interactive Persona Generation from Knowledge Base

知見ファイル群を分析し、対話を通じてペルソナを蒸留する。

## Constants

- Knowledge DB: `~/Develop/mywork/knowledge-base/docs/knowledges/knowledge.db`
- Output: `~/Develop/mywork/claude-code/gh-persona/personas/<domain>/`

## Step 1: Parse arguments

Parse `$ARGUMENTS` to determine domain and optional perspective.

- `investor` → domain=investor, perspective=undecided (decide at Step 5)
- `investor/value` → domain=investor, perspective=value

If no arguments, ask the user for domain name via AskUserQuestion.

## Step 2: Collect knowledge files

### 2a: Load categories from DB

```bash
sqlite3 ~/Develop/mywork/knowledge-base/docs/knowledges/knowledge.db "SELECT name, description FROM categories;"
```

### 2b: Category selection

Use AskUserQuestion with `multiSelect: true` to let the user select relevant categories.

### 2c: Load entries from selected categories

```bash
sqlite3 ~/Develop/mywork/knowledge-base/docs/knowledges/knowledge.db \
  "SELECT path, summary FROM entries WHERE category IN ('cat1','cat2');"
```

### 2d: Knowledge selection

Present entries and use AskUserQuestion with `multiSelect: true` to let the user select which knowledge files to include.

### 2e: Read selected files

Read all selected knowledge files to understand their content.

## Step 3: Analyze and cluster

Analyze the selected knowledge files and present:

1. **Theme clusters** — group related knowledge into themes
2. **Key patterns** — recurring principles or approaches
3. **Tensions** — contradictions or trade-offs between entries

Present this analysis to the user. This forms the foundation for the dialogue in Step 4.

## Step 4: Interactive thinking style exploration (3-5 turns)

Engage in dialogue with the user to understand how they think about this domain.
Use AskUserQuestion for structured questions, and open-ended follow-ups as needed.

### Question patterns (use 3-5, adapt based on responses)

1. **Core principle**: 「この知見群の中で最も重視している原則は何ですか？」
   - Present the themes from Step 3 as options

2. **Contradiction resolution**: Present a specific tension found in Step 3 and ask:
   「この2つの知見は矛盾しているように見えます。どちらを優先しますか？その理由は？」

3. **Starting point**: 「分析を始めるとき、まず何から考えますか？」
   - Offer frameworks identified from the knowledge as options

4. **Communication style**: 「分析結果を伝えるとき、どういうトーンが適切ですか？」
   - Options: Direct/blunt, Balanced, Cautious/hedged, Data-first

5. **Intentional bias**: 「意図的に持っておきたい偏り（バイアス）はありますか？」
   - Derive options from patterns in the knowledge

Adapt the questions based on previous answers. Skip questions that have already been answered implicitly.

## Step 5: Determine perspective name

If perspective was not specified in arguments:

Use AskUserQuestion to propose 2-3 perspective names based on the dialogue so far.
Each name should reflect the analytical lens that emerged from the conversation.

## Step 6: Generate drafts

Generate three draft files based on all information gathered:

### 6a: `base.md`

Follow the format of existing `personas/investor/base.md`:

```markdown
# Persona: <Domain>

## Role

(Domain-level role description)

## Core Principles

(Derived from Step 4 dialogue — what the user prioritizes)

## Thinking Style

(Derived from Step 4 dialogue — how the user approaches analysis)
- 深い分析や判断が必要な場面では `/oracle` を使って知見ベースを参照し、蓄積された知見に基づいて思考する

## Communication

(Derived from Step 4 dialogue — tone and style)
```

### 6b: `perspectives/<name>.md`

Follow the format of existing `personas/investor/perspectives/macro.md`:

```markdown
# Perspective: <Name>

## Focus Areas

(Derived from selected knowledge themes)

## Analytical Framework

(Derived from Step 4 dialogue — how analysis is structured)

## Biases (Intentional)

(Derived from Step 4 dialogue — deliberate analytical biases)
```

### 6c: `knowledge-index.md`

Auto-generated from the selected knowledge files:

```markdown
# Knowledge Index: <Domain>

## What I Know

(One-line summary per selected knowledge file, organized by theme)

## What I Don't Know

(Gaps identified from the knowledge analysis — areas not covered)
```

### Present drafts

Display all three drafts to the user for review.

## Step 7: Review and refine

Enter a review loop:
- Ask the user if they want to modify any of the drafts
- Apply requested changes
- Repeat until the user is satisfied

## Step 8: Write files

Write the finalized files:

```
~/Develop/mywork/claude-code/gh-persona/personas/<domain>/base.md
~/Develop/mywork/claude-code/gh-persona/personas/<domain>/perspectives/<perspective>.md
~/Develop/mywork/claude-code/gh-persona/personas/<domain>/knowledge-index.md
```

Create directories if they don't exist:
```bash
mkdir -p ~/Develop/mywork/claude-code/gh-persona/personas/<domain>/perspectives
```

If `base.md` or `knowledge-index.md` already exists, ask the user before overwriting.

## Step 9: Report completion

```
**`/distill` 完了**

生成ファイル:
- `personas/<domain>/base.md`
- `personas/<domain>/perspectives/<perspective>.md`
- `personas/<domain>/knowledge-index.md`

ペルソナを適用するには:
  gh persona <domain>/<perspective>
```

## Rules

- Never write files without user confirmation of the drafts (Step 7)
- Always include `/oracle` usage instruction in Thinking Style section of base.md
- Follow existing persona file formats exactly (see `personas/investor/` for reference)
- Knowledge-index.md is auto-generated; base.md and perspectives/ are dialogue-driven
- Minimum 3 turns of dialogue in Step 4 before generating drafts
- If the domain already has a base.md, read it first and ask whether to update or replace
