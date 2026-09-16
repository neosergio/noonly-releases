# Noonly Worklog Format

Status: Draft. v0.1 below, extended by the v0.2 thread markers and the v0.3 project markers,
which Noonly implements.

This document defines the initial worklog format understood by Noonly.

The format is intentionally minimal.

The goal is to keep worklogs:

- human-readable,
- Git-friendly,
- easy to edit manually,
- easy to record work into,
- simple to parse,
- independent from Noonly.

---

# Design principle

A worklog records what happened.

The user records facts and events. Noonly derives useful state from that history whenever reasonably possible.

Prefer:

```markdown
### 2026-09-04

- Finished ENG-15813 and shared the branch in Slack.
- Waiting for review.
```

over manually maintaining several representations of the same information:

```markdown
## Current status

ENG-15813 waiting for review.

## Pending

- ENG-15813 review.

## Activity

ENG-15813 finished today.
```

Dynamic state should be derived whenever reasonably possible.

The worklog should capture history, not require the user to continuously synchronize summaries of that history.

---

# Workspace

A worklog is a directory containing Markdown files.

It may also be a Git repository.

Example:

```text
worklog/
├── example-client/
│   └── atlas.md
├── beacon.md
├── personal/
│   ├── api.md
│   └── web.md
├── IDEAS.md
└── MAQUINAS.md
```

Noonly should not initially require a specific directory hierarchy.

Nested directories are allowed.

Not every Markdown file inside a workspace is necessarily a Noonly project.

---

# Project file

A minimal project file looks like:

```markdown
# Atlas

Marketplace application for discovering, pricing and delivering datasets.

## Context

- Client: Example Client
- Employer: Example Employer
- Repository: https://example.com/repository
- Issue tracker: https://example.com/issues
- Stack: Django, Python, PostgreSQL, AWS

## Log

### 2026-09-04

- Finished ENG-15813 and shared the branch for review.
- Waiting for review.

### 2026-09-03

- Started ENG-15813.
```

---

# Project identity

The first level-one heading represents the project name.

```markdown
# Atlas
```

A project should have one clear identity.

For v0.1, a writable project file must contain exactly one level-one heading.

---

# Description

Free text immediately after the project title may describe the project.

Example:

```markdown
# Atlas

Marketplace application for discovering, pricing and delivering datasets.
```

The description is stable project context.

The description is optional.

---

# Context

Stable project information may be stored under:

```markdown
## Context
```

Example:

```markdown
## Context

- Client: Example Client
- Employer: Example Employer
- Repository: https://example.com
- Issue tracker: https://example.com/issues
- Stack: Django, Python, PostgreSQL, AWS
```

Context is intentionally flexible.

Noonly should not require every field.

Unknown context fields should be preserved rather than discarded.

The absence of a `## Context` section does not prevent a file from being recognized as a project.

---

# Context parsing

For v0.1, structured context fields are direct list items under the exact:

```markdown
## Context
```

heading.

Example:

```markdown
- Client: Acme
- Employer: Example Consulting
```

Noonly may interpret the text before the first `:` as a field name and the remaining text as its value.

Unknown fields must be preserved.

Noonly must not require a predefined list of context fields in v0.1.

Duplicate `## Context` sections make the file invalid for automatic structured writes.

---

# Project discovery

Project discovery and write validation are separate concepts.

A file may be recognized as a project while still being considered unsafe for automatic writes.

## Discovery

A Markdown file is recognized as a Noonly project when:

1. it contains a level-one heading representing the project name;
2. it contains a `## Log` section.

Files that do not satisfy these conditions are ignored as Noonly projects.

Examples of files that may exist in a workspace but are not necessarily projects:

- `README.md`
- `IDEAS.md`
- `MAQUINAS.md`
- documentation files

Noonly must not modify files that were not recognized as projects.

## Write validation

A discovered project is considered writable by Noonly only when its structure is unambiguous according to the supported v0.1 format.

For automatic structured writes, the file must contain:

