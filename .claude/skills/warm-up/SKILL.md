---
name: warm-up
description: "Start a work session warm in this brain. Sweep the live signal that needs the user — open GitHub issues + PRs in THIS repo, plus the Gmail inbox and Slack threads via the attached MCP connectors — split it into work the skill can prep autonomously vs. calls that need a human decision, prep the autonomous batch in the BACKGROUND (drafts/staged only — never commit, push, PR, post, send, or merge), then walk the rest ONE item per turn. Requires PHASE 2: this repo pushed to GitHub with gh attached (Gmail/Slack optional lenses). Invoke when the user says 'warm up', 'warm-up', 'start my session', 'spin up', 'what should I pick up', 'get me going', or '/warm-up'."
---

# /warm-up — start a work session in this brain

Get the user oriented fast: gather everything live that wants their attention, sort
it into *what you can prep for them* vs *what only they can decide*, do the prep
quietly in the background, then walk the decisions one at a time.

> **Phase 2 skill.** Needs this repo **pushed to GitHub** with the **`gh`** connector
> attached. The Gmail and Slack lenses are optional — they light up if those
> connectors are attached, and degrade soft (skipped, noted) if not. On a
> local-only brain, say so and stop.

## Hard rules

- **Prep only — never act outward.** In this skill you may read, search, and stage
  drafts. You must **never** commit, push, open/merge a PR, post to Slack, send email,
  or close an issue **without the user's explicit say-so in the walk**. Staging a
  draft is fine; dispatching it is the user's call.
- **Never fabricate status** (the brain's one rule). Every item you surface traces to
  a real issue/PR/email/message. If a lens is unavailable, say "skipped — connector
  not attached", never invent its contents.
- **One decision per turn** in the walk. Don't dump a wall of choices.

## Phase 0 — Scope
Resolve the repo: `gh repo view --json nameWithOwner -q .nameWithOwner`. If it fails
(no remote / not pushed), stop — this is a Phase 2 skill. State which lenses are
available this session (GitHub always; Gmail/Slack iff their connectors respond).

## Phase 1 — Gather (in parallel where you can)
- **GitHub — this repo:** open issues assigned to or opened by the user, and open PRs
  (theirs + review-requested). `gh issue list` / `gh pr list` with sensible filters.
- **Gmail (if attached):** actionable threads awaiting the user — but do NOT re-triage
  the whole inbox (that's `/check-inbox`'s job). Surface only clear asks. If
  `/check-inbox` has been filing follow-ups, those already appear as GitHub issues —
  don't double-count.
- **Slack (if attached):** threads/mentions awaiting the user's reply.

## Phase 2 — Triage into two buckets
For each gathered item, classify:
- **Autonomous-preppable** — you can gather context, draft, or stage something useful
  without a decision (e.g. summarize an issue's history, draft a reply for review,
  pull the relevant brain records via the same search `/recall` uses, assemble a
  PR-review checklist). 
- **Needs-the-user** — a real judgment call, an approval, a "which way do we go".

Also run a **close-confirm backstop:** any issue whose delivering PR has already
merged (or that a prior session clearly resolved) → float it as a fast one-click
"close this?" instead of presenting it as fresh work.

## Phase 3 — Present the board
Give one compact overview: counts per lens, the autonomous batch you're about to
prep, and the count of items that will need them. Lead with anything time-sensitive.

## Phase 4 — Background prep (staged only)
Prep the autonomous batch. For a brain, "prep" is usually: context summaries,
`/recall`-style pulls of relevant records, and **draft** replies/notes saved to a
scratch location (e.g. `.warm-up/<timestamp>/`, which is gitignored). **Nothing is
committed, pushed, sent, or posted.** Note what you staged.

## Phase 5 — Walk the rest, one per turn
Take the **Needs-the-user** items one at a time. For each: state the item, show the
context you prepped, give a recommended next action, and ask. Act only on their
answer — and only within the hard rules (staging vs dispatching). Move to the next
only when the current one is resolved or explicitly parked.

## Phase 6 — Wrap
Short recap: what got decided, what you staged (and where), what's parked. If
anything wants capturing as a durable record, offer `/capture`. Don't post a summary
anywhere outward unless asked.
