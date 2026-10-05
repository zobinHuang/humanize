# humanize

A collection of coding conventions that steer code agents toward code a careful human engineer would write. Each Markdown file covers one topic and records the rules agents most often get wrong, so they can be handed to any agent as standing instructions.

## Usage

Add the repository as a submodule of your project:

```bash
git submodule add https://github.com/zobinHuang/humanize.git humanize
```

Then reference the conventions you need from your project's agent instruction file (`CLAUDE.md`, `AGENTS.md`, etc.):

```markdown
@humanize/cpp-coding-conventions.md
@humanize/git-conventions.md
```

Pull the latest conventions with `git submodule update --remote humanize`.
