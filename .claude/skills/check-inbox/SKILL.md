---
name: check-inbox
description: "Triage the connected Gmail inbox to inbox-zero and file actionable threads as deduplicated GitHub follow-up issues in the current repo. Use when the user says 'check my inbox', 'triage my email', 'process my inbox', 'clear my inbox', or '/check-inbox'. Scans everything in the inbox (read or unread, any age), gives each thread a disposition (file / archive-as-noise / park for review / leave), and is idempotent and read-only on mail except labels/archive/mark-read — never replies, sends, or deletes. Repo-agnostic (detects the repo from the git remote); uses the account-level Gmail and Slack MCP connectors (mcp__claude_ai_Gmail__*, mcp__claude_ai_Slack__*) and gh. Requires PHASE 2: this repo pushed to GitHub with gh + Gmail connectors attached."
model: sonnet
---

# /check-inbox

> **Phase 2 skill.** This does nothing useful until the brain repo is **pushed to
> GitHub** and the Claude account has the **`gh` and Gmail** connectors attached
> (Slack optional, for the digest). On a brand-new local-only brain, tell the user
> that and stop.

Sweep the inbox **toward inbox-zero**: every thread gets a disposition, the actionable ones become tracked GitHub issues, and the noise gets labeled-and-archived out of your face. What's left in the inbox after a run is only the deliberate residue — `Needs review` (parked) and `Leave` (untouched); everything filed or bucketed is archived out. Deterministic, idempotent, re-runnable without duplicates.

Invoke as `check my inbox`, `clear my inbox`, or `/check-inbox`. Pass `--dry-run` to report the dispositions it *would* apply without creating issues, labeling, archiving, or marking anything read. Run `--dry-run` the first time in any new repo.

## Why a skill, not a CI workflow

The Gmail connector (`mcp__claude_ai_Gmail__*`) is an **account-level claude.ai MCP** — it travels with the Claude account, so it's available in local Claude Code, in cowork, and in scheduled routines. A GitHub Actions job has **no** MCP access, so CI cannot read your mailbox. That's why this is a skill: no OAuth token, no service-account key, no repo secret to rotate. The mailbox it reads is simply *whichever Google account is connected to this Claude account*.

> **This skill triages whatever Google account is connected to the Claude account that runs it.** There's no per-repo override. Confirm the connected account before the first run / before scheduling.

## Configuration

Tunable constants — edit to retune. The target repo is auto-detected, so replication is a clean copy of this folder.

| Constant | Value | Notes |
|---|---|---|
| `SCAN` | `in:inbox` | The pool is **everything in the inbox** — read or unread, any age. `in:inbox` + archive-on-handle converges to zero and subsequent runs only see new arrivals (plus parked threads, which are cheap to skip). **Don't** rely on `-category:*` to filter noise; the disposition step does the filtering. |
| `TRIAGE_LABEL` | `triaged-to-github` | Applied to every thread that's been **filed** as an issue. Filed threads are archived, so they drop out of `SCAN`; this label is the durable "already filed" marker alongside the issue-body thread-id. |
| `REVIEW_LABEL` | `Needs review` | Borderline threads — labeled and **left in the inbox** (not filed, not archived). Parking also **marks the thread read** (the signal-bit). On later runs a parked thread is skipped while it stays read, but a **new inbound reply flips it back to unread**, so it resurfaces for re-triage (step 4). |
| `MARKETING_LABEL` | `Queued/Marketing` | Newsletters / marketing. |
| `RECEIPTS_LABEL` | `Queued/Receipts` | Financial/transactional receipts & confirmations. |
| `NOTIF_LABEL` | `Queued/Notifications` | Other automated system noise (DMARC, digests, bot notices). |
| `CALENDAR_LABEL` | `Queued/Calendar` | Calendar invites & updates. Only used when `ARCHIVE_CALENDAR` is on. |
| `ARCHIVE_CALENDAR` | `true` | When **true**, calendar invites/updates are bucketed to `CALENDAR_LABEL` and archived (they already live in your calendar). Set **false** to leave them untouched in the inbox. |
| `ISSUE_LABEL` | `inbox-triage` | GitHub label on every issue this skill opens. |
| `RULES_FILE` | `rules.yaml` (this folder) | Deterministic sender/subject → disposition routing, consulted **before** the LLM step. First match wins; unmatched threads fall through to LLM judgment. Keep entries conservative. |
| `SLACK_DIGEST_TARGET` | _(empty — disabled)_ | Slack channel the end-of-run summary is posted to (step 7), resolved by name at runtime. **Set this to your own channel or a self-DM to enable the digest** (recommended before scheduling). Leave empty to disable. If set but unresolvable, the run logs a warning and continues — the digest is best-effort, never fatal. |
| `MAX_THREADS` | `250` | Safety cap on **total** threads processed per run. The pool is paged through (50/page via `nextPageToken`) until the inbox is exhausted **or** this cap is hit. Raise it for a one-off bulk cleanup. |

