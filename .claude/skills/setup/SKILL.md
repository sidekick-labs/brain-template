---
name: setup
description: "One-time setup for a new brain created from brain-template. Interviews the user to name the brain and articulate its purpose (the 'owns X for whom, so that Y, without failure-mode' frame), then writes brain.yaml and rewrites the Purpose block in CLAUDE.md. Offline — touches only local files, never git/GitHub/tokens. Use when the user says 'set up', 'set up this brain', 'setup', or '/setup', or on the first session in a freshly created brain repo where brain.yaml is still blank."
---

# /setup — name and frame a new brain

A one-time, friendly interview. Goal: fill in `brain.yaml` and the **Purpose** block
of `CLAUDE.md` so this brain knows what it is. Assume the person is **not** technical
— ask plain questions, one at a time, and do all the writing yourself.

**Offline only.** This skill edits local files (`brain.yaml`, `CLAUDE.md`) and
optionally captures a first record. It NEVER runs git, creates a repo, pushes, or
sets any token. If asked to do those, explain they're a separate technical step.

## Before you start

- If `brain.yaml`'s `name:` is already filled in, this brain is already set up. Say
  so, show the current purpose, and ask if they want to re-run setup (overwrite) or
  stop. Don't silently overwrite an existing identity.

## The interview (one question at a time)

Ask these in order. Offer an example with each. Keep it conversational — accept
messy answers and tidy them up yourself.

1. **What is this brain called?**
   A short kebab-case name ending in `-brain` (e.g. `design-ops-brain`,
   `partnerships-brain`). If they give a plain phrase, propose the kebab form back.

2. **Who owns it?** Name + email.

3. **The purpose — four small blanks.** Explain you'll build one sentence together:
   *"This brain owns **X** for **whom**, so that they can **Y**, without
   **failure-mode**."* Ask for each blank separately, with examples:
   - **X — what it holds** ("the reasoning behind our vendor choices", "everything we
     know about our onboarding funnel").
   - **whom — who relies on it** ("the design team", "just me", "whoever picks up
     this project next").
   - **Y — what that lets them do** ("stop re-litigating settled decisions", "answer
     'why did we do it this way' in seconds").
   - **failure-mode — what it prevents** ("knowledge walking out the door when
     someone leaves", "the same debate every quarter", "guesses hardening into
     fake facts").
   Then read the assembled sentence back and confirm.

## Write the files

Once confirmed:

1. **`brain.yaml`** — fill `name`, `owner`, `purpose` (the one-line sentence),
   `created` (today's date, `YYYY-MM-DD`). Leave the network-wiring comment intact.

2. **`CLAUDE.md`** — replace everything between `<!-- setup:purpose -->` and
   `<!-- /setup:purpose -->` with a short **Purpose** body: the one-line sentence,
   then 2–4 lines expanding it (what X covers, who whom is, why the failure-mode
   matters). Keep the comment markers so setup can be re-run. Touch nothing else.

## Offer a first record

Ask if there's a decision, conviction, or note they already want to capture. If yes,
hand off to the **`/capture`** flow (same repo). If no, that's fine.

## Close — explain what's live and what isn't

End with a short, plain-language recap:

- ✅ **Ready now (no setup needed):** `/capture` to add records, `/recall` to ask
  what the brain knows.
- ⏳ **Needs one technical step first:** `/check-inbox` and `/warm-up`. They only work
  after this repo is **pushed to GitHub** and your Claude account has the **`gh`,
  Gmail, and Slack** connectors attached. Tell them to ask a technical teammate to
  push the repo once; after that, connecting the accounts is a few clicks in the
  desktop app.

Do not attempt the push or the connections yourself.
