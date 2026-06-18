# Memory is not the brain

How to build a durable knowledge system for AI agents without turning every remembered fact into startup context.

## The problem

Most agent systems use the word "memory" too broadly.

They mix together at least three different things:

1. the tiny set of facts an agent needs in every session;
2. durable knowledge that should be preserved somewhere;
3. transient task state that only mattered during one conversation.

Those are not the same thing. Treating them as one bucket creates two bad outcomes.

First, the agent's startup context gets polluted. Every new session begins with old project facts, stale decisions, implementation details, personal notes, one-off reminders, and logs. Some of that information was useful once, but most of it does not deserve to shape every future answer.

Second, the actual knowledge becomes harder to use. A long memory blob is not a brain. It has weak structure, no clear ownership, poor version history, and no good place for evidence, alternatives, or follow-up links.

A better pattern is to separate memory from the brain.

## The distinction

An agent's always-loaded memory is a bootloader. It should contain only the durable context that is useful before the agent knows what task it is doing.

Examples:

- stable user preferences;
- communication style;
- routing rules;
- pointers to authoritative sources;
- rules that should apply across almost every task.

The brain is different. The brain is the durable, searchable body of knowledge. It contains the actual material: decisions, research, project notes, operations notes, meeting notes, logs, drafts, and domain knowledge.

In practical terms:

| Type | Belongs in always-loaded memory? | Belongs in the brain? |
|---|---:|---:|
| "Prefer concise, source-grounded answers" | Yes | Maybe, as policy documentation |
| "Search the knowledge vault before asking me to repeat context" | Yes | Yes |
| "Decision: use tool X for project Y" | No | Yes |
| "Project Y currently has three open bugs" | No | Yes |
| "A local service runs on this private URL" | No | Yes, if private and access-controlled |
| "This task is halfway done" | No | Usually no; keep it in the session or issue tracker |
| "Token, password, or credential" | No | No |

This mirrors how human memory works, at least loosely. We do not keep every detail in active working memory all the time. We keep a few stable habits and pointers available, then retrieve details from notes, calendars, documents, people, or tools when the situation calls for them.

Agents should work the same way.

## The operating model

The core loop is simple:

1. keep startup memory small;
2. store durable facts in ordinary documents;
3. search before asking the human to repeat context;
4. read only the notes needed for the task;
5. write back decisions, research, and project state when they become durable;
6. verify writes by reading them back;
7. run periodic memory hygiene to reroute knowledge that landed in the wrong place.

The key phrase is:

> Search broad, read narrow.

Search should be cheap and low context. Filename search, tag search, and short content snippets are usually enough to find the right area. Only after that should the agent read full notes.

This avoids the common failure mode where an agent dumps an entire knowledge base into context and then becomes worse because it is now juggling irrelevant facts.

Memory hygiene is not an afterthought. It is part of the architecture.

At the moment a fact appears, the agent may not know whether it belongs in global startup memory, a project note, a decision record, an operations note, or nowhere. The agent only has local context. It is fine for the first write to be imperfect as long as the system has a maintenance loop that reviews those writes later.

That maintenance loop should periodically inspect always-loaded memory and ask:

- Is this still true?
- Does this need to shape every future session?
- Would this be better as a note with context, evidence, and links?
- Should this be deleted because it was transient?

In other words, memory can be provisional. Hygiene turns provisional memory into a cleaner routing system: some entries stay in startup memory, some get demoted into the brain, some get compressed into pointers, and some disappear.

## Why plain Markdown works

A good brain does not need to be a database, vector store, plugin, or custom memory engine.

Plain Markdown gives you most of what you need:

- readable by humans;
- readable by agents;
- searchable by filename and content;
- easy to diff;
- easy to version in Git;
- portable across editors;
- friendly to links and frontmatter;
- simple to back up;
- durable without a special runtime.

You can use Obsidian, VS Code, a static site generator, a GitHub repo, or a folder in a synced drive. The important part is not the app. The important part is the discipline: clear routing, predictable templates, good links, and search-before-ask behavior.

## Why instructions beat memory plugins

It is tempting to build a memory plugin.

The usual pitch sounds reasonable: create a tool that automatically extracts facts, writes them to a database, retrieves them with embeddings, and injects them into future prompts. That can be useful in some systems. But for many personal and small-team agent workflows, it adds complexity before the problem is well understood.

Frontier models are now good enough to follow written operating rules. They can search files, read templates, summarize notes, update indexes, and decide whether a fact belongs in startup memory, durable notes, or nowhere. That means a lot of "memory system" work can be encoded as instructions and documents instead of code.

