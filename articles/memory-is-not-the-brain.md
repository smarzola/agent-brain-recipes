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
6. verify writes by reading them back.

The key phrase is:

> Search broad, read narrow.

Search should be cheap and low context. Filename search, tag search, and short content snippets are usually enough to find the right area. Only after that should the agent read full notes.

This avoids the common failure mode where an agent dumps an entire knowledge base into context and then becomes worse because it is now juggling irrelevant facts.

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

Here is a concrete version of the pattern using Hermes Agent and an Obsidian vault. The same structure works with any Markdown folder, but Hermes is a useful example because it has both persistent startup memory and file/search tools.

### 1. Keep Hermes memory as the bootloader

Hermes memory should contain only the facts that are useful before the agent knows the task. A good bootloader memory might look like this:

```markdown
The durable brain is an Obsidian vault at `~/path/to/brain-vault`.
Search the vault before asking the user to repeat stored project, decision, research, or operations context.
Keep Hermes memory small: store only stable preferences, routing rules, and source-of-truth pointers.
Durable facts belong in Obsidian notes, not in always-loaded memory.
Use templates from `Templates/` when creating new notes.
```

A user profile memory might contain stable preferences such as:

```markdown
User prefers concise, source-grounded answers.
User prefers durable project state and decisions to be stored in the Obsidian brain.
```

Notice what is missing: project status, service details, research findings, decision rationale, logs, and temporary TODOs. Those belong elsewhere.

### 2. Put the actual brain in Obsidian

Use a normal Obsidian vault with predictable roots:

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

The important parts are the indexes and templates. They give agents stable entry points and predictable note shapes.

### 3. Teach Hermes the workflow as instructions, not a plugin

You do not need a custom memory plugin for the first version. Put the workflow in a Hermes skill, system instruction, or project instruction. For example:

```markdown
When working with the brain:

1. Search filenames first.
2. Search note contents with targeted terms.
3. Read only the most relevant notes.
4. If creating a durable note, read the closest template from `Templates/`.
5. Fill the template with the live date and source-grounded content.
6. Link the new note from the relevant index.
7. Read the written note back before reporting success.
8. Do not promote specific project facts into Hermes memory unless they are stable bootloader context.
```

That instruction is usually enough for a frontier model with file tools. It can search, read, write, patch, and verify ordinary Markdown.

### 4. Use staged retrieval

A typical Hermes turn should look like this:

```text
User: What did we decide about the deployment model?

Agent:
1. Search `Decisions/` and `Projects/` for `deployment model`.
2. Inspect matching filenames and snippets.
3. Read the one or two relevant notes.
4. Answer from those notes and link the decision.
```

The agent should not read the whole vault. It should not paste a large decision index into context if a filename search already found the exact note.

### 5. Use write-back rules

When a conversation creates durable knowledge, Hermes should write it to the vault. Common write-backs:

| Event | Write-back |
|---|---|
| A decision is made | Create or update a note in `Decisions/` |
| A project changes state | Update the project dashboard in `Projects/` |
| Research was useful beyond the session | Create a source-grounded note in `Research/` |
| A local operating procedure is discovered | Create or update an `Operations/` note |
| A stable preference is discovered | Add a short declarative entry to Hermes memory |
| A temporary task advances | Leave it in the session or task tracker |

The rule is not "remember everything." The rule is "put durable things in the smallest useful place."

### 6. Run occasional hygiene

Over time, startup memory will still accumulate too much. A simple hygiene pass can fix that:

1. Read the current Hermes memory entries.
2. Classify each entry as `keep`, `move to Obsidian`, or `delete`.
3. For each moved entry, find or create the right Obsidian note.
4. Verify the note was written.
5. Remove or compress the Hermes memory entry.

The output should be a short report of what changed, not a dump of the migrated content.

This gives you the useful part of a memory system without adding an opaque memory system. Hermes remembers where the brain is and how to use it. Obsidian holds the actual knowledge.

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
