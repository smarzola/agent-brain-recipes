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
- privacy constraints;
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

Not every fact should be remembered. Some facts are sensitive, stale, misleading, or too specific. Agents can make a contextual judgment when the rule is explicit: save durable knowledge to the brain, keep only bootloader facts in memory, and avoid storing secrets or unnecessary personal data.

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
- privacy constraints;
- canonical knowledge locations;
- stable role or domain context;
- rules such as "search the brain before asking for stored context."

Avoid:

- project status;
- local service details;
- logs;
- one-off task progress;
- credentials;
- private identifiers;
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

### Never store

Some things should not be saved into memory or public notes.

Examples:

- passwords;
- API keys;
- recovery codes;
- private contact details;
- unnecessary personal identifiers;
- private URLs or internal hostnames in public repos;
- medical, legal, or financial details unless the storage location is private and intentionally chosen.

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
4. avoid storing secrets or unnecessary personal data;
5. link new notes from the relevant index;
6. verify writes by reading notes back.
```

That is enough to start.

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

## Privacy and sharing

If the brain pattern is documented in a public repository, examples must be generic.

Do not publish:

- real names;
- addresses;
- private locations;
- local usernames or home paths;
- account IDs;
- emails;
- device names;
- service URLs;
- internal hostnames;
- logs;
- screenshots containing personal data;
- medical, legal, or financial specifics;
- secrets or tokens.

Use placeholders:

- `Example User`;
- `Example Project`;
- `~/path/to/vault`;
- `https://example.com/private-service`;
- `device-001`;
- `decision-001`.

A recipe repository should teach the pattern without leaking the life that produced it.

## The deeper bet

The bet here is that agent systems should become more like literate workflows and less like opaque memory machinery.

A good agent does not need every fact in its prompt. It needs clear instructions, good tools, a searchable source of truth, and permission to use judgment. The brain should be durable and inspectable. Memory should be small and boring.

That split is what keeps the system useful as it grows.
