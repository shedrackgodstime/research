# Research Repository Objective

## Core Objective

This repository exists to preserve research work that would otherwise be lost,
forgotten, or left scattered across temporary notes, browser tabs, terminals,
and memory.

The main problem is not a lack of research activity. The real problem is that
useful research is being done without a consistent place to keep it. Once that
happens, the work becomes difficult to revisit, difficult to build on, and
difficult to turn into deeper learning, a project, or a public writeup.

This repository is meant to solve that problem by giving non-project research
a stable home.

## Primary Purpose

The first priority of this repository is simple:

1. Record research
2. Preserve findings
3. Make past work easy to revisit
4. Make future work easier to continue

This repository should make research cumulative instead of disposable.

## Scope

This repository is for:

- exploratory cybersecurity research
- random technical investigations
- partial findings that are still useful
- unfinished topic notes
- commands, observations, references, and ideas worth keeping
- research that is interesting but does not yet justify its own repository

This repository is especially useful for work that sits between a passing
curiosity and a full project.

## Non-Goals

This repository is not primarily:

- a portfolio repository
- a collection of polished public articles
- a replacement for dedicated project repositories

If a topic grows into a real project, it can later be moved into its own
repository with its own structure. This repository should remain focused on
preserving research that does not yet have a better home.

## Secondary Benefit

Some topics recorded here may later become strong material for the portfolio.

That is useful, but it is not the driving reason for this repository.

The portfolio is a secondary outcome. Research preservation is the main
objective.

## Structure

The repository is organized by topic. Each research topic lives in its own
directory at the root of the repository.

```text
Researches/
  INDEX.md
  objective.md
  _template/
    README.md
  topic-name/
    README.md
```

`INDEX.md` is a single table listing every topic with its status and portfolio
flag. It is the entry point for browsing the archive.

`_template/` contains the base files for a new topic. Every new topic is
created from this template.

Each topic directory contains one required file:

- `README.md` — topic summary, metadata, and any notes needed to start

Additional files are optional and should only be added when the topic grows
enough to justify them.

Examples include:

- `notes.md` for raw working notes, commands, findings, and observations
- `references.md` for source tracking
- scripts, samples, artifacts, or screenshots when needed

No deeper subdirectory structure is enforced by default.

## Topic Metadata

Every `README.md` opens with the following frontmatter:

```yaml
---
title: Topic Title
summary: One-line summary of the topic
status: draft
portfolio: false
started: YYYY-MM
tags: []
---
```

Status options: `draft` `active` `completed` `archived`

Meaning of each status:

- `draft` — the topic has been started, but the research is still early,
  sparse, or not yet well formed
- `active` — the topic is currently being researched or is likely to receive
  more work soon
- `completed` — the topic has reached a useful stopping point and the current
  findings are considered sufficient for now
- `archived` — the topic is no longer being actively worked on, but is kept as
  part of the long-term research record

`portfolio: true` means the topic is eligible to be surfaced by the portfolio
engine. `portfolio: false` means it stays in the repository but is ignored by
the portfolio engine.

## README Contract

Since `README.md` is the only required file in a topic directory, it must be
able to stand on its own even when no other files exist yet.

Every topic `README.md` should contain:

- frontmatter for metadata
- a short summary of the topic
- the current state of the research
- the main findings, questions, or working notes available so far

The `README.md` does not need to be polished. It only needs to be clear enough
that the topic can be understood and resumed later.

### Minimum Structure

The minimum expected shape is:

```md
---
title: Topic Title
summary: One-line summary of the topic
status: draft
portfolio: false
started: YYYY-MM
tags: []
---

# Topic Title

## Summary

## Current State

## Notes
```

### Section Expectations

- `Summary` explains what the topic is about and why it matters
- `Current State` says whether the research is just starting, in progress,
  blocked, or effectively done
- `Notes` holds the useful content currently available, even if it is rough

### Optional Sections

As a topic grows, the `README.md` may also include sections such as:

- `Findings`
- `References`
- `Open Questions`
- `Next Steps`
- `Commands`
- `Artifacts`

If the `README.md` starts becoming too crowded, those parts can be moved into
separate files such as `notes.md` or `references.md`.

## Key Constraint

Not every topic in this repository should be treated as portfolio material.

The structure must support:

- private or internal research
- rough exploratory notes
- unfinished work
- abandoned but still valuable findings
- selected topics that may later be polished for public display

The repository should not assume that every topic is public-facing, complete,
or presentation-ready.

## Tooling

A CLI tool will be built to manage this repository. The CLI handles topic
creation, status updates, portfolio flag changes, and INDEX.md maintenance.

The CLI is the preferred path for structured operations because it keeps the
repository consistent regardless of whether a human or an AI agent is doing
the work.

Direct editing is still allowed when speed matters, especially during active
research. The structure should support low-friction note capture first, with
tooling helping enforce consistency where it adds value.

The CLI will also expose an interface that AI tools can use to interact with
the repository without writing directly to files.

## Portfolio Integration

This repository will eventually be read by the portfolio engine as a data
source for research content.

Only topics marked `portfolio: true` will be surfaced. All other topics remain
in the repository but are ignored by the engine.

The `portfolio` field in each topic's frontmatter is therefore a contract
between this repository and the portfolio engine, not just a personal label.

## INDEX Contract

`INDEX.md` is the top-level browsing file for the repository.

Its job is simple:

- list every topic in one place
- show the current status of each topic
- show whether a topic is marked for portfolio eligibility
- make old research easy to rediscover

The index should stay lightweight. It is not meant to duplicate the full
content of each topic.

### Minimum Columns

The minimum information for each topic entry is:

- topic
- status
- portfolio
- started
- summary

### Example Shape

```md
| Topic | Status | Portfolio | Started | Summary |
| --- | --- | --- | --- | --- |
| [kerberos-delegation](./kerberos-delegation/) | active | no | 2026-05 | Delegation abuse paths, notes, and lab ideas |
| [dns-rebinding](./dns-rebinding/) | completed | yes | 2026-05 | Notes on browser trust boundaries and attack flow |
```

`INDEX.md` should be updated whenever a new topic is created or when a topic's
metadata changes enough that the index would become inaccurate.

## Guiding Principle

This repository should reduce the cost of saving research.

If the structure is too loose, the archive becomes hard to use. If the
structure is too heavy, research stops being captured.

The system should be strict enough to stay useful and light enough to be
used every time.
