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
license: MIT
compatibility: >-
  Requires git. Commands are given for a POSIX shell (bash or zsh on Linux and
  macOS, Git Bash, WSL) and for PowerShell on Windows.
---

# git-commit

Analyse the repository's changes, write a Conventional Commits message, and run `git commit`. By default only
the files the user has already staged are committed, and nothing is pushed. This skill creates one new commit:
it does not rewrite history (the only `--amend` is the hook exception in step 5), rebase, or resolve merge
conflicts.

## Invocation

```text
/git-commit [--auto-add] [--all] [--push] [--date <when>]
```

Codex invokes skills with `$`: `$git-commit [...]`. Users may also ask in plain words ("commit this and push");
map the request to the same options.

| Option | Description |
| --- | --- |
| `--auto-add` | The agent may pick **related** files and `git add` them. Without it, only files the user already staged are committed and the agent must not add files. |
| `--all` | Stage **every** change in the working tree (`git add -A`: modified, added, deleted, untracked) before committing. Broader than `--auto-add` and not selective; wins when both are given. |
| `--push` | Run `git push` after a successful commit. Without it, commit only. |
| `--date <when>` | Set the commit date, with **author date and committer date set to the same value**. `<when>` is anything git can parse, such as `'11 hours ago'` or `'2026-06-10 13:00 +0800'`; relative values are first normalised to an absolute time (step 5). |

## Workflow

```text
- [ ] 1. Gather state in parallel: git status / git diff --staged / git log
- [ ] 2. Decide the commit scope (staged only / --auto-add / --all)
- [ ] 3. Check for secrets and files that should not be versioned
- [ ] 4. Write the message (Conventional Commits + attribution trailer)
- [ ] 5. Run git commit (message from a file or HEREDOC; one date for author and committer with --date)
- [ ] 6. --push: push after a successful commit
- [ ] 7. Verify and report
```

### 1. Gather state (in parallel)

- `git status` — staged, unstaged, and untracked files.
- `git diff --staged` — the staged changes (for `--auto-add`, see step 2).
- `git log --oneline -15` — the repository's existing conventions (common types, scope names, message
  language). Follow them.

### 2. Decide the commit scope

- **Default (no `--auto-add` / `--all`)**: commit only what is staged.
  - Nothing staged: stop and tell the user to `git add` first, or to use `--auto-add` / `--all`. Do not add
    files yourself.
  - Unstaged and untracked changes are normal; leave them out of this commit.
- **`--auto-add`**: run `git diff` and look at untracked files, then `git add` the files **related** to this
  change. Leave out unrelated scratch files, build output, and logs; respect `.gitignore`.
- **`--all`**: run `git add -A` at the repository root. Not selective, but step 3 still applies.

### 3. Secrets and safety

- Do not commit files that look like secrets (`.env`, `credentials.json`, private keys, tokens). If one is
  staged, stop and warn the user.
- Do not change git config, do not use interactive flags (`-i`), and do not add `--no-verify`.

### 4. Write the message

The message must follow Conventional Commits 1.0.0. When unsure about footers or whether a change is breaking,
read the bundled specification, [`references/conventional-commits-1.0.0.md`](references/conventional-commits-1.0.0.md).

```text
<type>[(scope)][!]: <description>

[body]

[footer(s); the agent attribution trailer is the last line by default]
```

- **type** (required): a noun followed by `: `. `feat` adds a feature, `fix` fixes a bug; other common types
  are `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `style`.
- **scope** (optional): a noun in parentheses naming the affected area, such as `fix(parser):`. Reuse the
  scope names already in `git log`.
- **description** (required): a short summary right after `: ` (aim for 72 characters or fewer), in the
  imperative mood, about the change itself.
- **body** (optional): one blank line after the description; explain the motivation and impact (the why).
- **other footers** (optional): one blank line after the body; tokens use `-` instead of spaces
  (`Reviewed-by:`, `Refs: #123`).
- **breaking change**: put `!` before the `:` (`feat(api)!:`) or add an uppercase `BREAKING CHANGE: <text>`
  footer. Either is enough. Types and scopes are case-insensitive; `BREAKING CHANGE` must be uppercase.
- Do not claim that tests ran, and do not add markers the user did not ask for.

**Agent attribution (on by default).** The agent that runs the commit signs the message on its last line, with
one blank line before it even when there is no body. Omit it only when the user asks for no agent attribution.
Each agent uses its own identity only, never another agent's.

