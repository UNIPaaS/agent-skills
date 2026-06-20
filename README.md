<picture>
  <source media="(prefers-color-scheme: dark)" srcset="./assets/unipaas-logo-white.svg">
  <img alt="Unipaas" src="./assets/unipaas-logo-black.svg" width="200">
</picture>

# Unipaas Agent Skills

Skills that teach your AI agent to build on Unipaas: the integration map, the two-step `/authorize`
flow, and the Web SDK path.

## What your agent can do

Once installed, ask your agent things like:

- "Add Unipaas checkout to my Next.js app"
- "Wire up the Unipaas Web SDK secure fields"
- "Set up the two-step Unipaas /authorize flow"

## Available skills

| Skill | What it does |
|---|---|
| `unipaas` | Build Unipaas integrations: authorization, the Web SDK, and accepting payments. |

## Install

### Claude Code

    claude plugin marketplace add UNIPaaS/agent-skills
    claude plugin install unipaas@unipaas

### Codex

    codex plugin marketplace add UNIPaaS/agent-skills

Then enable the `unipaas` plugin inside Codex.

### Gemini CLI

    gemini extensions install https://github.com/UNIPaaS/agent-skills

### Any other agent

The [`skills`](https://www.npmjs.com/package/skills) CLI installs the same skill into Cursor, GitHub
Copilot, and 20+ other agents:

    npx skills add docs.unipaas.com

### No terminal

Working in claude.ai or ChatGPT? Take the skill by hand from the
[agent setup page](https://docs.unipaas.com/agents): download the skill bundle, or paste
[the full docs corpus](https://docs.unipaas.com/llms-full.txt) into any assistant.

## What is an agent skill?

A skill is a folder of instructions your agent discovers and loads when it is relevant. The `unipaas`
skill stays in lockstep with the Unipaas developer docs, so your agent builds integrations the way the
docs describe them. See the [agent setup page](https://docs.unipaas.com/agents) for every install path.
