---
name: create-knowledge-rules
description: >-
  Interview the user about how their team creates and uses knowledge, then write
  knowledge-management rules for agents: a rules document that covers reading,
  distilling, writing, and updating knowledge, plus a short always-on section in
  AGENTS.md. The agent picks the methodology (Diataxis, ADR, Zettelkasten, KCS
  ideas, ISO 30401, Google developer style) from the user's situation, so the
  user does not need to know any of it. Use when the user wants to set up or
  revise a knowledge base, RAG corpus, docs folder, or notes repository that
  agents read and write, or asks how agents should record and maintain what
  they learn. Do not use for building a retrieval or vector-search pipeline, or
  for writing one individual document.
license: MIT
compatibility: >-
  Works in any agent that can read and write files in the target repository. No
  scripts are required; Python 3 is optional for the size check.
---

# create-knowledge-rules

Produce knowledge-management rules that fit the user's situation. The user describes how knowledge is used and
maintained; you choose the methodology elements, rank the rules, cut them to the chosen token budget, and write
two files. This skill does not build retrieval infrastructure and does not migrate existing documents.

Outputs:

- A rules document, by default `docs/knowledge-governance.md`, with four parts: reading, distilling, writing,
  and updating. Each rule names the methodology it comes from. The document ends with an "Excluded rules" list.
- An always-on section in the user's `AGENTS.md`, between `<!-- knowledge-rules:start -->` and
  `<!-- knowledge-rules:end -->`: reading rules, pre-write checks, and a link to the rules document.

## Token budget tiers

Agents load the always-on section on every task and the rules document whenever they write knowledge, so every
rule costs tokens. Offer three tiers; the default is **Standard**.

| Tier | Fits | Always-on section in `AGENTS.md` | Rules document |
| --- | --- | --- | --- |
| Lean | One person or a small knowledge base | at most 14 lines and 250 tokens | at most 80 lines and 1,200 tokens |
| Standard (default) | A team knowledge base of moderate size | at most 22 lines and 400 tokens | at most 250 lines and 3,500 tokens |
| Full | Several maintainers or a large corpus | at most 32 lines and 550 tokens | at most 600 lines and 9,000 tokens |

- Line counts include the marker lines and headings. Both limits bind: long lines can stay under the line
  limit and still exceed the token limit. Estimate tokens as non-CJK characters ÷ 4 plus one per CJK character.
- Rank rules by how much the user's situation needs them (the catalog gives each rule a priority per
  situation). Keep rules from the top until the next one would exceed the limit. List every dropped rule under
  "Excluded rules" with why it was dropped and when it is worth adding back.
- [`assets/examples/knowledge-governance.example.md`](assets/examples/knowledge-governance.example.md) shows a
  complete Full-tier document. Use it as the reference for structure and wording, not as text to paste.

## Workflow

```text
- [ ] 1. Inspect the repository and infer what you can
- [ ] 2. Interview: one dimension per turn, each with a suggested answer
- [ ] 3. Map the situation to rules and rank them
- [ ] 4. Fit the rules to the token budget
- [ ] 5. Confirm a summary with the user
- [ ] 6. Write the rules document and the AGENTS.md section
- [ ] 7. Check sizes and links, then report
```

### 1. Inspect the repository

Before asking anything, read what already answers the questions:

- `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursor/rules/`, `.claude/rules/`, `.agents/rules/`: existing rules,
  and an existing `knowledge-rules` section (then this run updates it instead of adding a second one).
- Folders such as `knowledge/`, `docs/`, `notes/`, `wiki/`, `runbooks/`, `adr/` or `decisions/`: where knowledge
  lives, its language, its file types (procedures, incident notes, decision records, concept notes, copied
  vendor docs), and whether files carry frontmatter.
- Git history (`git log --format='%an' | sort | uniq -c | sort -rn | head`): one author or many.

Record each inference with its evidence. Do not ask questions the repository already answers; state the
inference instead and let the user correct it.

### 2. Interview

Read [`references/interview.md`](references/interview.md) before the first question. Rules for the interview:

- Ask about **one dimension per turn**. Every question comes with a suggested answer and a one-line reason, so
  the user can reply "use the suggestion".
