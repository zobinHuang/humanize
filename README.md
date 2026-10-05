# humanize

A collection of coding conventions that steer code agents toward code a careful human engineer would write. Each Markdown file covers one topic and records the rules agents most often get wrong, so they can be handed to any agent as standing instructions.

## Usage

Clone the repository, then reference the conventions you need from your project's agent instruction file (`CLAUDE.md`, `AGENTS.md`, etc.):

```markdown
@path/to/humanize/cpp-coding-conventions.md
@path/to/humanize/git-conventions.md
```

| File | Topic |
|---|---|
| [cpp-coding-conventions.md](cpp-coding-conventions.md) | C++ |
| [python-coding-conventions.md](python-coding-conventions.md) | Python |
| [git-conventions.md](git-conventions.md) | Commit messages and boundaries |
| [readme-conventions.md](readme-conventions.md) | README structure |
