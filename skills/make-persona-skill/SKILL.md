---
name: make-persona-skill
description: Meta-skill that generates a persona-{nickname} skill from the user's own writing corpus (Obsidian notes, JIRA comments, PR reviews, Slack messages). Invoke when the user asks to "create a persona skill", "make-persona-skill", "generate my writing style skill", "내 스타일 스킬 만들기", "페르소나 스킬 생성", or provides a nickname plus a corpus path. The generated skill drafts text in that user's voice for copy-paste into JIRA/PR/Slack/email.
---

# make-persona-skill

## Purpose

Generate a personalized writing-style skill (`persona-{nickname}`) from a user's own text corpus. The resulting skill lives at `~/.claude/skills/persona-{nickname}/` and drafts work text (JIRA, PR, Slack, email) in that user's voice.

This skill is a **ghostwriter generator**, not an AI persona switcher. Output is draft text meant for copy-paste.

## Required inputs

When invoked, gather these from the user. Ask if missing — do not invent defaults.

1. **nickname**: short identifier for the persona (e.g., `foundy`, `alice`). Becomes `persona-{nickname}` and the `/persona-{nickname}` slash command.
2. **data sources**: one or more paths or descriptions. Examples:
   - Obsidian vault path (`~/Documents/Obsidian/vault`)
   - Exported JIRA comments, PR reviews, Slack DMs
   - Plain markdown or text files
3. **mode** (implicit): `create` (default) or `update` (if `~/.claude/skills/persona-{nickname}/` already exists).

## Pipeline

Execute steps in order. Each step writes artifacts under `~/.claude/skills/persona-{nickname}/`.

### Step 1 — Collect & filter corpus

Write filtered text to `~/.claude/skills/persona-{nickname}/corpus/{source}.md` (one file per source type).

Filtering is critical — only keep the user's own first-person writing.

**Include**
- First-person journal / daily notes
- Retrospectives, technical judgment notes
- Text the user clearly authored

**Exclude**
- External clippings, web scraps
- AI response paste-backs (patterns: `claude says:`, `gpt:`, `AI:`, output following AI-style prompts)
- Code block contents (between triple backticks)
- Blockquotes (lines starting with `>`)
- Text by non-user authors in collaborative docs

Implementation hints
- Whitelist by frontmatter tags (`type: journal`, `type: retro`)
- Whitelist by path (`Journal/`, `Projects/`, `Daily/`, `Reading/`)
- Regex strip code fences and blockquotes
- When unsure, include with a suspicion marker — later steps re-filter

Report sample count and source breakdown to the user before moving on. Do not proceed if the corpus is too small (suggest < 20 samples = insufficient; ask the user how to proceed).

### Step 2 — Extract patterns (patterns.md)

Produce `patterns.md` using the schema in `templates/patterns.template.md`. Fill every section. Use `n/a — insufficient data` when a section cannot be filled honestly.

Include concrete examples from the corpus for every claim. Anonymize names/IDs.

### Step 3 — Distill style card (style-card.md)

Produce `style-card.md` using `templates/style-card.template.md`. **Hard cap: 20 lines.** This file is injected at draft time, so keep it dense and actionable. Keep the long-form analysis in `patterns.md`.

### Step 3.5 — Blind test (validation loop)

1. Randomly set aside **N = 3** samples from the corpus before steps 2-3 begin.
2. Summarize only the topic/context of each held-out sample.
3. Using only `style-card.md`, regenerate a draft for each.
4. Diff generated vs original. Append specific deltas to `patterns.md` under `## Blind Test Feedback — {YYYY-MM-DD}`.
5. If deltas are large (core rules miss, tone off), loop back to Step 3 and re-distill.

### Step 4 — Generate SKILL.md

Produce `~/.claude/skills/persona-{nickname}/SKILL.md` from `templates/persona-skill.template.md`. Substitute `{{nickname}}`. Verify:
- `name: persona-{nickname}`
- `description` includes the nickname, bilingual trigger phrases (English + Korean), and use cases (JIRA / PR / Slack / email drafts)
- Body references `style-card.md` as always-loaded, `patterns.md` as on-demand

### Step 5 — Place files & verify

Final layout:

```
~/.claude/skills/persona-{nickname}/
├── SKILL.md
├── patterns.md
├── style-card.md
└── corpus/
    ├── obsidian.md
    ├── jira.md
    ├── slack.md
    └── github-pr.md
```

Claude Code hot-reloads skills — no session restart needed. Tell the user to verify:
- `/persona-{nickname}` appears as a slash command
- Natural-language trigger (`persona-{nickname} 스타일로…`) invokes the skill

## Update mode

When invoked as "update persona-{nickname}" with new corpus data:

1. Read existing `patterns.md`, `style-card.md`.
2. Append new corpus content to the appropriate `corpus/*.md` (do not overwrite).
3. Re-run pattern extraction. Merge with prior patterns: update frequency/confidence on existing entries, add new ones, retire entries contradicted by newer data.
4. Re-distill `style-card.md`.
5. Re-run Step 3.5 blind test.
6. Update `Metadata.Last updated` in `patterns.md`.

## Non-goals

- Do not create `commands/` directories — Skills alone surface as slash commands in Claude Code 2026.
- Do not hardcode persona content into this meta-skill — always route through `templates/`.
- Do not commit `corpus/` or any `persona-*` output back into the make-persona-skill repo.
- Do not mix outputs for different nicknames — each lives under its own directory.
