---
title: Copier Template Design
status: stable
owners: [template-team]
---

# Copier Template Design

This repository is a [Copier](https://copier.readthedocs.io/) template. Users run `copier copy` to generate projects from it.

## Template Variables

### User Inputs

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `project_name` | str | "My Python Project" | Human-readable name |
| `package_name` | str | (derived) | Python package name (snake_case) |
| `author_name` | str | "Your Name" | Author |
| `author_email` | str | "you@example.com" | Email (validated) |
| `python_version` | choice | "3.11" | Min Python: 3.10, 3.11, 3.12 |
| `include_example_code` | bool | true | Full working examples |
| `include_github_actions` | bool | true | CI/CD workflows |
| `include_dev_scripts` | bool | true | Setup/test scripts |
| `license` | choice | "MIT" | MIT, Apache-2.0, BSD-3-Clause, GPL-3.0, Proprietary |

### Computed

- `cli_command` — derived from `package_name`
- `module_path` — `<package_name>.cli.main`
- `current_year` — auto-generated

## What Stays Fixed

- Architecture pattern (functional services, protocols, context objects)
- Directory structure (`domain/`, `services/`, `adapters/`, `cli/`)
- Tooling choices (uv, ruff, pytest, Typer)
- Testing strategy (fakes over mocks)

## Conditional Content

Files use `{% if include_example_code %}` to include or omit working examples. When disabled, the shared
files (`__init__.py` exports, `services/context.py`, `cli/main.py`) keep only a skeleton with comments
explaining the pattern, and the example-only modules and tests are not generated at all: `copier.yml` `_exclude`
lists them under `{% if not include_example_code %}` (as docstring-only stubs, every `copier update`
recreated them in a project that had deleted them).

Conditional features:
- `include_github_actions` — `.github/workflows/` lint and test workflows
- `include_dev_scripts` — `scripts/setup.sh` and `scripts/test.sh`
- `graft_docs` — `flake.nix` (grafts python-kit's docs and knowledge-base-kit into `scions/`) and the Forgejo
  workflows: graft's `graft-check` and `graft-upgrade`, and python-kit's own `python-kit-upgrade`

## Post-Generation

Copier runs automatically after generation:
1. `uv sync` — install dependencies
2. `git init` — initialize repository
3. `git add .` && `git commit` — initial commit

## Template Updates

Generated projects can pull template updates:

```bash
copier update
```

`copier update` runs the `_migrations` in `copier.yml` for each version it crosses: commands for what
the update itself can't do, such as renaming paths in the files it skips. Each release's CHANGELOG
"Migration" section says what is left to do by hand. With `graft_docs`, the generated
`python-kit-upgrade` workflow runs the update weekly and opens a PR.

Protected files (`_skip_if_exists`, generated when missing, never overwritten): with example code,
domain/entity.py, example_service.py, in_memory_repository.py, CLI commands and unit/integration tests; always
README.md, knowledge-base.yaml, and with `graft_docs` graft's two workflows (graft-upgrade moves them) and
`flake.nix`. The project owns `flake.nix`: graft-upgrade moves graft's pins, and python-kit-upgrade moves the
python-kit pin to the tag it updated to (the v2.2.0 migration does it once, for the update that introduced
this). `docs/agents.md` is generated once: `_exclude` lists it when `_copier_operation == 'update'` (copier
9.6.0+), so an update neither changes it nor recreates it after deletion.

## Sources

- [copier.yml](../../copier.yml) — template configuration
- [Copier documentation](https://copier.readthedocs.io/)
- [Dependency pattern](dependency-pattern.md) — template-as-dependency approach
