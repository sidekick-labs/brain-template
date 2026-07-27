---
name: recall
description: "Answer 'what do we know about X?' or 'why did we decide Y?' by searching this brain's records and summarizing with citations back to record ids and their sources. Never invents an answer the records don't support. Offline — read-only over local files. Use when the user says 'recall', 'what do we know about', 'why did we decide', 'what did we say about', 'search the brain', or '/recall'."
---

# /recall — ask the brain what it knows

Search the four record folders and answer the user's question, **grounded only in
what the records actually say**. Read-only. Offline.

## Steps

### 1. Understand the question
Pull out the key terms (topic, decision, person, date range). If it's a "why did we
decide X" question, you're looking mainly in `decisions/`; a broad "what do we know
about X" spans all four folders.

### 2. Search
Grep across `decisions/`, `convictions/`, `notes/`, `reference/`:
- Match the key terms in both **front-matter** (`title`, `type`, `date`, `source`)
  and **body** text. Try a few phrasings/synonyms — records may use different words.
- Note each hit's `id`, `title`, `date`, `status`, and `source`.

### 3. Answer with citations
Write a short, direct answer, then back every claim with the record it came from:

- Cite as **`[DEC-0004]`**, **`[NOTE-0012]`**, etc., inline.
- For a "why" question, quote or paraphrase the decision's reasoning and give its
  `source:` so the user can see it traces to something real.
- If records **conflict** (e.g. a decision and a later one that supersedes it),
  surface both and point out the `superseded` status — the newest adopted decision
  wins, but say so explicitly.

### 4. Never fabricate
- If the records **don't** cover the question, say **"the brain has nothing on
  this"** (or only tangential records) — do not fill the gap with a guess or general
  knowledge. Offer to `/capture` what the user knows so the gap closes.
- Don't state a fact more confidently than its source warrants. If a record's source
  is `firsthand — open question`, present it as an open question, not a settled fact.

### 5. Point to the files
End with the list of record files you drew on (path + id + title) so the user can
open the originals.
