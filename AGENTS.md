# UNIPaaS/agent-skills

Installable agent skills for building on Unipaas. This repo is both a plugin marketplace and the skill
payload; the install commands are in the README.

## Layout

Hand-maintained in this repo (the repo's presentation):

- `README.md`, `LICENSE`, `CONTRIBUTING.md`, `AGENTS.md` / `CLAUDE.md`, `assets/`.

Generated from the Unipaas docs and synced in by CI (do not hand-edit):

- `skills/unipaas/` - the skill (`SKILL.md` plus reference files), a knowledge projection of the docs
  corpus.
- `.claude-plugin/`, `.codex-plugin/`, `.agents/`, `gemini-extension.json` - the per-harness plugin and
  extension manifests.
- `.mcp.json` - reserved for a future MCP server; empty today.

The source of truth is the Unipaas developer docs (docs.unipaas.com). The skill and the manifests are
regenerated there and pushed here on each docs release, so edits to those paths are overwritten on the
next sync. Fix skill content upstream in the docs.

## Consuming the skill

Install with the README's commands, or take the skill by hand from https://docs.unipaas.com/agents.
The portable unit is `skills/unipaas/SKILL.md`.

## Conventions

Conventional Commits. The generated paths are synced by `unipaas-docs[bot]`; the hand-maintained chrome
is edited by maintainers.