This has several advantages.

### Instructions are inspectable

A human can open a Markdown file and see the rules. There is no hidden extraction heuristic deciding what gets remembered.

### Instructions are easy to change

If the routing policy is wrong, edit the policy. If a template is bad, edit the template. You do not need to migrate a database or redeploy a plugin.

### Instructions preserve judgment

Not every fact should be remembered. Some facts are stale, misleading, too specific, or better represented as a real note with evidence and links. Agents can make a contextual judgment when the rule is explicit: save durable knowledge to the brain, keep only bootloader facts in memory, and never store secrets.

### Instructions keep the system portable

A folder of Markdown recipes works across agents, editors, operating systems, and hosting providers. A custom plugin often locks the workflow to one runtime.

### Instructions reduce false precision

A memory plugin can make weak inferences feel official. A written note forces the agent to say what was decided, why, when to revisit it, and where the evidence came from.

The point is not that code is bad. The point is that code should come later. Start with the practice. If the practice stabilizes and repeats often enough, then automate the boring parts.

## What belongs where

A useful routing policy is the heart of the system.

### Always-loaded memory

Use this for small facts that should influence nearly every session.

Good candidates:

- preferred tone and answer style;
- canonical knowledge locations;
- stable role or domain context;
- rules such as "search the brain before asking for stored context."

Avoid:

- project status;
- local service details;
- logs;
- one-off task progress;
- credentials or secrets;
- decisions that deserve a real note.

### Brain documents

Use the brain for durable knowledge that should be preserved but not loaded by default.

Good candidates:

- decisions and rationale;
- project dashboards;
- research notes;
- operations notes;
- meeting notes;
- source lists;
- recurring procedures;
- domain references.

### Session transcript only

Some things do not need to be remembered at all.

Good candidates:

- intermediate commands;
- failed attempts that taught nothing durable;
- temporary TODOs;
- partial reasoning that was superseded;
- information that will be stale within days.

## Templates make the brain searchable

Freeform notes are better than nothing, but standard templates make the brain much more useful.

A decision record should have predictable headings:

- status;
- context;
- decision;
- rationale;
- alternatives considered;
- consequences;
- revisit trigger;
- links.

A project note should have:

- purpose;
- current status;
- architecture or approach;
- key decisions;
- open loops;
- useful paths or repos;
- verification or last known state;
- links.

A research note should have:

- practical answer;
- scope and assumptions;
- findings;
- sources;
- risks or caveats;
- action checklist;
- revisit trigger.

These sections are not bureaucracy. They are retrieval handles. They give humans and agents predictable places to look.

## Search without context pollution

Context pollution happens when irrelevant facts enter the prompt and start steering the model.

Avoid it with a staged retrieval pattern:

1. Search filenames first.
2. Search content with targeted terms.
3. Inspect snippets, not full files.
4. Read the smallest number of likely notes.
5. Stop reading when the question is answered.
6. Cite or link the notes used.

Do not load the whole brain. Do not read every related note just because it exists. More context is not automatically better context.

A useful agent instruction is:

> Before answering from stored context, search the brain. Read only the notes needed to answer. If the search is weak, say so.

This keeps the agent grounded without turning the knowledge base into a prompt landfill.

## A minimal implementation

You can implement the pattern with a folder of Markdown files.

Suggested structure:

```text
brain/
  README.md
  decisions/
    index.md
  projects/
    index.md
  research/
    index.md
  operations/
    index.md
  templates/
    decision.md
    project.md
    research.md
    operations.md
```

Add a short operating policy in `brain/README.md`:

```markdown
# Brain operating policy

This folder is the durable source of truth.

Agents should:

1. search this folder before asking humans to repeat stored context;
2. use templates when creating new durable notes;
3. keep always-loaded memory small;
4. link new notes from the relevant index;
5. verify writes by reading notes back.
```

That is enough to start.

## Implementation example: Hermes memory plus an Obsidian vault

This section is written as an implementation recipe. An agent should be able to read it and build the pattern for its own Hermes profile.

### Target architecture

Use three layers:

| Layer | Purpose | Contains | Must not contain |
|---|---|---|---|
| Hermes memory | Startup bootloader | Stable preferences, routing rules, source-of-truth pointers | Project state, research findings, decision rationale, logs |
| Hermes skill | Operating procedure | How to search, write, route, template, and verify brain notes | Specific facts that belong in the brain |
| Obsidian vault | Durable brain | Decisions, projects, research, operations, indexes, templates | Secrets or temporary task chatter |