- Describe concrete situations ("an incident at 2 a.m.", "a new hire's first week", "why did we pick this
  database?"), never methodology names. The user is not expected to know Diátaxis or KCS.
- Skip any dimension that step 1 or an earlier answer already settles.
- Offer reading rules as a ready-made proposal and ask only about the adjustable points.

### 3. Map the situation to rules

Use [`references/rule-catalog.md`](references/rule-catalog.md): it lists every rule with its source methodology
and its priority for each situation signal from the interview. Take each rule's highest priority across the
signals that apply. When a situation is not covered, read
[`references/methodology-map.md`](references/methodology-map.md) to pick elements and explain the choice.

### 4. Fit the budget

Apply the tier limits above. Core rules (priority `core` in the catalog) stay in every tier; if they alone
exceed a Lean limit, shorten their wording instead of dropping them. Place each kept rule in one layer only:
reading rules and pre-write checks in the always-on section, everything else in the rules document.

### 5. Confirm

Show the user, in their language, before writing anything:

1. A situation summary: content types, readers and use, sources, maintenance habits, language, locations.
2. The tier and the expected size of both outputs.
3. The main rules per part (read, distil, write, update), each with a one-line reason tied to their situation.
4. The rules you plan to exclude.

Write only after the user confirms. Apply corrections and show the changed parts again.

### 6. Write the outputs

- Fill [`assets/knowledge-governance.md.tmpl`](assets/knowledge-governance.md.tmpl) and save it at the chosen
  path (default `docs/knowledge-governance.md`) without the `.tmpl` suffix. Write in the user's chosen
  language; keep rule IDs and frontmatter keys in English so later runs can match them.
- Fill [`assets/agents-md-section.md.tmpl`](assets/agents-md-section.md.tmpl) and insert it into the root
  `AGENTS.md` (create the file if missing). If the markers already exist, replace only the text between them.
- Tool notes, when the repository uses these tools:
  - Claude Code reads `AGENTS.md` only when no `CLAUDE.md` exists. If there is a `CLAUDE.md`, make sure it
    contains the line `@AGENTS.md`; add it if not.
  - Gemini CLI reads `GEMINI.md` by default. Tell the user to add `AGENTS.md` to `context.fileName` in
    `.gemini/settings.json`, or add `@AGENTS.md` to an existing `GEMINI.md`.
  - Cursor, Codex, and Antigravity read `AGENTS.md` directly; no extra file is needed.

### 7. Check and report

- Count lines and estimate tokens for the always-on section (marker to marker) and the rules document. With
  Python 3 in a POSIX shell (for a file with the markers, only the text between them is counted):

  ```sh
  for f in AGENTS.md docs/knowledge-governance.md; do python3 -c "import re,sys;t=open(sys.argv[1],encoding='utf-8').read();m=re.search(r'<!-- knowledge-rules:start -->.*?<!-- knowledge-rules:end -->',t,re.S);t=m.group(0) if m else t;c=len(re.findall(r'[\u2e80-\u9fff\uac00-\ud7af\uff00-\uffef]',t));print(sys.argv[1],len(t.splitlines()),'lines, ~',round((len(t)-c)/4+c),'tokens')" "$f"; done
  ```

  Without Python, count the lines by hand and estimate tokens from the character count.

  Over the limit: cut the lowest-priority rules into "Excluded rules" and count again.
- Check that every relative link in both outputs resolves.
- Report: the files written or changed, the tier with both sizes against their limits, the excluded rules, and
  any tool note the user still has to act on.

## Gotchas

- Users rarely know what they want from reading rules. Lead with the proposal in
  `references/interview.md`; an open question ("how should agents read knowledge?") stalls the interview.
- A rule that sounds universal still costs tokens. "Use relative links" is cheap and often worth keeping;
  a full frontmatter schema is expensive and only pays off when the corpus is large or agents filter by it.
- Do not put the same rule in both layers. The always-on section links to the rules document instead of
  repeating it.
- Do not copy text from the methodology sources into the outputs; write the rules in your own words and link
  to the sources. Some sources (KCS, for example) do not allow redistributing adaptations.
- Re-running the skill on a repository with existing rules is an update: keep rule IDs stable, show the user a
  diff of the rules that change, and move dropped rules to "Excluded rules" rather than deleting them silently.
