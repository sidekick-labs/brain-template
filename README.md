# brain-template

A starter kit for a **brain** — a small, version-controlled home for the durable
knowledge behind a piece of work: the **decisions** you've made, the **convictions**
you hold, the **notes** you take, and the **reference** material you lean on.

You don't need to be a developer to use it. Everything is plain text, and five
guided skills do the typing for you inside the **Claude Code desktop app**.

> **What a brain is (and isn't).** A brain is *cognition* — the reasoning, the
> "why", the running memory of a body of work. It is **not** the work itself (that
> lives in your product/app/design repos). If you find yourself explaining the same
> decision twice, or wishing you'd written down *why* you chose something — that's a
> record for the brain.

## Start here

1. Someone (a technical teammate, or GitHub's **"Use this template"** button) has
   already created this repo for you and opened it in the Claude Code desktop app.
2. Type **`/setup`** and answer the questions. It takes about five minutes and
   writes this brain's identity and purpose for you.
3. From then on:
   - **`/capture`** — record a decision, conviction, note, or reference. You talk;
     it writes a clean, well-formed file.
   - **`/recall`** — ask "what do we know about X?" or "why did we decide Y?" and it
     searches the brain and answers with citations.

That's the whole day-one loop. No terminal, no config files, no accounts.

## The two phases of a brain

This template has **five** skills, split by what they need:

| Phase | Skills | Needs |
|-------|--------|-------|
| **1 — Day one, offline** | `setup`, `capture`, `recall` | Nothing. Works on the local files alone. |
| **2 — Connected** | `check-inbox`, `warm-up` | The repo pushed to GitHub, plus `gh` and the Gmail/Slack connectors attached to your Claude account. |

`check-inbox` turns actionable emails into tracked follow-up issues on this repo.
`warm-up` starts a work session by sweeping this repo's open issues/PRs plus your
inbox and Slack. **Both do nothing until Phase 2** — a technical teammate pushes the
repo and you connect the accounts once. The `setup` skill reminds you of this at the
end of its run.

## The one rule — never fabricate

Every record traces to something real: a document, a person, a meeting, or your own
firsthand knowledge. If something is unknown, it stays `UNKNOWN` — never a guess. The
`capture` skill enforces this by asking for a **source** on every record, and
`recall` never invents an answer the records don't support. A brain you can't trust
is worse than no brain.

## What's in here

```
decisions/     — choices made, and why (DEC-*)
convictions/   — durable stances / principles you hold (CONV-*)
notes/         — working notes, observations, open questions (NOTE-*)
reference/     — pointers to external docs, links, specs (REF-*)
brain.yaml     — this brain's identity (name, owner, purpose) — /setup fills it
CLAUDE.md      — guidance for Claude in this repo — /setup fills the purpose
.claude/skills — the five skills above
```

Each record is a single Markdown file. You never write the header by hand — the
skills do. See `decisions/EXAMPLE-decision.md` for what one looks like.

## Making your own brains from this

This repo is a GitHub **template repository**. To start a new brain, click
**"Use this template" → Create a new repository**, open it in the Claude Code
desktop app, and run `/setup`. Nothing structural needs changing — `/setup` handles
the identity and purpose.
