---
name: capture
description: "Record a decision, conviction, note, or reference in this brain as a well-formed Markdown file, so the user never has to write front-matter or pick an id by hand. Enforces the 'never fabricate' rule by requiring a source on every record. Offline — writes one local file. Use when the user says 'capture', 'record this', 'log a decision', 'note that', 'add a conviction', 'save this reference', or '/capture'."
---

# /capture — record something in the brain

Turn what the user tells you into one clean record file. **You** write the
front-matter and pick the id; they just talk. Offline — this writes a single local
Markdown file and nothing else (no git, no push).

## Steps

### 1. Pick the type
Decide (or ask) which of the four this is:

| Type | Folder | Prefix | It's a… |
|------|--------|--------|---------|
| `decision` | `decisions/` | `DEC-` | choice that was made, and why |
| `conviction` | `convictions/` | `CONV-` | durable stance / principle |
| `note` | `notes/` | `NOTE-` | working note, observation, open question |
| `reference` | `reference/` | `REF-` | pointer to an external doc/link/spec |

If it's ambiguous, ask one short question. When torn between decision and note,
prefer **note** unless a real choice was made.

### 2. Get the substance
Ask for the content in plain language if not already given. For a **decision**, draw
out both *what* was decided and *why*. Keep the user's own words where you can.

### 3. Get the source — REQUIRED (never fabricate)
Every record must trace to something real. Ask: *"Where does this come from?"* —
a meeting, a document, a person, a link, or `firsthand` (their own knowledge/choice).
**Do not write the record without a source.** If they don't have one, it's not a
fact yet — capture it as a `note` whose body says what's still unknown, with
`source: firsthand — open question`, rather than inventing certainty.

### 4. Allocate the id
Scan the target folder for existing `<PREFIX>NNNN` ids and use the next number,
zero-padded to 4 digits (e.g. the folder has `DEC-0000` and `DEC-0003` → next is
`DEC-0004`). Ids are never reused. Filename: `<id>-<short-kebab-title>.md`.

### 5. Write the file
```markdown
---
id: <PREFIX-NNNN>
title: <short title>
type: <decision|conviction|note|reference>
date: <YYYY-MM-DD today>
status: <proposed|adopted|superseded>   # decisions ONLY — omit this line otherwise
source: <meeting / doc / person / link / "firsthand …">
---

<the body, in the user's framing — for a decision, a "What we decided" and a "Why">
```

- `status` line appears **only** for decisions (default `adopted` unless they say
  it's still `proposed`).
- If this record **supersedes** an earlier one, ask which id, set this one's body to
  reference it, and update the older file's `status:` to `superseded` (the one time
  it's fine to edit an existing record's header).

### 6. Confirm
Report the file path and id, and read the one-line title back. Done — no commit, no
push (that's a separate technical step; mention it only if they ask how it gets
shared).