- exactly one level-one project heading;
- exactly one `## Log` section;
- zero or one `## Context` sections;
- no structural ambiguity that prevents Noonly from determining the correct insertion location.

A discovered project that fails write validation may still be displayed and read when safely possible.

It must be treated as read-only for structured writes.

---

# Log

Work history lives under:

```markdown
## Log
```

The log is event-oriented.

The preferred presentation is newest-first because recent work is normally the most useful when opening the file manually.

Example:

```markdown
## Log

### 2026-09-04

- Finished ENG-15813.
- Shared the branch in Slack.
- Waiting for review.

### 2026-09-03

- Started ENG-15813.
```

Noonly should not require users to manually maintain derived sections after adding an event.

Although the work history is conceptually append-oriented, the newest-first presentation means that Noonly normally performs a targeted insertion near the beginning of the `## Log` section rather than appending bytes to the end of the file.

---

# Dates

The initial date format is:

```text
YYYY-MM-DD
```

Example:

```markdown
### 2026-09-04
```

Multiple WorkEvents may exist under the same date.

Dates represent calendar days, not precise timestamps.

Support for timestamps may be introduced later if real usage demonstrates a need.

Do not introduce timestamps prematurely.

---

# Work events

A WorkEvent represents one thing the user chose to record about their work.

Examples:

```markdown
- Started ENG-15813.

- Finished ENG-15813 and shared the branch in Slack.

- Mariana asked me to review ENG-15921 tomorrow.

- ENG-15813 is waiting for review.

- Investigated the SignalR reconnection problem.

- Decided not to upgrade React 19 because the previous versions are currently more stable.
```

The original event text is canonical.

Noonly may derive structured information from a WorkEvent, but derived information must not replace or silently rewrite the original event.

---

# Event boundary

In worklog format v0.1, one entry explicitly saved by the user represents exactly one WorkEvent.

One WorkEvent is stored as one direct Markdown unordered list item.

Example:

```markdown
- Finished ENG-15813 and shared the branch in Slack. Waiting for review.
```

Noonly must not automatically split one user entry into multiple WorkEvents.

Similarly, Noonly must not automatically combine separately recorded WorkEvents into a single canonical event.

---

# Direct list items

For v0.1, a structured WorkEvent is a top-level unordered list item directly inside a dated log section.

Example:

```markdown
### 2026-09-04

- Reviewed ENG-123.
- Started ENG-124.
```

These represent two WorkEvents.

Nested list items are not separate WorkEvents in v0.1.

For example:

```markdown
### 2026-09-04

- Reviewed ENG-123:
  - tests pass;
  - one issue remains.
```

This represents one WorkEvent.

The nested content belongs to the parent event.

Noonly should preserve nested content even when it does not interpret that content structurally.

---

# Supported log structure

For worklog v0.1, structured WorkEvents are only direct unordered list items located under:

```markdown
### YYYY-MM-DD
```

inside the exact:

```markdown
## Log
```

section.

Noonly only interprets date headings matching the supported `YYYY-MM-DD` format as structured log dates.

Other Markdown content must be preserved but is not required to be interpreted structurally.

Noonly is not intended to be a general-purpose Markdown semantic parser.

---

# Event insertion

The preferred log presentation is newest-first.

When Noonly records a new WorkEvent:

- if a `### YYYY-MM-DD` section for the current date already exists, insert the new WorkEvent as the first direct WorkEvent under that date;
- otherwise, create a new `### YYYY-MM-DD` section immediately after `## Log` and place the WorkEvent inside it.

Example:

Before:

```markdown
## Log

### 2026-09-03

- Started ENG-15813.
```

After recording an event on September 4:

```markdown
## Log

### 2026-09-04

- Finished ENG-15813.

### 2026-09-03

- Started ENG-15813.
```

Noonly must not rewrite or reorder older dated sections merely to insert a new WorkEvent.

Noonly must not reorder existing WorkEvents.

## Back-dated events

The rule above is written for the common case, where the new WorkEvent is the most recent one.

It is wrong for a back-dated event. Placing a `### 2026-09-01` section immediately after `## Log`
in a newest-first log puts it above `### 2026-09-10`, which breaks the presentation the format
asks for, and does so silently.