The skill is the missing middle layer. Memory tells the agent that the brain exists. The skill tells the agent how to operate it. The vault stores the actual knowledge.

### 1. Configure Hermes memory as a bootloader

Add only compact, broadly useful entries to Hermes memory.

Example `MEMORY` entry:

```markdown
The durable brain is an Obsidian vault at `~/path/to/brain-vault`. Load the `obsidian-brain` skill before working with stored decisions, projects, research, operations, or cross-session context. Hermes memory is only a bootloader; durable details belong in the vault.
```

Example `USER` entry:

```markdown
User prefers concise, source-grounded answers and expects durable decisions, project state, and research to be stored in the Obsidian brain rather than in always-loaded memory.
```

Do not put the vault's contents into Hermes memory. Memory should point to the brain and the skill, not copy either of them.

### 2. Create the Obsidian vault structure

Create a normal Markdown vault with stable roots and indexes:

```text
brain-vault/
  Brain/
    Brain Home.md
    Operating Principles.md
  Decisions/
    Decision Index.md
  Projects/
    Project Index.md
  Research/
    Research Index.md
  Operations/
    Operations Index.md
  Templates/
    Decision.md
    Project.md
    Research.md
    Operations.md
```

Minimum index notes:

```markdown
# Decision Index

Use this folder for durable decisions. Each decision should include status, context, decision, rationale, alternatives, consequences, revisit trigger, and links.
```

```markdown
# Project Index

Use this folder for durable project dashboards. Link active projects here.
```

```markdown
# Research Index

Use this folder for research notes worth preserving beyond one conversation. Prefer source-grounded notes with assumptions, findings, caveats, and sources.
```

```markdown
# Operations Index

Use this folder for local operating procedures, services, automation, and troubleshooting notes.
```

### 3. Create an `obsidian-brain` Hermes skill

Create a Hermes skill that encodes the operating procedure. For example:

```text
~/.hermes/skills/note-taking/obsidian-brain/SKILL.md
```

Minimal skill content:

```markdown
---
name: obsidian-brain
description: Use the Markdown/Obsidian vault as the durable brain for decisions, projects, research, operations, and cross-session context.
---

# Obsidian Brain

## Vault

`~/path/to/brain-vault`

## When to load this skill

Load this skill when the user asks about stored context, prior decisions, projects, durable research, operations notes, memory hygiene, or anything that should persist beyond the current session.

## Routing policy

- Hermes memory is a bootloader: stable preferences, routing rules, and source-of-truth pointers.
- The vault is the durable brain: decisions, projects, research, operations, indexes, and templates.
- Session transcript is for transient task progress and intermediate attempts.

## Search-before-ask workflow

1. Search filenames first.
2. Search note contents with targeted terms.
3. Read only the most relevant notes.
4. If search is weak or inconclusive, say so.
5. Do not load the whole vault.

## Write workflow

When durable knowledge is created:

1. Choose the destination root: `Decisions/`, `Projects/`, `Research/`, or `Operations/`.
2. Read the closest template from `Templates/`.
3. Fill the template using the live date.
4. Link the new or updated note from the relevant index.
5. Read the written note back before reporting success.
6. Do not promote specific facts into Hermes memory unless they are stable bootloader context.

## Maintenance workflow

Periodically review Hermes memory and classify entries as:

- `keep`: still belongs in always-loaded memory;
- `compress`: keep only a shorter pointer or routing rule;
- `move`: migrate durable detail into the vault;
- `delete`: remove because it is stale, transient, or duplicated.

Move before deleting. Verify the destination note before removing or compressing the memory entry.
```

This skill is what makes the pattern reliable. Without it, a future agent may know that a vault exists but not how to search it, where to write, when to update indexes, or how to keep startup memory clean.

### 4. Use staged retrieval during normal work

Agent procedure for answering from stored context:

```text
Input: user asks about a previous decision, project, research topic, or operation.

1. Load `obsidian-brain`.
2. Identify likely roots: Decisions, Projects, Research, Operations, Brain.
3. Search filenames with the user's terms and likely synonyms.
4. Search content only if filename search is insufficient.
5. Read the smallest set of likely notes.
6. Answer from the notes read.
7. Link or name the source notes.
8. If nothing reliable is found, say that the brain search did not find it.
```

Example:

