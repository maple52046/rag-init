# rag-init

A starter kit for knowledge bases that AI coding agents read and write — the corpus behind a RAG setup, a
team wiki in a repository, or a folder of notes your agents keep up to date.

[繁體中文說明](README.zh-TW.md)

It contains:

- **`create-knowledge-rules`**, a skill that interviews you about how your team creates and uses knowledge,
  then writes knowledge-management rules for your agents: how to read, distil, write, and update knowledge.
  You describe your situation; the agent picks the methodology.
- **`git-commit`**, a standalone skill that writes Conventional Commits messages and commits safely.
- **A source archive** in [`knowledge/docs/`](knowledge/docs/README.md): the methodology sources the rules
  are built from (Diátaxis, ADR, Zettelkasten, ISO 30401, Google developer style, The Good Docs Project, and
  the Agent Skills standard).

Both skills follow the [Agent Skills](https://agentskills.io/) open standard and work in Cursor, Claude Code,
OpenAI Codex, Google Antigravity, and Gemini CLI.

## Quick start

1. Copy the skills into your project (or into your user-level skills folder, see [Tool support](#tool-support)):

   ```sh
   mkdir -p .agents/skills
   cp -R path/to/rag-init/.agents/skills/create-knowledge-rules .agents/skills/
   cp -R path/to/rag-init/.agents/skills/git-commit .agents/skills/
   ```

   For Claude Code, copy the same folders into `.claude/skills/` as well. Copy the real folders from
   `.agents/skills/`, not the thin wrappers in this repository's `.claude/skills/`.
2. Ask your agent: "Set up knowledge-management rules for this repository", or run `/create-knowledge-rules`
   (in Codex, `$create-knowledge-rules`).
3. Answer the questions. Each one comes with a suggested answer you can accept as is. Before writing
   anything, the agent shows a summary of your situation and the rules it plans to use.

You can also open this repository itself in any of the tools below; the skills are discovered automatically.

## What `create-knowledge-rules` produces

- **A rules document**, by default `docs/knowledge-governance.md`, covering reading, distilling, writing, and
  updating knowledge. Each rule names the methodology it comes from, and the document ends with the rules that
  were left out, why, and when to add them back.
- **An always-on section in `AGENTS.md`**, between `<!-- knowledge-rules:start -->` and
  `<!-- knowledge-rules:end -->`, holding only the reading rules and the checks to run before writing. Agents
  read the full rules document only when they write knowledge.

The interview covers what the knowledge base holds, who reads it and how, where knowledge comes from, writing
and review habits, language and location, the token budget, and how agents should read knowledge. You choose
one of three budgets:

| Tier | Fits | Always-on section | Rules document |
| --- | --- | --- | --- |
| Lean | One person or a small knowledge base | ≤ 14 lines, ≤ 250 tokens | ≤ 80 lines, ≤ 1,200 tokens |
| Standard (default) | A team knowledge base of moderate size | ≤ 22 lines, ≤ 400 tokens | ≤ 250 lines, ≤ 3,500 tokens |
| Full | Several maintainers or a large corpus | ≤ 32 lines, ≤ 550 tokens | ≤ 600 lines, ≤ 9,000 tokens |

A complete Full-tier result is in
[`assets/examples/`](.agents/skills/create-knowledge-rules/assets/examples/knowledge-governance.example.md).
Running the skill again updates the existing rules instead of adding a second copy.

## What `git-commit` does

`/git-commit [--auto-add] [--all] [--push] [--date <when>]` (in Codex, `$git-commit`) commits only what you
staged unless you pass `--auto-add` or `--all`, never pushes unless you pass `--push`, refuses to commit
likely secrets, and ends the message with the agent's attribution trailer. It gives commands for both POSIX
shells and PowerShell. See [`SKILL.md`](.agents/skills/git-commit/SKILL.md).

## Tool support

The skills live as real folders in `.agents/skills/`, one level deep and without symbolic links, so a clone
works the same on Linux, macOS, and Windows.

| Tool | Skills in this repository | Rules in this repository | Use the skills in your own project |
| --- | --- | --- | --- |
| [Cursor](https://cursor.com/docs/context/skills) | `.agents/skills/` | `AGENTS.md` | Copy to `.agents/skills/` or `.cursor/skills/`, or `~/.cursor/skills/` for all projects |
| [Claude Code](https://code.claude.com/docs/en/skills) | `.claude/skills/` (thin wrappers) | `CLAUDE.md`, which imports `AGENTS.md` | Copy to `.claude/skills/`, or `~/.claude/skills/` for all projects |
| [OpenAI Codex](https://developers.openai.com/codex/skills) | `.agents/skills/` | `AGENTS.md` | Copy to `.agents/skills/`, or `~/.agents/skills/` for all projects |
| [Google Antigravity](https://antigravity.google/docs/skills/) (app and CLI) | `.agents/skills/` | `AGENTS.md` | Copy to `.agents/skills/`, or `~/.gemini/config/skills/` for all projects |
| [Gemini CLI](https://geminicli.com/docs/cli/skills/) | `.agents/skills/` | `AGENTS.md` after the setting below | Copy to `.agents/skills/`, or `~/.agents/skills/` for all projects |

Notes:

- **Cursor** also reads `.claude/skills/`. When a skill exists in both places it is listed once (checked with
  Cursor CLI 2026.11.05).
- **Claude Code** does not read `.agents/skills/`, so this repository keeps a wrapper per skill in
  `.claude/skills/` that points to the real skill. `CLAUDE.md` contains `@AGENTS.md`; an import is used
  instead of a symbolic link because symbolic links break in default Git for Windows clones.
- **Antigravity** scans `.agents/skills/` one level deep only; keep each skill folder directly inside it.
- **Gemini CLI** reads `GEMINI.md` by default. To make it read `AGENTS.md`, add this to
  `.gemini/settings.json` in your project (or `~/.gemini/settings.json`):

  ```json
  {
    "context": {
      "fileName": ["AGENTS.md", "GEMINI.md"]
    }
  }
  ```

  Since June 18, 2026, Gemini CLI no longer serves individual Google accounts; Google's successor is the
  [Antigravity CLI](https://antigravity.google/docs/cli/gcli-migration/), which reads `AGENTS.md` and
  `.agents/skills/` without extra settings.

## Repository layout

```text
rag-init/
├── README.md, README.zh-TW.md
├── LICENSE                         # MIT, for this repository's own content
├── THIRD_PARTY_NOTICES.md          # upstream licenses, folder by folder
├── AGENTS.md                       # maintenance rules for this repository
├── CLAUDE.md                       # @AGENTS.md
├── .agents/skills/
│   ├── create-knowledge-rules/     # SKILL.md, references/, assets/ (templates and examples)
│   └── git-commit/                 # SKILL.md, references/conventional-commits-1.0.0.md
├── .claude/skills/<name>/SKILL.md  # Claude Code wrappers that point to .agents/skills/<name>/
└── knowledge/docs/                 # methodology source archive
```

## Licenses

This repository's own content is MIT-licensed ([`LICENSE`](LICENSE)). The source archive keeps each upstream
license, and some folders are CC BY-SA 4.0 rather than MIT; see
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md). Knowledge-Centered Service (KCS) is part of the
methodology but is not included, because its license does not allow sharing adaptations; the archive links to
the original instead.

Most notes in `knowledge/docs/` are unofficial Traditional Chinese summaries; upstream snapshots stay in their
original language.
