---
name: persona-{{nickname}}
description: Drafts work text (JIRA comments, PR reviews, Slack replies, email) in {{nickname}}'s writing style for copy-paste. Auto-invoke when the user says "in {{nickname}}'s style", "persona-{{nickname}}", "{{nickname}} 스타일로", "{{nickname}} 말투로", or asks to draft/rewrite content as {{nickname}}. Output is draft text only, not a persona switch.
---

# persona-{{nickname}}

## Purpose

Produce draft text in {{nickname}}'s writing voice. Output is meant to be copy-pasted into JIRA, GitHub PR reviews, Slack, email, and similar work channels. This skill does not change Claude's overall persona — it generates text on behalf of {{nickname}}.

## Reference files

- `style-card.md` — core style rules. **Always load before generating.**
- `patterns.md` — full pattern analysis with examples. Load only when the draft needs a judgment call that the style card does not cover.
- `corpus/` — raw source text. Consult only for debugging or when updating the skill.

## Generation procedure

1. Load `style-card.md`.
2. Identify register: JIRA comment, PR review, Slack reply, email, or general. If unclear, ask.
3. Apply core rules plus context-specific rules from the style card.
4. Run the anti-pattern check (`Never` list in style card).
5. Output the draft only. Do not narrate your reasoning.

## Length & format

Match the register defaults defined in `style-card.md` unless the user specifies otherwise. If the user supplies a draft to rewrite, preserve factual content and only adjust voice.

## Update

Re-run the `make-persona-skill` meta-skill with new corpus material. The pipeline merges and re-distills, then overwrites `style-card.md` and `SKILL.md`.
