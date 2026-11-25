# Maintaining rag-init

Rules for agents and people who change this repository. They cover the repository itself, not projects that
use its skills.

## Skills

- The canonical skill is the real folder `.agents/skills/<name>/`, directly inside `.agents/skills/` (one level
  deep; Antigravity does not scan deeper). Do not use symbolic links anywhere in the repository; they break in
  default Git for Windows clones.
- Each skill has a Claude Code wrapper at `.claude/skills/<name>/SKILL.md`. Its frontmatter `name` and
  `description` are identical to the canonical skill's, and its body only points to the canonical
  `SKILL.md`. When a canonical description changes, update the wrapper in the same commit.
- Skills follow the [Agent Skills specification](https://agentskills.io/specification): frontmatter uses only
  `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`; `name` equals the folder
  name; `SKILL.md` stays under 500 lines.
- Skills are standalone: they reference only files inside their own folder and external URLs, never other
  parts of this repository, so a user can copy one folder into another project.
- Each skill sets `license: MIT` and carries a copy of the root `LICENSE`.
- Skills, rule templates, and examples are written in English.
- Template files end in `.tmpl`. Inside a skill folder, the only file named `SKILL.md` is the one at its root,
  and no file is named `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, or `*.mdc`: tools scan recursively and would
  load such files as real skills or rules.
- Validate every changed skill and wrapper with the reference validator:

  ```sh
  uvx --from skills-ref agentskills validate .agents/skills/<name>
  uvx --from skills-ref agentskills validate .claude/skills/<name>
  ```

## Source archive (`knowledge/docs/`)

- Keep upstream snapshots unchanged and in their original language. Replace a snapshot with a newer upstream
  copy rather than editing it, and record the source commit or retrieval date in the folder's `README.md`.
- Notes keep the language they were written in.
- Add material only when its license allows redistribution of the form you add (copy, summary, or
  translation). No-derivatives licenses rule out summaries and translations; link to the source instead.
- Every folder is listed in `knowledge/docs/README.md` and in `THIRD_PARTY_NOTICES.md` with its source,
  license, and the changes made. A folder whose license is not MIT carries its own license file.

## Documentation

- `README.md` (English) and `README.zh-TW.md` (Traditional Chinese) say the same thing. Change both in the same
  commit.
- Claims about how a tool discovers skills or rules link to that tool's official documentation.

## Commits

- Commit messages follow Conventional Commits 1.0.0 and are written in English. The `git-commit` skill in this
  repository does this.
- Never commit credentials, tokens, private keys, internal host names, or personal contact details.