## Hard rules (do not violate)

1. **Never destroy or act outward.** No delete, no trash, no reply, no RSVP, no send, no forward — **on the mailbox**. Archiving (removing `INBOX`) and marking read (removing `UNREAD`) are the only state changes, and they're reversible. If a thread asks for something, the *issue* captures it; a human does it. (The *one* outward message the skill sends is the end-of-run Slack digest to your own channel — step 7. That's a self-notification, never a reply to triaged mail.)
2. **Confidence-gated archiving.** Only the **Noise / Notification / Receipt** dispositions get archived + marked read, and only when you're clearly confident of the bucket. Anything unsure goes to **Needs review** (marked read, stays in inbox) or **Leave** (untouched) — **never** silently archive an ambiguous thread. This matters most on an unattended scheduled run.
3. **Idempotent — never double-file.** Filed (and noise/receipt/notification) threads are **archived**, so they leave the inbox and drop out of `SCAN` next run; every issue body carries a hidden marker `<!-- gmail-thread:<threadId> -->`. Before creating an issue, search existing issues for that marker (`gh issue list --search "gmail-thread:<threadId> in:body,comments" --state all`); an **open** match → skip creation and apply the disposition; a **closed-only** match → route to Needs review (a new reply landed after the issue was resolved). Because automated senders emit **sibling threads** for one event, a marker miss falls through to the sibling-event check before creating (step 4.3).
4. **Numbers and facts are quoted from the mail, never invented.** The issue summarizes what the email says. Don't infer amounts, deadlines, or commitments not in the text. Cite stated dates/figures; if absent, say "no date given".
5. **Privacy boundary.** Keep issue **titles** abstract — sender + topic, never a sensitive figure. Put detail in the body and link to the thread rather than pasting long quotes. Honor the repo's privacy posture (brains are usually private).

## Dispositions

For each thread, pick exactly one:

| Disposition | When | Action |
|---|---|---|
| **File** | Needs a response, decision, task, or tracks a deadline/commitment owed by you. | Create GH issue (dedup first) → apply `TRIAGE_LABEL` → archive + mark read. |
| **Noise** | Newsletters, marketing, product announcements, changelogs. | Apply `MARKETING_LABEL` → archive + mark read. |
| **Receipt** | Payment/transaction confirmations & financial notices. | Apply `RECEIPTS_LABEL` → archive + mark read. |
| **Notification** | Automated system notices with no action needed — DMARC, CI digests, bot notices. | Apply `NOTIF_LABEL` → archive + mark read. |
| **Calendar** | A calendar invite, update, or cancellation. | If `ARCHIVE_CALENDAR`: apply `CALENDAR_LABEL` → archive + mark read. Else: **Leave**. |
| **Needs review** | Borderline: *might* be actionable, or a weighty decision you shouldn't auto-judge. | Apply `REVIEW_LABEL` → **mark read, leave in inbox.** No issue, no archive. |
| **Leave** | Can't confidently bucket. | Untouched. |

The actionable-vs-noise-vs-borderline judgement is the one cognitive task here — that's the agent's job. Everything else (query, dedup, labeling, archiving) is mechanical. **When torn between File and Needs review, choose Needs review.**