Noonly therefore refines the rule:

- if a `### YYYY-MM-DD` section for the recorded date already exists, insert the new WorkEvent as
  the first direct WorkEvent under that date;
- otherwise, create the new `### YYYY-MM-DD` section immediately before the first existing dated
  section that is older than it;
- if no existing dated section is older, create it at the end of the `## Log` section.

For a log that is already newest-first, this is identical to inserting immediately after `## Log`
whenever the event is the newest one.

Existing dated sections are still never rewritten or reordered. Only the position of the newly
created section is chosen.

---

# Event text preservation

The text entered by the user is canonical.

When persisting a WorkEvent, Noonly must preserve the user's original event text exactly except for the minimal Markdown escaping, indentation, or line handling required to store it safely as a WorkEvent.

For example, if the user records:

```text
Mariana asked me to review ENG-15921 tomorrow.
```

Noonly must not silently replace the canonical text with something such as:

```text
Review task ENG-15921 assigned by Mariana, due tomorrow.
```

The latter may exist later as derived structured information, but it is not a replacement for the original event.

AI-generated rewrites, summaries, classifications, or interpretations must not silently modify canonical WorkEvents.

---

# Timeline ordering

Worklog v0.1 stores dates but not timestamps.

The global timeline is therefore date-grouped rather than strictly chronological within a day.

Within the same project and date, source order is preserved.

Across different projects on the same date, v0.1 does not define a meaningful chronological order.

The UI may use a deterministic presentation order, such as:

1. date descending;
2. project name;
3. source order within the project.

This presentation order must not be interpreted as the actual chronological order in which events occurred.

Timestamps may be introduced later if real usage demonstrates a need.

---

# Derived concepts

From WorkEvents, Noonly may eventually derive concepts such as:

```text
Task
Decision
Blocker
Completion
Waiting
Assignment
Investigation
Review
```

These classifications are not required to appear explicitly in the Markdown v0.1 format.

For example:

```markdown
- ENG-15813 is waiting for review.
```

may eventually produce derived information such as:

```text
entity: ENG-15813
state: waiting
waiting_for: review
```

without changing the original WorkEvent.

Derived information is not canonical unless the worklog format explicitly defines it as such in a future version.

---

# Pending work

A separate manually synchronized `Pending` section is not required by the v0.1 format.

Pending work should eventually be reconstructed from WorkEvents whenever reasonably possible.

For example:

```markdown
### 2026-09-03

- Mariana asked me to review ENG-15921 tomorrow.
```

followed later by:

```markdown
### 2026-09-05

- Finished the review of ENG-15921.
```

may eventually allow Noonly to determine that the work is no longer pending.

However, reliable task reconstruction from natural-language events is still an assumption.

It must be validated against real worklogs before the format becomes v1.

If deterministic reconstruction proves unreliable, explicit task syntax may be introduced in a future format version.

---

# Decisions

A separate manually synchronized `Decisions` section is not required.

Decisions may be recorded directly in the work history.

Example:

```markdown
### 2026-09-04

- Decided to keep PostgreSQL instead of introducing DynamoDB because the current access patterns are relational.
```

Noonly may later identify this WorkEvent as a decision.

The original event remains canonical.

---

# Current state

The worklog should not require a manually maintained current-state summary.

Noonly should eventually derive or generate current state from recent history.

For example, a project may have:

```markdown
### 2026-09-04

- ENG-15813 is waiting for review.
```

Noonly may display:

```text
Waiting for review on ENG-15813
```

as derived state.

Generated summaries are derived data and should not automatically become canonical worklog content.

---

# Metrics

Metrics are derived.

Do not store generated metrics inside project Markdown files.

Examples:

- days since last activity;
- events this week;
- active projects;
- pending work;
- blockers;
- waiting items.

Derived metrics should be rebuildable from canonical worklog data.

---

# Human editing

A user must always be able to edit the Markdown manually.

Noonly must avoid generating obscure syntax that makes the file unpleasant to read.

