---
title: Template as Dependency Pattern
status: stable
owners: [template-team]
---

# Template as Dependency Pattern

Generated projects treat the template as a **dependency** rather than duplicating its documentation.

## Problem

Traditional templates copy all docs into generated projects, creating duplication, drift, and maintenance burden across N projects.

## Solution

1. Template docs stay in the template (single source of truth)
2. Generated projects **graft** them: a pinned copy of this template's export (`.gitattributes`:
   the pattern docs and the sources they cite) in `scions/python-kit/`, at the commit the
   project was generated from (`_copier_conf.vcs_ref_hash`, in the rendered `flake.nix`)
3. Generated docs link into `scions/`, so links resolve at the version the code came from
4. The project's `flake.nix` is its own (`copier update` skips it): the python-kit-upgrade workflow runs
   `copier update` and moves the pin to the same tag; `nix run .#graft` refreshes `scions/`

With `graft_docs: false`, the same links point at GitHub at that commit instead.

## File Categories

**Never generate** (template-only):
- `docs/architecture/`, `docs/decisions/`, `docs/guides/`, `docs/reference/`

**Generate once, protect** (`_skip_if_exists`):
- `README.md`, project `knowledge-base.yaml`, `flake.nix`

**Generate once, never update or recreate** (`_skip_if_exists`, and `_exclude` on update):
- `docs/agents.md`

**Always update**:
- `pyproject.toml`, infrastructure files

## Agent Discovery Flow

1. Agent reads the project's `docs/agents.md`
2. Follows its links into `scions/python-kit/docs/` (architecture, decisions, guides)
3. Applies patterns to project-specific code

## When Not to Use

- Template is unstable with frequent breaking changes
- Projects that significantly fork the patterns

## Sources

- [Template design](template-design.md) — Copier configuration
- [copier.yml](../../copier.yml) — template configuration
- [Copier documentation](https://copier.readthedocs.io/)
