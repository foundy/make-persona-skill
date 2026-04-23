# Architecture

## Two-tier skill model

```
┌─────────────────────────────────────────────┐
│  make-persona-skill  (this repo)            │
│  Installed once via plugin marketplace.     │
│  Role: generator / updater.                 │
└──────────────────┬──────────────────────────┘
                   │ generates
                   ▼
┌─────────────────────────────────────────────┐
│  persona-{nickname}                         │
│  ~/.claude/skills/persona-{nickname}/       │
│  Role: on-demand ghostwriter.               │
└─────────────────────────────────────────────┘
```

The meta-skill is generic and reusable — any user can generate their own persona skill. The generated persona skill is specific to one user.

## Artifact split

Two outputs per generated skill, with different update cadences and roles.

| File | Role | Size | Loaded when |
|------|------|------|-------------|
| `patterns.md` | Full analysis with examples | Hundreds of lines | On demand (debug, judgment calls) |
| `style-card.md` | Distilled rules | ≤ 20 lines | Always (at draft time) |

Rationale: `patterns.md` grows over time as more corpus feeds in. Injecting all of it at every draft would waste context. The 20-line style card is the actionable summary.

## Pipeline stages

1. **Collect & filter** — source-specific exporters write to `corpus/*.md`. Filter rules exclude non-user text (AI pastes, blockquotes, code blocks, external clippings).
2. **Extract patterns** — structured analysis into 8 sections. Every claim has a corpus example.
3. **Distill style card** — 20-line rule set. Context-specific sections for JIRA, PR, Slack, email.
4. **Blind test** — hold-out validation. Regenerate N=3 samples from style card alone, diff against originals, feed deltas back into `patterns.md`.
5. **Emit SKILL.md** — from `persona-skill.template.md`, substituting nickname and bilingual trigger phrases.

## Update mode

Update is not a rebuild from scratch. It merges new corpus with existing patterns — frequency counts update, new patterns appended, contradicted entries retired. This preserves the accumulated analysis while tracking drift over time.

## Filter rationale

The most common failure mode is training on text the user didn't write:
- External articles clipped into Obsidian
- AI responses pasted into notes
- Code blocks copied from docs
- Quoted messages from others in collaborative docs

Filter rules are explicit whitelists (frontmatter tags, path prefixes) plus regex exclusion of code fences and blockquotes. When filtering is uncertain, the item is included with a marker; patterns extraction sees the marker and re-evaluates.

## Skills = slash commands

Claude Code 2026 unifies Skills and slash commands. A SKILL.md at `~/.claude/skills/persona-{nickname}/` automatically exposes `/persona-{nickname}`. The same `description` field powers natural-language auto-invocation. No separate `commands/` directory needed.

## Plugin marketplace layout

Marketplace metadata lives in `.claude-plugin/marketplace.json`. The actual skill sits at `skills/make-persona-skill/`. This split lets the repo grow later (multiple skills, commands, agents) without restructuring.