```text
User: What did we decide about the deployment model?

Agent:
1. Load `obsidian-brain`.
2. Search `Decisions/` and `Projects/` for `deployment model`, `deploy`, and the project name if known.
3. Inspect snippets.
4. Read the matching decision note.
5. Answer with the decision, rationale, consequences, and link to the note.
```

### 5. Use write-back rules

Agent procedure for deciding where new durable knowledge goes:

| Event | Destination |
|---|---|
| A decision is made | `Decisions/` or a domain-specific decision note |
| A project changes state | Existing project note in `Projects/` |
| Research remains useful beyond the session | `Research/` |
| A local operating procedure is discovered | `Operations/` |
| A stable preference or routing rule is discovered | Hermes memory |
| A temporary task advances | Session transcript or task tracker |

Write-back algorithm:

```text
1. Decide whether the fact is durable.
2. If no, leave it in the session.
3. If yes, decide whether it must shape nearly every future session.
4. If yes, add a short declarative Hermes memory entry.
5. If no, write or update an Obsidian note using the right template.
6. Link from the relevant index.
7. Verify by reading the note back.
```

### 6. Schedule memory hygiene as a Hermes cron job

Memory hygiene should be recurring because first-pass routing is allowed to be imperfect. The agent making the original memory write only has local context. A later maintenance pass can reroute that knowledge with more distance.

Example Hermes CLI flow:

```bash
hermes cron create "0 3 * * 0"
```

Use a name such as `memory-hygiene`. The cron prompt should be self-contained because scheduled jobs run in a fresh session.

Example cron prompt:

```markdown
You are maintaining this Hermes profile's memory and Markdown brain.

Load and follow the `obsidian-brain` skill.

Goal: keep always-loaded Hermes memory small and useful, and move durable but non-global knowledge into the Obsidian vault.

Sources of truth:

- Hermes memory is only a bootloader: stable preferences, routing rules, and pointers to durable sources.
- The Obsidian vault at `~/path/to/brain-vault` is the durable brain for decisions, projects, research, and operations.

Procedure:

1. Review the current Hermes memory entries.
2. Classify each entry as one of:
   - `keep`: still belongs in always-loaded memory;
   - `compress`: keep only a shorter pointer or rule;
   - `move`: migrate the durable detail into the Obsidian vault;
   - `delete`: remove because it is stale, transient, duplicated, or no longer useful.
3. For each `move` entry:
   - search the Obsidian vault for the best existing destination;
   - if none exists, create the right note using a template from `Templates/`;
   - preserve enough context that a future agent or human can understand the fact without the old memory entry;
   - update the relevant index or home note;
   - read the written note back to verify it.
4. For each `compress` or `delete` entry, update Hermes memory only after any needed Obsidian migration has been verified.
5. Produce a short hygiene report with counts and paths changed. Do not include full migrated note contents in the report.

Rules:

- Move before deleting.
- Prefer pointers over details in Hermes memory.
- Do not turn project state, operations details, research findings, or decision rationale into always-loaded memory.
- If uncertain, keep the entry and mention the uncertainty in the report.
```

### 7. Implementation checklist

An agent implementing this for itself should finish with all of the following true:

- [ ] Hermes memory contains a pointer to the vault and the `obsidian-brain` skill.
- [ ] The vault has roots for decisions, projects, research, operations, brain policy, and templates.
- [ ] Each root has an index or home note.
- [ ] The vault has templates for decisions, projects, research, and operations.
- [ ] The `obsidian-brain` skill exists and contains routing, search, write, verification, and maintenance procedures.
- [ ] A recurring hygiene cron exists, or the operator has intentionally chosen to run hygiene manually.
- [ ] The agent has tested the workflow by searching the vault, creating or updating one note from a template, linking it from an index, and reading it back.

## When to automate

Automate only after the manual pattern is clear.

Good automation candidates:

- creating a note from a template;
- updating `updated` dates;
- adding a decision to an index;
- checking for missing frontmatter;
- scanning for obvious secrets before publishing;
- running scheduled hygiene reports.

Poor first automation candidates:

- automatic fact extraction from every conversation;
- broad embedding retrieval with no routing policy;
- silent writes into memory;
- custom plugins that hide what was remembered;
- systems that inject lots of retrieved text into every session.

The automation should support the practice, not replace the judgment.

## The deeper bet

The bet here is that agent systems should become more like literate workflows and less like opaque memory machinery.

A good agent does not need every fact in its prompt. It needs clear instructions, good tools, a searchable source of truth, and permission to use judgment. The brain should be durable and inspectable. Memory should be small and boring.

That split is what keeps the system useful as it grows.