Machine convenience must not dominate human readability.

A valid Noonly worklog should remain useful when opened with:

- a plain text editor;
- VS Code;
- Vim;
- GitHub;
- any Markdown viewer.

Noonly-specific functionality must not make ordinary Markdown editing impractical.

---

# Unknown content

Noonly should be conservative with content it does not understand.

It must not delete unknown sections or content merely because they are outside the current specification.

This is especially important when writing back to existing files.

Unknown content should be treated as user-owned content.

When Noonly cannot safely determine how to modify a file without affecting unknown content, it should prefer refusing the structured write over guessing.

---

# Write safety

User worklogs are valuable user data.

Correctness and preservation take precedence over convenience.

Noonly must:

- make targeted edits rather than rewriting complete files whenever practical;
- preserve unrelated Markdown;
- preserve unknown content;
- preserve existing WorkEvents;
- detect external file changes before saving;
- avoid overwriting newer external changes;
- use safe or atomic writes where practical;
- preserve the existing newline style and encoding where feasible;
- never automatically repair malformed Markdown;
- never silently normalize the entire document;
- never create a missing `## Log` section without explicit user confirmation;
- never rewrite canonical WorkEvent text using AI-generated or derived text.

A file that is ambiguous or malformed may remain readable when safe to parse.

It must be treated as read-only for structured writes when Noonly cannot determine a safe targeted edit.

Noonly should surface a clear diagnostic explaining why the file cannot currently be modified.

---

# External changes

A worklog may be edited outside Noonly.

Examples include:

- another text editor;
- Git operations;
- another computer;
- another collaborator;
- scripts.

Noonly must assume that files can change after they have been loaded.

Before performing a structured write, Noonly should verify that the underlying file has not changed in a way that would invalidate the planned edit.

If a conflicting external change is detected, Noonly should stop the write and reload or ask the user to resolve the situation rather than overwriting the external change.

---

# Migration

Existing worklogs may contain sections such as:

```text
Estado actual
Decisiones clave
Pendientes
Links
Bitácora
```

These files may contain duplicated or derived information that predates the Noonly worklog format.

Noonly must not automatically destroy, rewrite, or normalize legacy worklogs.

Migration from legacy worklogs must be a separate and explicit operation.

A migration process should prioritize preservation of the original historical information.

The initial implementation should use synthetic fixtures for the new format.

Real worklogs should later be used manually to validate whether the proposed format is sufficient before automatic migration functionality is designed.

---

# Format evolution

The worklog format is currently experimental.

Version v0.1 intentionally avoids solving every future requirement.

Changes to the format should be driven by:

1. real usage;
2. limitations discovered during implementation;
3. limitations discovered when testing against real worklogs;
4. concrete product requirements.

Do not add syntax solely because it may theoretically be useful later.

Backward compatibility becomes increasingly important as users accumulate work history.

---

# v0.2 — closing an open item

Status: **implemented**. Noonly reads these markers and writes them from the composer, and it
keeps reading v0.1 files unchanged: a file with no markers is a valid worklog in which nothing
is open.

## The problem this solves

v0.1 assumed pending work could be reconstructed from natural-language events. The *Pending
work* section above flags that as an assumption to validate before v1. It was measured, and
it does not hold. Two findings, both in `Research/README.md`:

1. **Priority cannot be recovered from event text.** Apple's on-device model, given 67
   pending items whose priority the author had assigned by hand, scored a Cohen's kappa of
   0.03–0.04 across two prompts — no agreement beyond chance. Priority recorded a decision
   *about* an item; it was never a property of the text, so nothing recovers it.

2. **A closure has never been recorded at all.** The legacy sections held 69 open items and
   zero completed ones: finished work was deleted from the list rather than marked done. No
   worklog in either format contains the fact that something ended.

The second finding is the important one. "What is still open" is not hard to derive — it is
impossible, because the closing half of every pair was never written down. No amount of
inference fixes an absent fact.

## What the format must not do

