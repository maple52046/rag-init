---
name: git-commit
description: >-
  Analyse the repository's changes, write a Conventional Commits 1.0.0 message
  that ends with the agent's attribution trailer, and run git commit. By default
  it commits only files the user already staged and does not push; flags opt in
  to staging related files, staging everything, pushing, or setting one date for
  both author and committer. Use when the user runs /git-commit (or $git-commit
  in Codex) or asks to commit their changes, commit with a generated message, or
  commit and push. Do not use for amending or rewriting existing commits,
  rebasing, resolving merge conflicts, or opening pull requests.
---

# git-commit

Claude Code adapter for the canonical skill
[`.agents/skills/git-commit/SKILL.md`](../../../.agents/skills/git-commit/SKILL.md).
Read that file and follow it. Paths inside it are relative to its own directory,
`.agents/skills/git-commit/`.
