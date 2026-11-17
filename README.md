# humanize

A collection of coding conventions that steer code agents toward code a careful human engineer would write. Each skill under `skills/` covers one topic and records the rules agents most often get wrong, so it can be loaded by any agent that reads skills or handed to one as standing instructions.

## Usage

Install them as skills, which an agent loads when its task matches a skill's description:

```bash
npx skills add zobinHuang/humanize
```

Or keep them in force for every task: add the repository as a submodule of your project and reference the conventions you need from its agent instruction file (`CLAUDE.md`, `AGENTS.md`, etc.):

```bash
git submodule add https://github.com/zobinHuang/humanize.git humanize
```

```markdown
@humanize/skills/cpp-coding-conventions/SKILL.md
@humanize/skills/git-conventions/SKILL.md
```

Pull the latest conventions with `git submodule update --remote humanize`.