Reintroducing a manually synchronised `## Pending` section is rejected: it is exactly what
this format was built to remove, and the deletion habit it encouraged is what destroyed the
history in the first place.

Editing an existing event to mark it done is also rejected. It would rewrite canonical text,
break the append-only property, and lose the date the work actually ended — repeating the
original mistake in a smaller form.

Deriving state with an AI model is rejected as a source of truth, per *AI* in `PRODUCT.md`.

## The markers

Closing is an event, like everything else.

Three markers may appear in a WorkEvent's text. Each names an **anchor**, written as `^`
followed by a short identifier:

```markdown
### 2026-09-05

- Mariana me pidió revisar ENG-15921 mañana. (opens ^revisar-eng-15921)

### 2026-09-07

- Revisé ENG-15921 y dejé dos comentarios. (closes ^revisar-eng-15921)
```

- `(opens ^id)` — this event took on work that is not finished.
- `(closes ^id)` — this event finished it.
- `(drops ^id)` — this work is no longer going to happen.

`closes` and `drops` are different facts, and the distinction is the point: "I finished it"
and "it stopped mattering" are both worth remembering, and collapsing them loses the more
interesting half. Without `drops`, an open list only grows, which is how the previous format
became useless.

Both are ordinary WorkEvents. Nothing is edited, nothing is reordered, and the file still
reads as a plain chronological log. The history now contains both halves: when work was
taken on, and how it ended.

Open work is then a deterministic subtraction — anchors opened, minus anchors closed or
dropped — producing the same answer every time from the same file, with no model involved.

## Anchors are meant to be read

An anchor is written for a human first. `(closes ^revisar-eng-15921)` says what was closed
without looking anywhere else; `(closes ^k3f)` sends the reader hunting up the file. Since a
worklog must stay useful in a plain text editor, the readable form is the one the format
asks for.

Noonly proposes an anchor from the event's own text when the user marks an event as opening
work — lowercased, ASCII, hyphenated, trimmed to a few words, with a numeric suffix if the
file already uses it. The user never has to invent one, and can always edit it. An anchor
that repeats a ticket number or a name the event already contains is a good anchor.

## Identity and meaning are separate

A bare `^id` in an event is an anchor and nothing more: it makes the event addressable. Only
`opens`, `closes` and `drops` carry thread meaning.

This separation is deliberate. Overloading "has an anchor" to mean "is unfinished work"
would make it impossible to reference an event for any other reason — linking a decision to
the event that caused it, for instance — without accidentally declaring a commitment.

## Rules

- An anchor is `^` followed by 1–48 characters of `[A-Za-z0-9_-]`.
- An anchor is unique within one project file. Threads never cross projects, so a `closes`
  only ever resolves against an `opens` in the same file.
- A marker may appear anywhere in a WorkEvent's text; the end of the first line reads best.
- One event may carry several markers. Finishing one piece of work and taking on the next is
  a single recorded fact: `- Terminé la revisión; quedo esperando el merge. (closes ^revisar-eng-15921) (opens ^merge-eng-15921)`
- An event with no marker is exactly what it is in v0.1 — a recorded fact and nothing more.
  Most events will carry no marker.

## Diagnostics, never data loss

Every one of these is reported and preserved. None makes a file unwritable, and none causes
Noonly to alter what the user wrote:

- `closes` or `drops` naming an anchor that was never opened;
- the same anchor opened twice in one file;
- an anchor closed or dropped more than once — the first resolution stands;
- an anchor opened after it was closed.

Malformed markers are left alone as ordinary text. A worklog written by hand will contain
mistakes, and a format that punishes them by hiding content is worse than one that explains
them.

## Compatibility

A v0.1 file is a valid v0.2 file with zero open threads. A v0.2 file opened by a v0.1 reader
shows the markers as literal text inside the event: unusual, but readable and harmless.
Markers are plain text in a plain list item, so every editor, Git diff and Markdown viewer
keeps working.

## Deliberately unresolved