**Recipient position is a strong signal of ownership.** "Owed by you" (the File bar) means *you* are on the hook. When the account owner is only in **`cc`** and the **`to`** recipient is someone else who clearly leads the thread, the action is almost always *theirs* — lean **Leave** (or **Needs review** if it might still need your input), not **File**. Direct `to:` (or a thread you've been replying to) is genuinely yours.

## Procedure

### 1. Resolve the target repo
```bash
gh repo view --json nameWithOwner -q .nameWithOwner   # e.g. sidekick-labs/design-ops-brain
```
If this fails (not in a repo / no remote / not pushed yet), stop and tell the user this is a Phase 2 skill — the repo must be on GitHub first.

### 2. Resolve / create labels (capture IDs) + the account address
- `mcp__claude_ai_Gmail__list_labels`. For each of `TRIAGE_LABEL`, `REVIEW_LABEL`, `MARKETING_LABEL`, `RECEIPTS_LABEL`, `NOTIF_LABEL`, and `CALENDAR_LABEL` (only if `ARCHIVE_CALENDAR`): if absent, `create_label`; capture every **labelId**. **Gmail queries and label calls take the ID, not the display name.** (System IDs: `INBOX`, `UNREAD`.)
- **Capture the connected account's email** (the common `to`/`cc` recipient across the pool). Use it as `authuser` in issue links (step 4.3) so they resolve regardless of which Google account is first in the reader's browser.
- GitHub: `gh label list`; if `ISSUE_LABEL` is absent, create it with `gh label create inbox-triage --color FBCA04 --description "Follow-up filed from inbox triage"` (idempotent). **Do NOT** bootstrap the label by opening a placeholder issue.

### 3. Pull the pool
Page through `SCAN` newest-first with `mcp__claude_ai_Gmail__search_threads` (`pageSize: 50`), following `nextPageToken` until it's absent **or** you've collected `MAX_THREADS` threads. **Don't stop at the first page** — `in:inbox` has no recency limit. Empty first page → report "Inbox clean — nothing to triage" and stop. If you hit `MAX_THREADS` first, triage that batch (newest first) and report the remaining count.

> **Quirk:** `in:inbox` matches **loosely** — Gmail returns the whole thread if *any* message matches, so the pool can include threads whose **latest** message is already archived. Before triaging, check the **latest** message's `labelIds`:
> - **skip (leave untouched) if it no longer contains `INBOX`** — already archived.
> - **if it contains `REVIEW_LABEL`:** parked thread. Skip while its latest message is read; only if the latest message is **unread** (a new reply landed) drop the parked status and re-triage fresh.
>
> Don't `get_thread` everything; when you do, if FULL_CONTENT exceeds the tool limit, fall back to the search snippet of the latest message.

### 4. Cluster, then triage

**First, collapse near-duplicates.** Group the pool by **normalized subject + sender domain** (strip `Re:`/`Fwd:`/`Reminder:` prefixes and trailing counts). Within a cluster, **disposition only the latest thread**; the rest inherit the same bucket action without a separate issue. Report collapsed clusters as "+N earlier". Distinct threads that merely share a topic are **not** a cluster.

Load `RULES_FILE` once before the loop. **Then triage each remaining thread (newest first):**
0. **Apply the step-3 skip-checks first, then check `rules.yaml`.** The rule fast-path runs **only on threads that survived the step-3 skip-checks**. This ordering is load-bearing: a rule must **never** be applied to a thread the user deliberately parked or that's already archived. If the latest (surviving) message matches a rule (all of its `from`/`subject` conditions), apply that disposition directly — **no `get_thread`, no model judgment** — and go to step 5. A rule asking to `file` is ignored → fall through. No match → continue.
1. **Otherwise disposition from the search snippet** — sender, subject, and the latest-message snippet are enough to bucket most (all noise/receipts/notifications/calendar). Only call `get_thread` (FULL_CONTENT) when you need the body: a **File** candidate, or a thread the snippet can't disambiguate. Always judge on the **latest** message.
2. Pick a disposition (table above).
3. **If File:** dedup via `gh issue list --search "gmail-thread:<threadId> in:body,comments" --state all --json number,title,state`.
   - An **open** issue matches → don't create; note "already tracked (#N)" and apply the File disposition (archive).
   - **Only a closed** issue matches → new reply after resolution → **Needs review** (leave in inbox); do **not** silently re-archive.
   - No marker match **and the sender is automated** → run the **sibling-event check**: `gh issue list --search "<normalized subject keywords> in:title" --state all --json number,title,state,body`, treating a hit as a duplicate only under the cluster test across runs (same sender domain + same recurring notice/escalating series). Shared topic alone is **not** a match.
     - **Open** sibling → don't create; archive, and comment this thread's link + marker onto the matched issue.
     - **Closed-only** sibling → this email older/same-day as the issue's date → late duplicate, archive + marker-comment. **Newer** → re-fired after resolution → **Needs review**.
   - No match (human sender, or no sibling) → create:
   ```bash
   gh issue create \
     --title "<sender>: <concise topic>" \
     --label inbox-triage \
     --body "$(cat <<'EOF'
   **From:** <sender> · **Received:** <date>
   **Thread:** https://mail.google.com/mail/?authuser=<account-email>#all/<threadId>

   **Why this needs follow-up:** <one line>

   **What's being asked / the action:** <1–3 bullets; quote stated dates/figures verbatim, invent nothing>

   <!-- gmail-thread:<threadId> -->
   EOF
   )"
   ```
   Keep the title abstract (rule 5).

### 5. Apply the disposition (the mutation)
- **File / Noise / Receipt / Notification / Calendar** (Calendar only when `ARCHIVE_CALENDAR`): `label_thread` with the bucket label id, then `unlabel_thread` to remove `INBOX` (archive) and `UNREAD` (mark read).
- **Needs review:** `label_thread` with `REVIEW_LABEL` id, then `unlabel_thread` to remove `UNREAD` (mark read) — but **keep `INBOX`**. Read-and-in-inbox is the parked state.
- **Leave:** do nothing.
- **Collapsed cluster members:** apply the same archive/label as the representative; never open a second issue.

> In `--dry-run`: do the *decision* (steps 1, 2, and the dedup lookup) but create nothing and apply no labels/archive/read changes. Report the table only.

### 6. Report
```
Inbox triage — <repo> — <N> scanned → <C> cleared from inbox · <S> still in inbox
  cleared (C) = <M> filed + <K> already-tracked + <A> archived   — all archived; they LEAVE the inbox
  still (S)   = <R> review + <L> left                            — they REMAIN in the inbox
  invariant: N = C + S
Filed (M):
  #12  Acme: renewal question     →  https://mail.google.com/mail/?authuser=<account-email>#all/<threadId>
Needs review (R) — still in your inbox:
  • GitHub: bot permissions request          →  ...
Archived (A): 12 noise · 5 receipts · 3 notifications · 2 calendar
Collapsed clusters: Framer invite reminders (+3)
Already tracked (K): #8
Left untouched (L): 1
Remaining beyond cap: <0 or count>
```

### 7. Post the digest to Slack
Unless `--dry-run` or `SLACK_DIGEST_TARGET` is empty, post the step-6 summary to Slack so an **unattended run is observable**.
- Resolve `SLACK_DIGEST_TARGET` to a channel id (`mcp__claude_ai_Slack__slack_search_channels` by name, or a self-DM), then `mcp__claude_ai_Slack__slack_send_message`.
- Lead with a one-line headline: `📥 Inbox triage — <C> cleared (<M> filed · <A> archived) · <R> to review · <L> left`. On a clean run, still post `📥 Inbox clean — nothing to triage`.
- **Best-effort, never fatal:** if the channel can't be resolved or Slack errors, log a warning and finish normally.
- This is the **only** outward-facing message the skill sends, and it goes to your own channel — never a reply to triaged mail (hard rule 1).

## Scheduling

Run it daily unattended via a `/schedule` routine pointing at `/check-inbox`. Attach **both** the Gmail and Slack account-level connectors to the routine. The confidence gate (rule 2) is what makes unattended archiving safe: only clear-cut noise leaves the inbox; anything borderline is parked in `Needs review`. Set `SLACK_DIGEST_TARGET` first — the digest is your window into (and heartbeat for) each unattended run.

> **Be the only automated triager on the mailbox.** If a mail client with its own AI triage is also moving messages on this account, the two will fight. Retire the other one before scheduling this.
