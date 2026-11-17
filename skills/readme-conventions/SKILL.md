# README Conventions

A README tells a newcomer what the repository is and how to use it, and nothing more.

## 1. Introduction

- **Open with one paragraph of 2–3 sentences** directly under the title: what the repository is, what problem it solves, and who it is for.
- **No preamble before it.** No badges wall, table of contents, or "Welcome to …" line ahead of the introduction.

## 2. Usage

- **Show the shortest path to a working result.** Install, build, run: a few commands in a code block, each one copy-pasteable.
- **Keep it simple.** Cover the common case only; options, edge cases and internals belong in separate docs, linked rather than inlined.
- **No exhaustive reference.** Do not list every flag, config key or API; link to the source or the full documentation instead.

## 3. No Repository Tour

- **No file-by-file summary.** Do not add a table (or list) describing what each file or directory in the repository does; the tree speaks for itself.
- **No architecture explanation.** The README does not walk through the repository's structure, modules or design; that belongs in separate docs if anywhere.

```markdown
# project-name

One to three sentences on what this is, what it solves, and who it is for.

## Usage

    pip install -e .
    project-name run config.yaml

See [docs/](docs/) for configuration and advanced usage.
```