**Whether priority belongs in the format at all.** The measurement says it cannot be derived,
so keeping it means writing it by hand on every item. Whether that is worth doing is a
product question, not a format one, and the usage data argues both ways: the author's own
labelling ran 46% / 46% / 7%, which is close to a two-level distinction, and the emoji for
project status saw three uses across seventeen files before being abandoned. The format
therefore has no priority syntax. It should be added only if working without it proves the
need.

Related open questions already listed below: whether WorkEvents need unique IDs — an anchor
is a partial answer, since it makes an event addressable but only where someone chose to
write one — and whether tasks require explicit syntax, where this is the smallest
syntax that closes the gap.

---

# v0.3 — finishing a project

Status: **implemented**. It resolves open question 8, how archived projects are represented.

## The problem this solves

Projects end. A client relationship is over, a repository is handed back, a scope moves to
someone else. Nothing in v0.2 could say so, and a finished project kept the same weight as a
live one everywhere: in the list of projects, in the project picker, and in the view of which
projects have gone quiet — where the finished one looks like the most neglected thing in the
worklog, which is exactly backwards.

The real worklog had already reached for a workaround. After the migration, 27 of its project
files grew a `Estado: 🟢 activo / 🟡 pausado / ✅ terminado` line in their Context. The emoji
status abandoned in the legacy format came back on its own, which is the evidence of need
this document asks for before adding syntax.

## Why not the Context line

A status in Context is a summary synchronised by hand: the thing *Design principle* rejects.
It carries no date, so when the project ended is lost the moment the line is edited. Its key
and values are whatever the author's language and emoji habit happen to be, so reading it
would mean guessing. And changing it means editing an existing line, which the product
promises never to do.

## The markers

Finishing is an event, exactly like closing a thread:

```markdown
### 2026-09-14

- Se acabó el trabajo con el cliente; se da por cerrado el repo. (closes project)
```

- `(closes project)` — this event finished the project.
- `(reopens project)` — this event made it active again.

`project` is a reserved word, not an anchor: there is no caret. `(closes ^project)` is still
an ordinary thread named `project`, and never finishes anything. The difference is visible
in a plain text editor, which is the test every marker has to pass.

## Rules

- A project is **finished** when its most recent project marker is `closes`, and it counts as
  finished from the day of that event.
- Events are read oldest day first. Within one day, an entry nearer the top of the day is the
  newer one, because that is where new entries are written.
- An ordinary event recorded after a project finished does **not** reopen it. The final
  invoice, a handover note, a question answered months later — all of these happen to finished
  projects. Only `(reopens project)` reopens one.
- Finishing a project does **not** close or drop its open threads. Doing so would record facts
  nobody wrote. Open threads in a finished project stay open and are reported, so the author
  can close or drop each one.
- The marker may appear anywhere in an event's text; the end of the first line reads best. It
  may share an event with thread markers.

## Diagnostics, never data loss

Reported and preserved, and none makes a file unwritable:

- a finished project with open threads;
- `(closes project)` on a project that is already finished — the first one stands;
- `(reopens project)` on a project that is not finished.

## Compatibility

A v0.2 file is a valid v0.3 file in which no project is finished. A v0.3 file opened by a v0.2
reader shows the marker as literal text inside the event: readable and harmless.

Finishing is the only state this adds. The worklog also says `🟡 pausado`, and pausing is left
out until working without it proves the need.

---

# Open questions

The following decisions intentionally remain open:

1. Whether WorkEvents eventually need unique IDs.
2. Whether timestamps are necessary.
3. Whether tasks require explicit syntax.
4. Whether decisions require explicit syntax.
5. Whether project metadata should remain Markdown or use front matter.
6. Whether large logs should be split by month or year.
7. Whether a workspace needs a `.noonly` configuration file.
8. ~~How archived projects should be represented.~~ Resolved by v0.3: finishing is an event.
9. How project relationships should be represented.
10. How Noonly should represent events created by collaborators.
11. Whether explicit event types should ever become part of canonical Markdown.
12. How legacy worklogs should be migrated.
13. Whether projects need stable identifiers independent of filenames and titles.

These questions should be resolved from implementation experience and real usage rather than speculation.
