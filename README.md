# Agent brain recipes

Practical recipes for building agent-readable, human-readable knowledge systems with plain Markdown.

This repository starts from a simple idea: an agent's always-loaded memory should be tiny. It should tell the agent where to look, what the durable sources of truth are, and which operating rules matter in every session. The actual facts, decisions, research, project notes, logs, and local details should live in an external brain that agents can search and read on demand.

The first article is:

- [Memory is not the brain](articles/memory-is-not-the-brain.md)

## Principles

- Keep startup memory small.
- Store durable knowledge in ordinary documents.
- Search broadly, read narrowly.
- Prefer templates and instructions over custom memory plugins.
- Make notes useful for both humans and agents.
- Avoid storing or publishing personal data, credentials, logs, private paths, or identifiers.

## Templates

The `templates/` folder contains generic Markdown templates that can be copied into an Obsidian vault, a docs repository, or any folder of Markdown files.

- [Decision record](templates/decision.md)
- [Project note](templates/project.md)
- [Research note](templates/research.md)
- [Operations note](templates/operations.md)

## Privacy note

Examples in this repository are intentionally generic. Do not copy private vault paths, family details, account names, tokens, service URLs, device names, or other identifying information into public recipes.
