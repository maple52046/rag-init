# rag-init

A starter kit for knowledge bases that AI coding agents read and write — the corpus behind a RAG setup, a
team wiki in a repository, or a folder of notes your agents keep up to date.

[繁體中文說明](README.zh-TW.md)

It contains:

- **A ready-to-use knowledge base**: `AGENTS.md` and [`docs/knowledge-governance.md`](docs/knowledge-governance.md)
  hold default rules for how agents read, distil, write, and update knowledge, and [`knowledge/`](knowledge/README.md)
  holds the index, taxonomy, and glossary to start from.
- **`create-knowledge-rules`**, a skill that interviews you about how your team creates and uses knowledge and
  rewrites those rules to fit. You describe your situation; the agent picks the methodology.
- **`git-commit`**, a standalone skill that writes Conventional Commits messages and commits safely.
- **The methodology sources** in [`knowledge/sources/`](knowledge/sources/README.md): Diátaxis, ADR,
  Zettelkasten, ISO 30401, Google developer style, The Good Docs Project, and the Agent Skills standard, with
  study notes.

Both skills follow the [Agent Skills](https://agentskills.io/) open standard and work in Cursor, Claude Code,
OpenAI Codex, Google Antigravity, and Gemini CLI.

## Three ways to use it

### 1. Start a new knowledge base from this template

Click **Use this template** on GitHub. You get a new repository with these files and no history, and you can
make it private. Then:

1. Open the repository in your agent and run `/create-knowledge-rules` (in Codex, `$create-knowledge-rules`).
   Answer the questions — each comes with a suggested answer — and the agent rewrites the default rules for
   your situation. You can also keep the defaults and start writing.
2. Replace this `README.md` and `README.zh-TW.md` with your own, and delete `CONTRIBUTING.md`, which is for
   rag-init itself.
3. `LICENSE` covers rag-init's own files. Choose the license for your own content; keep
   `THIRD_PARTY_NOTICES.md` and the license files inside `knowledge/sources/` for as long as you keep those
   folders.
4. Not interested in the methodology sources? Delete `knowledge/sources/` (or the folders you do not want)
   and the matching rows in `THIRD_PARTY_NOTICES.md`.

A repository created from a template does not receive later changes to rag-init. If you want them, fork the
repository instead (a fork of a public repository stays public) or add rag-init as a second remote and merge
from it; you can also update only the skills with the second way below.

### 2. Add the skills to an existing project

Copy the skill folders into your project, or into your user-level skills folder to use them everywhere (see
[Tool support](#tool-support)):

```sh
mkdir -p .agents/skills
cp -R path/to/rag-init/.agents/skills/create-knowledge-rules .agents/skills/
cp -R path/to/rag-init/.agents/skills/git-commit .agents/skills/
```

For Claude Code, copy the same folders into `.claude/skills/` as well. Copy the real folders from
`.agents/skills/`, not the thin wrappers in this repository's `.claude/skills/`. Then ask your agent to "set up
knowledge-management rules for this repository", or run `/create-knowledge-rules`.

### 3. Bootstrap from the command line

[`rag-init-cli`](https://github.com/maple52046/rag-init-cli) provides a `rag-init` binary for Linux, macOS, and
Windows that does either of the above in one command, offline, and then starts your agent to tailor the rules:

```sh
rag-init new my-knowledge-base     # a new repository: template files, git init, first commit
rag-init init                      # add rag-init to the project in the current directory
```

`init` never overwrites your files. Download the binary from the
[releases page](https://github.com/maple52046/rag-init-cli/releases) or run
`go install github.com/maple52046/rag-init-cli/cmd/rag-init@latest`; see its README for all flags.

## What `create-knowledge-rules` produces

- **A rules document**, by default `docs/knowledge-governance.md`, covering reading, distilling, writing, and
  updating knowledge. Each rule names the methodology it comes from, and the document ends with the rules that
  were left out, why, and when to add them back.
- **An always-on section in `AGENTS.md`**, between `<!-- knowledge-rules:start -->` and
  `<!-- knowledge-rules:end -->`, holding only the reading rules and the checks to run before writing. Agents
  read the full rules document only when they write knowledge.

The interview covers what the knowledge base holds, who reads it and how, where knowledge comes from, writing
and review habits, language and location, the token budget, and how agents should read knowledge. You choose
one of three budgets; the defaults in this repository use Standard:

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
├── README.md, README.zh-TW.md      # about rag-init; replace in your own repository
├── CONTRIBUTING.md                 # rules for changing rag-init itself
├── LICENSE                         # MIT, for rag-init's own content
├── THIRD_PARTY_NOTICES.md          # upstream licenses, folder by folder
├── AGENTS.md                       # default always-on knowledge rules
├── CLAUDE.md                       # @AGENTS.md
├── docs/knowledge-governance.md    # default rules document (Standard tier)
├── knowledge/
│   ├── README.md, taxonomy.md, glossary.md
│   └── sources/                    # methodology sources and study notes
├── .agents/skills/
│   ├── create-knowledge-rules/     # SKILL.md, references/, assets/ (templates and examples)
│   └── git-commit/                 # SKILL.md, references/conventional-commits-1.0.0.md
└── .claude/skills/<name>/SKILL.md  # Claude Code wrappers that point to .agents/skills/<name>/
```

## Licenses

rag-init's own content is MIT-licensed ([`LICENSE`](LICENSE)). The methodology sources keep each upstream
license, and some folders are CC BY-SA 4.0 rather than MIT; see
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md). Knowledge-Centered Service (KCS) is part of the
methodology but is not included, because its license does not allow sharing adaptations; the sources folder
links to the original instead.

Most notes in `knowledge/sources/` are unofficial Traditional Chinese summaries; upstream snapshots stay in
their original language.