- Use `Co-authored-by` only with an address the tool's vendor documents for that tool:
  - Cursor: `Co-authored-by: Cursor <cursoragent@cursor.com>` — the address Cursor documents as its agent
    commit identity.
  - Claude Code: `Co-authored-by: Claude <noreply@anthropic.com>` — the address in Claude Code's default
    commit trailer. Use the product name, not a model version.
- Any other agent (for example OpenAI Codex, Google Antigravity, Gemini CLI) writes
  `Assisted-by: <product name>`, such as `Assisted-by: OpenAI Codex`. Do not take an email from git config,
  the user's identity, another agent's example, or a model name, and do not invent one.

### 5. Run git commit

Pass the message through a quoted HEREDOC (POSIX shell) or a message file (PowerShell) so blank lines and
footers survive intact.

**POSIX shell (bash, zsh, Git Bash, WSL):**

```bash
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<optional body>

<optional footers>
<attribution trailer; the last line unless the user opted out>
EOF
)"
```

**PowerShell (Windows):** write the message to a UTF-8 file without a byte order mark, then commit with `-F`.

```powershell
$msgFile = Join-Path ([System.IO.Path]::GetTempPath()) "git-commit-msg-$PID.txt"
$msg = @'
<type>(<scope>): <description>

<optional body>

<attribution trailer>
'@
[System.IO.File]::WriteAllText($msgFile, ($msg -replace "`r`n", "`n") + "`n", [System.Text.UTF8Encoding]::new($false))
git commit -F $msgFile
Remove-Item $msgFile
```

**`--date <when>`: one value for author and committer.** `git commit --date` sets only the author date; the
committer date comes from `GIT_COMMITTER_DATE`. Normalise `<when>` to an absolute time first, then pass the
same value to both (see Gotchas for why).

```bash
WHEN="$(date -d '<when>' '+%Y-%m-%d %H:%M:%S %z')"   # GNU date (Linux, Git Bash, WSL)
GIT_COMMITTER_DATE="$WHEN" git commit --date="$WHEN" -m "$(cat <<'EOF'
<type>(<scope>): <description>
EOF
)"
```

```powershell
$when = (Get-Date).AddHours(-11).ToString('yyyy-MM-ddTHH:mm:ssK')   # or (Get-Date '2026-06-10 13:00')
$env:GIT_COMMITTER_DATE = $when
git commit --date=$when -F $msgFile
Remove-Item Env:GIT_COMMITTER_DATE
```

This affects only the commit being created; it does not rewrite existing history.

If a pre-commit hook runs:

- The hook fails: fix the problem and create a **new** commit. Do not `--amend`.
- The hook passes but changes files: `git add` those changes, then `--amend` to fold them into this commit.

### 6. Push (only with `--push`)

- Check whether the branch tracks a remote: if it does, `git push`; if not, `git push -u origin HEAD`.
- Never force push to `main` or `master`; warn the user before any force push.

### 7. Verify and report

- Run `git status` to confirm the commit. Run `git log -1 --format=%B` to check that exactly one attribution
  trailer is present. With `--date`, run `git log -1 --pretty=fuller` and confirm AuthorDate equals CommitDate.
- Report: the commit summary, the files included (list the files you added in `--auto-add` / `--all` mode),
  the date applied (with `--date`), and whether it was pushed and to which branch.

## Gotchas

- `--date` accepts relative values (`'11 hours ago'`), but `GIT_COMMITTER_DATE` does not and fails with
  `fatal: invalid date format`. Always normalise to an absolute value first.
- macOS and BSD `date` have no `-d`. Use `date -v-11H '+%Y-%m-%d %H:%M:%S %z'` or pass an absolute time.
- PowerShell has no HEREDOC, and `$(...)` there means something else. Use the message file and `-F`.
  Windows PowerShell 5.1 writes a byte order mark with `-Encoding utf8`, which can end up in the subject
  line; that is why the example uses `UTF8Encoding($false)`.
- An unquoted heredoc (`<<EOF`) expands `$` and backticks inside the message; keep the quoted `<<'EOF'` form.
- Some tools add their own attribution trailer to commits they run. If the final message would carry the
  same trailer twice, keep one.
- A failed pre-commit hook means no commit was created; `--amend` at that point rewrites the previous,
  unrelated commit.
- Nothing to commit (nothing staged without `--auto-add` / `--all`, or no changes at all): do not create an
  empty commit. Stop and report.
- The default keeps side effects small: staged files only, no push, current time. Only `--auto-add` / `--all`,
  `--push`, and `--date` relax it.
