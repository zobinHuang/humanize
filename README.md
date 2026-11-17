# humanize

A collection of coding conventions that steer code agents toward code a careful human engineer would write. They come as one skill, `skills/humanize`, whose `SKILL.md` tells an agent which convention to read for its task; each convention beside it covers one topic and records the rules agents most often get wrong, so they can be loaded by any agent that reads skills or handed to one as standing instructions.

## Usage

Install it as a skill, which an agent loads when its task matches the skill's description:

```bash
npx skills add zobinHuang/humanize
```

Or keep them in force for every task: add the repository as a submodule of your project and reference the conventions you need from its agent instruction file (`CLAUDE.md`, `AGENTS.md`, etc.):

```bash
git submodule add https://github.com/zobinHuang/humanize.git humanize
```

```markdown
@humanize/skills/humanize/cpp-coding-conventions.md
@humanize/skills/humanize/git-conventions.md
```

Pull the latest conventions with `git submodule update --remote humanize`.
