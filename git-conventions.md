# Git Conventions

## 1. Commit Message

A commit message is one line in Conventional Commits form, not a paragraph.

```
<type>(<scope>): <one sentence summarizing the change>
```

- **Type** is one of `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `build`, `ci`, `chore`.
- **Scope** names the module or component touched, e.g. `feat(parser): support nested blocks`.
- **Summary** is a single imperative sentence in lowercase, with no trailing period, kept under 72 characters.
- **No body by default.** Add a body only when the reason for the change is not obvious from the diff; keep it to a few lines. A bulleted changelog of every file touched is not a commit message.

```
fix(scheduler): skip empty windows when merging timelines
```

## 2. Commit Boundaries

One commit holds one logical change, and its boundary is clear from the summary line alone.

- **One concern per commit.** A feature, a bug fix, a refactor and a formatting sweep are four commits, not one. If the summary needs "and", split it.
- **Separate mechanical from semantic.** Renames, moves and reformatting go in their own commit, ahead of the behaviour change that depends on them, so the behaviour diff stays small and reviewable.
- **Each commit builds and passes tests on its own.** Do not split so finely that an intermediate commit is broken; a change and the tests that cover it belong together.
- **Do not lump unrelated work.** Stage selectively (`git add -p`) rather than committing the whole working tree when it holds more than one change.

## 3. Authorship

- **No code-agent attribution.** Never add `Co-Authored-By` trailers, "Generated with …" lines, or any other attribution to a code agent (Claude, Copilot, Codex, Cursor, etc.) in commit messages or PR descriptions.
