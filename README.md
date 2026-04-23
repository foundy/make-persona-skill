[English](README.md) | [한국어](README.ko.md)

# make-persona-skill

A Claude Code meta-skill that generates a personalized **writing-style skill** from your own text corpus (Obsidian notes, JIRA comments, PR reviews, Slack messages).

The generated skill drafts work text — JIRA comments, PR reviews, Slack replies, email — in **your voice**, ready to copy-paste. It does not change Claude's overall persona; it is a ghostwriter you can invoke on demand.

## Why

Writing tone matters in work communication. A single "LGTM, looks good to me" sounds different coming from a senior engineer versus a junior. Most AI drafts read like AI. This skill captures *your* vocabulary, sentence structure, and register so drafts sound like you.

## How it works

Two-tier architecture:

1. **`make-persona-skill`** (this repo) — the meta-skill. Installed once.
2. **`persona-{nickname}`** — generated per user. Lives at `~/.claude/skills/persona-{nickname}/`.

Pipeline:

```
corpus  →  patterns.md  →  style-card.md  →  blind test  →  SKILL.md
(filter)   (analysis)     (20-line distill)  (validation)   (generated)
```

- `patterns.md` holds the full analysis (can grow large).
- `style-card.md` is the 20-line distilled rule set injected at draft time.
- A blind-test loop validates that regenerated samples match the original voice.

## Install

### Option 1 — Plugin marketplace (recommended)

```
/plugin marketplace add foundy/make-persona-skill
/plugin install make-persona-skill
```

### Option 2 — Local path (for development)

```
/plugin marketplace add /path/to/cloned/make-persona-skill
/plugin install make-persona-skill
```

### Option 3 — Manual clone

```bash
git clone https://github.com/foundy/make-persona-skill.git /tmp/mps
cp -r /tmp/mps/skills/make-persona-skill ~/.claude/skills/
rm -rf /tmp/mps
```

## Usage

### Create your persona skill (one-time)

In any Claude Code session:

```
Run make-persona-skill.
 - nickname: alice
 - data source: ~/Documents/Obsidian/vault
```

Claude will collect samples, filter to your own first-person writing, extract patterns, distill a style card, run a blind test, and place the generated skill at `~/.claude/skills/persona-alice/`.

### Use your persona skill

After generation, `/persona-alice` is immediately available:

```
/persona-alice Draft a JIRA comment for this ticket: <paste context>
```

Natural-language triggers also work:

```
Rewrite this Slack reply in alice's style: ...
```

### Update with new data

```
Update persona-alice. I added new PR reviews to corpus/github-pr.md.
```

The meta-skill re-runs pattern extraction, merges with prior patterns, re-distills, and re-runs the blind test.

## Repository layout

```
make-persona-skill/
├── .claude-plugin/
│   └── marketplace.json
├── skills/
│   └── make-persona-skill/
│       ├── SKILL.md
│       └── templates/
│           ├── persona-skill.template.md
│           ├── patterns.template.md
│           └── style-card.template.md
├── docs/
│   └── architecture.md
├── README.md
├── README.ko.md
├── LICENSE
└── .gitignore
```

## Data safety

- `persona-*/` directories are **generated output** and live only on your machine (`~/.claude/skills/`).
- `persona-*/corpus/` contains your raw writing samples. It never leaves your disk unless you share it.
- `.gitignore` excludes `persona-*/` and `corpus/` so you cannot accidentally commit them.
- Review `corpus/` before using the meta-skill on a machine you share.

## Design principles

1. Meta-skill (generator) and generated skill (consumer) are separate.
2. Analysis artifact (`patterns.md`) and injection artifact (`style-card.md`) are separate.
3. Filter corpus to the user's **own** first-person writing only.
4. Validate by blind test, not self-grading.
5. Skills = slash commands in Claude Code 2026 — one file gives both triggers.
6. Keep corpus inside the persona directory for easy updates.

## Contributing

Issues and PRs welcome. Keep changes scoped: the meta-skill should stay small and composable.

## License

[MIT](LICENSE)
