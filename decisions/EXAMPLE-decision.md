---
id: DEC-0000
title: Use a brain to hold this work's decisions
type: decision
date: 2026-07-28
status: adopted
source: firsthand — the choice to start this brain
---

This is an **example record** so you can see the shape. `/capture` writes files
exactly like this — you never type the header yourself. Delete this file once your
brain has real records (or keep it as a reference).

## What we decided

We keep the durable reasoning behind this work in a brain: a version-controlled set
of Markdown records rather than scattered docs, chat messages, and memory.

## Why

- The "why" behind a decision is the first thing lost and the most expensive to
  reconstruct. Writing it down once beats re-deriving it.
- Plain text in git gives us history, diff, and review for free.
- A consistent record shape lets `/recall` actually find things.

## Notes on the format

- **`id`** — auto-assigned by `/capture` (`DEC-`, `CONV-`, `NOTE-`, `REF-`).
- **`type`** — one of decision / conviction / note / reference.
- **`status`** — decisions only: `proposed`, `adopted`, or `superseded`. When a new
  decision replaces this one, set this to `superseded` and reference the new id.
- **`source`** — **required on every record.** What this traces to: a meeting, a
  document, a person, or `firsthand`. This is the "never fabricate" rule made
  concrete — a record with no real source doesn't get written.
