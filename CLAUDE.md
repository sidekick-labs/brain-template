# CLAUDE.md

Guidance for Claude Code working in this brain.

> **New brain?** This file is a template. Run **`/setup`** and it will rewrite the
> **Purpose** section below for this specific brain. Everything else already applies.

## Purpose

<!-- setup:purpose -->
_This brain does not have a purpose statement yet. Run `/setup`._

_Once set, this reads as:_
_"This brain owns **X** for **whom**, so that they can **Y**, without **failure-mode**."_
<!-- /setup:purpose -->

## The one rule — never fabricate

Every date, decision, fact, and status in this brain traces to something real: a
document, a person, a meeting, or explicit firsthand knowledge. If something is
unknown, record it as `UNKNOWN` — never a guess. A wrong guess that later looks like
a recorded fact is how a brain loses the trust that makes it worth keeping. This is
non-negotiable and it applies to every skill here: `capture` will not write a record
without a `source:`, and `recall` never answers beyond what the records support.

## The folders

| Folder | What goes here | Id prefix |
|--------|----------------|-----------|
| `decisions/` | A choice that was made, and the reasoning behind it. | `DEC-` |
| `convictions/` | A durable stance or principle this work holds to. | `CONV-` |
| `notes/` | Working notes, observations, open questions. | `NOTE-` |
| `reference/` | Pointers to external docs, links, specs (not the content itself). | `REF-` |

Every record is one Markdown file with a small front-matter header (the skills write
it — see `decisions/EXAMPLE-decision.md` for the shape). Never hand-edit the header
format; use `/capture` so ids and fields stay consistent for `/recall` to search.

## The skills

| Skill | Use it to | Needs |
|-------|-----------|-------|
| `/setup` | One-time: name this brain and write its purpose. | nothing |
| `/capture` | Add a record (decision / conviction / note / reference). | nothing |
| `/recall` | Ask what this brain knows, with citations. | nothing |
| `/check-inbox` | File actionable emails as follow-up issues on this repo. | Phase 2 |
| `/warm-up` | Start a work session: sweep this repo's issues/PRs + inbox + Slack. | Phase 2 |

**Phase 2** = the repo is pushed to GitHub and `gh` + the Gmail/Slack connectors are
attached to the Claude account. `check-inbox` and `warm-up` are inert until then.

## Working conventions

- **Records are append-friendly, edit-carefully.** Prefer capturing a new record
  (e.g. a decision that supersedes an earlier one, with `status: superseded` set on
  the old one) over rewriting history. The value of a brain is the trail.
- **Keep it private-safe.** This brain may hold internal reasoning and people
  references. Don't paste secrets or credentials into records.
- **When unsure, ask — don't invent.** See the one rule.
