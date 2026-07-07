# LLM Wiki — Schema & Operating Manual

*This is a sanitized copy of the instruction file the librarian follows. Every real name, company, and project codename has been replaced with a synthetic equivalent (Jane Doe, Acme Corp, Project Atlas). The real vault stays local.*

This file tells the LLM how this personal wiki is structured and how to maintain it.
The curator gathers sources and asks questions; the LLM does all writing, filing, cross-referencing, and bookkeeping. The curator never writes wiki pages by hand.

Pattern reference: Andrej Karpathy's LLM-wiki idea (see the README).

---

## 0. Hard rules (read first)

1. **This vault is LOCAL-ONLY. Never commit or push it.** It contains real personal information about real people (founders, investors, contacts) by design. Do not run `git commit`, `git push`, `git add`, or any repo-mutating command in this directory, ever, even if asked offhandedly. Confirm explicitly first and remind the curator this vault is meant to stay uncommitted. PII is *allowed on pages* precisely because nothing here leaves the disk. (This public repo is a separate, sanitized artifact.)
2. **The LLM owns `wiki/`. The curator owns `raw/`.** Read from `raw/`, write to `wiki/`. Never edit a source file in `raw/`.
3. **Sources are immutable and citable.** Every claim on a wiki page cites the source file it came from.
4. **One thing = one home.** Prefer updating an existing page over creating a duplicate. If two pages describe the same entity, merge them.
5. **Move a source to `raw/processed/` only AFTER its wiki pages are successfully written** — never before.
6. **Clippings are external reading, not the curator's own writing.** A source under `raw/inbox/Clippings/` (or `raw/processed/Clippings/`) is something the curator read online — a Reddit post, article, tweet, LinkedIn thread. Two schema rules apply: (a) cite claims from clippings with **`(reading: …)`** — never `(source: …)` — so wiki readers can tell external reading apart from the curator's own meetings/logs/records at a glance; (b) every clipping gets a row in **[[Reading List]]** (a `type: record` index page) *in addition to* distributing insights into concept pages. Never create a per-clipping wiki page — the verbatim already lives in `raw/processed/Clippings/`.

---

## 1. Directory layout

```
Vault/
├── schema.md                 # this file — the schema
├── log.md                    # append-only operations log (ingest/query/lint)
├── raw/
│   ├── inbox/                # the curator drops new sources here
│   │   └── Review Queue.md   # CONTROL FILE — never a source (see §4c)
│   └── processed/            # sources the LLM has finished ingesting —
│       │                     #   MIRRORS the inbox subfolder structure (see below)
│       ├── Working Log/      # processed daily-log sources
│       ├── Meeting Log/      # processed meeting-note sources
│       ├── Learnings/        # processed event/learning sources
│       ├── Clippings/        # processed web clippings
│       ├── Correspondence/   # processed email threads (feeder → ingest; see §4d)
│       ├── Workflow/ …       # (one subfolder per inbox subfolder)
│       └── assets/           # images, after extraction
└── wiki/                       # LLM-owned. Six page types, no others.
    ├── index.md                # catalog of all pages, by type
    ├── Exploration Spine.md    # the SPINE project — lives at wiki root (not in Projects/)
    ├── People/                 # contacts, founders, investors
    ├── Projects/               # active efforts with a goal + changing status
    ├── Concepts/               # bodies of knowledge that just accumulate
    ├── Meetings/               # meeting notes/transcripts, verbatim
    ├── Log/                    # daily log entries, by date, verbatim
    └── Records/                # verbatim reference docs (incorporation, financials, …)
```

**Folders are Title Case** (`People/`, `Projects/`, …). The spine project `Exploration Spine.md` sits at the wiki root next to `index.md` for quick access; it is still a `type: project` page. Obsidian resolves `[[wikilinks]]` by filename, so a page's folder never appears in links.

**Processed mirrors inbox.** When moving a source from `raw/inbox/` to `raw/processed/`, recreate the same subfolder it came from (a file from `inbox/Working Log/` lands in `processed/Working Log/`; root-level inbox files stay at the processed root). Cite the full processed subpath, e.g. `(source: raw/processed/Working Log/2026-05-29, Fri.md)`.

**Do not invent new top-level page types.** There is no `companies/`, no `topics/`, no `ideas/`. Everything maps to one of the six. (A company is not a page — see §2.1.)

**Two different "logs," don't confuse them:**
- `wiki/Log/` = the curator's *daily journal entries*, filed verbatim by date.
- `log.md` (vault root) = the *operations ledger* — one line per ingest/query/lint the LLM performs.

---

## 2. The six page types

### 2.1 `wiki/People/`
Contacts, founders, investors — any real person.

- **Filename:** Title Case full name — `Jane Doe.md` (see §8). Disambiguate collisions with a qualifier (`Jane Doe (Acme).md`).
- **Company affiliation is NOT a separate page.** A person's company lives as a field/note *on their page* (plain text or an external note), never as a `companies/` page and never as a `[[wikilink]]` that would spawn a company page. If a company matters as an *active effort of yours*, that's a project; if it matters as *a thing you're studying* (a competitor, a market player), that's a concept.
- Link people to the projects and concepts they're involved in with `[[wikilinks]]`.
- **Optional `## Correspondence` and `## Email tone` sections (for the email drafter).** `## Correspondence` collects facts distilled from the curator's email threads with this person (contact info, commitments, "last emailed re: X"); it is fed by Correspondence sources (§4d). `## Email tone` is an **optional override, not a required field** — add it *only* to correct a tone the drafter gets wrong, or to record how the curator writes to someone whose relationship isn't visible in the mailbox (e.g. a former colleague on the broadcast list). Otherwise tone is **inferred** per draft from the person's relationship + the curator's past replies (see `_automation/voice-profile.md`, the drafter's rubric). Do **not** add a tone line to every page — that is O(N) busywork and a mislabel silently biases every future draft.

```markdown
---
type: person
tags: [founder]
company: "Acme Corp"          # plain text — NOT a wikilink, NOT its own page
role: "CEO & co-founder"
created: 2026-06-12
updated: 2026-06-12
aliases: []
sources: 0
---

# Jane Doe

One-line who-they-are.

## Context
- Met at … (source: raw/processed/2026-04-21-acme-intro-call.md)

## Relationships
- Investor in [[fundraising-seed]] · co-founder with [[john-smith]]

## Notes
- …

## Correspondence            # optional — facts from the curator's email threads (§4d)
- 2026-06-18: … (source: raw/processed/Correspondence/…)

## Email tone                # optional OVERRIDE only — usually omit (see bullet above)
- …
```

### 2.2 `wiki/Projects/`
Things you are **actively doing** that have a goal and a status that changes over time — your startup, a fundraise, an experiment.

- **Filename:** kebab-case — `fundraising-seed.md`.
- **Must contain `## Current status` and `## Open questions / next steps`.** If a page would naturally have those sections, it is a project (see §3, the decision rule).
- Cross-link freely to concepts and people.

```markdown
---
type: project
status: active                # active | paused | done | abandoned
tags: []
created: 2026-06-12
updated: 2026-06-12
sources: 0
---

# Fundraising — Seed

Goal: …

## Current status
- … (source: raw/processed/…)

## Open questions / next steps
- [ ] …

## Background / notes
- …

## Related
- [[jane-doe]] · [[saas-pricing]]
```

### 2.3 `wiki/Concepts/`
Bodies of knowledge that are **true independent of you** and just accumulate — a technology, a technique, a market, a competitor.

- **Filename:** kebab-case — `retrieval-augmented-generation.md`.
- No status, no next-steps — a concept page just grows as facts arrive. (If it grows a "what should I do about this" section, it has become a project — split it.)
- Cross-link freely to projects and people.

```markdown
---
type: concept
tags: []
created: 2026-06-12
updated: 2026-06-12
sources: 0
---

# Retrieval-Augmented Generation

## Summary
…

## Facts
- … (source: raw/processed/…)

## Open threads / gaps
- (things worth a future web search or source — surfaced by lint)

## Related
- [[llm-wiki-pattern]]
```

### 2.4 `wiki/Meetings/`
Meeting notes and transcripts, **filed verbatim. Never summarized, never paraphrased, never trimmed.**

- **Filename:** `YYYY-MM-DD Title Slug.md` — `2026-04-21 Acme Intro Call.md` (ISO date prefix + Title Case; see §8).
- The body is the original text exactly as received. Insights get *distributed* into people/projects/concepts pages (see §4) — they are not extracted *out of* the meeting page.

```markdown
---
type: meeting
date: 2026-04-21
attendees: ["[[jane-doe]]", "[[john-smith]]"]
tags: []
source: raw/processed/2026-04-21-acme-intro-call.md
---

# 2026-04-21 — Acme intro call

<verbatim notes / transcript, unedited>
```

### 2.5 `wiki/Log/`
Your daily log entries, **filed verbatim by date.**

- **Filename:** `YYYY-MM-DD.md`.
- Verbatim, like meetings. Distribute insights outward; never rewrite the entry itself.

```markdown
---
type: log
date: 2026-06-12
tags: []
source: raw/processed/2026-06-12-daily.md
---

# 2026-06-12

<verbatim daily log>
```

### 2.6 `wiki/Records/`
Verbatim **reference documents** — incorporation, financials, tax, monthly newsletter templates, and similar records the curator wants to *find while browsing the vault*.

- **Same rules as meetings:** filed **verbatim, never summarized**, and citable. Other pages cite/link them (e.g. `[[Incorporation]]`) rather than copying their contents.
- These are evolving-knowledge-adjacent but stable; they live in the wiki (not `raw/`) so they're browsable. (This supersedes the earlier "records stay in raw" rule.)
- **Filename:** Title Case (see §8) — `Incorporation.md`, `Financials.md`.

```markdown
---
type: record
date: 2026-04-20
tags: [incorporation]
source: raw/inbox/Corporation/Incorporation.md
---

# Incorporation

<verbatim record, unedited>
```

---

## 3. Project vs. concept — the decision rule

> **If a page would naturally have a "current status" or "next steps" section, it is a PROJECT.**
> **If it is a body of knowledge that just grows as facts accumulate, it is a CONCEPT.**

Test it by asking *"is this mine to move forward, or is it true regardless of me?"* Your fundraise = project. The mechanics of SAFEs = concept. Your startup = project. The market it competes in = concept. **Projects and concepts should cross-link freely** — a project page links out to the concepts it depends on, and concept pages link back to the projects that touch them.

When a single source is genuinely ambiguous between the two, **pause and ask** (see §4).

---

## 4. Operation: INGEST

Triggered when the curator says to process `raw/inbox/`. Runs **autonomously in batch, but pauses to ask on ambiguity.**

For each source in `raw/inbox/` (excluding the `Review Queue.md` control file — see §4c):

1. **Read it fully.**
   - Text: read the whole file.
   - **Images/photos:** view the image, then extract its content — whiteboard text, slide contents, who/what is shown, handwriting. Write that extracted information into the relevant pages, citing the image file.
2. **Classify.** Decide which page type(s) it informs and whether it is itself a verbatim artifact (a meeting or a daily log).
3. **File verbatim first (if applicable).** If the source is a meeting or daily log, write the original verbatim into `wiki/Meetings/` or `wiki/Log/` before anything else.
4. **Distribute insights.** Update the relevant `people/`, `projects/`, and `concepts/` pages with what the source adds. **Prefer updating an existing page** over creating a new one. A single source may touch 5–15 pages. Every claim added gets a citation: `(source: raw/processed/<file>)` — cite the *processed* path, since that's where the file will live once ingest succeeds.
   - **Every real external person a meeting/log/correspondence names gets a People page.** If an attendee or correspondent has no page, **create one** (name + one-line who-they-are + how the curator met them + relationship), then link them (as a meeting `attendee`, or from the relevant pages). Watch for transcription misspellings of known names (§7b) so you update the existing page instead of spawning a duplicate — add the misspelling as an `alias`. (This is how contacts like meeting guests and email correspondents become first-class pages the email drafter can ground on.)
5. **PAUSE AND ASK** when:
   - a person/project/concept might be new *or* might be an existing page under a different name;
   - a source is ambiguous between project and concept (§3);
   - a new claim **contradicts** an existing claim — surface both, with both citations, and ask which holds (don't silently overwrite);
   - a source looks like it wants a new page type outside the five (it doesn't — resolve it into one of the five, asking if unsure).
6. **Records ≠ knowledge.** Financials, incorporation docs, tax returns, cap tables and similar are *reference material*. Leave them in `raw/` and *cite* them from the relevant project/concept pages — do **not** create concept pages for them.
7. **Update `index.md`** to reflect new/changed pages (§6).
8. **Append to `log.md`:** `## [YYYY-MM-DD] ingest | <title>` followed by a one-line note of what was touched (§6).
9. **Move the source** from `raw/inbox/` to `raw/processed/` (images to `raw/processed/assets/`) — **only now, after the writes above succeeded.** If any write failed, leave the source in `inbox/` and report.
10. **Bump frontmatter:** set `updated:` and increment `sources:` on every page you touched.

After the batch, **report**: list every page created/updated, every source moved, and every point where you paused.

---

## 4b. Operation: INGEST-QUEUE (no-pause mode)

A non-interactive variant of INGEST for when the curator wants the inbox cleared without being asked anything. Same filing rules as §4, but **never pause and never guess.**

1. Process every source in `raw/inbox/` (excluding the control file — see §4c) in the normal way: classify, **file verbatim** (meetings/logs/records), **update existing pages** and create clearly-warranted new ones, cite every claim, update `index.md`, append to `log.md`, and move each source to `raw/processed/<mirrored subfolder>/` only after its writes succeed.
2. **Do not pause to ask. Do not guess.** Whenever you would normally pause (§4.5) — a new-vs-existing ambiguity, project-vs-concept uncertainty, a contradiction, an unclear classification, a thin source you can't place — **do not resolve it by guessing.** Instead:
   - File whatever *is* unambiguous (e.g. the verbatim copy) and skip the uncertain distribution, OR leave the source in `inbox/` if nothing can be done safely; and
   - **Append an entry to `raw/inbox/Review Queue.md`** describing exactly what's unclear, which source it's from, and the options you see — so the curator can decide later.
3. Prefer **updating existing pages** over creating near-duplicates (one thing = one home), exactly as in §4.
4. After the run, report what was filed, what was moved, and a summary of everything written to the Review Queue.

Use plain **INGEST (§4)** when the curator wants to stay in the loop; use **INGEST-QUEUE** when they want a hands-off sweep with questions deferred to the Review Queue.

## 4c. `Review Queue.md` — control file, never a source

`raw/inbox/Review Queue.md` is a **control file**, not a journal or a source.

- **Never ingest it, never move it, never process it, never write it into the wiki.** It stays in `raw/inbox/` permanently.
- It is the *only* file in `raw/inbox/` that ingest skips. Everything else there is a source.
- INGEST-QUEUE **appends** open questions to it (don't overwrite prior entries). The curator reads it, resolves items, and clears them by hand. The LLM may remove an item only after the curator confirms it's resolved.
- Entry format: `## [YYYY-MM-DD] <source file> — <what's unclear>` followed by the options/what-you-need-to-know.

---

## 4d. Correspondence sources — the curator's own email threads

`raw/inbox/Correspondence/` holds the curator's **own two-way email threads**, filed automatically by the `email-to-wiki-feeder` task (see `_automation/`). They are the curator's own record, so:

- **Cite them `(source: raw/processed/Correspondence/<file>)`** — NOT `(reading: …)`. (Contrast Clippings: those are *external* reading the curator consumed → `(reading: …)`; correspondence is the curator's *own* record → `(source: …)`, like meetings and logs.)
- The feeder writes **one dated `Daily Email Log` per day** into `raw/inbox/Correspondence/` — that day's *new* messages only, grouped by thread (each block tagged with the Gmail `thread_id` + participants). The nightly **INGEST-QUEUE** routes each thread block to the participant's living page (participants → which page; `thread_id` + subject → recognize a continuation), distilling contact info, commitments, "last emailed re: X", and a tone signal, then **moves the log** to `raw/processed/Correspondence/`. Mechanism, watermark, and file format: `_automation/auto-inbox.md`.
- **Never file** newsletters, receipts, automated/no-reply, or promotional mail. The feeder is deliberately selective; polluting the wiki with automated mail is worse than missing one thread.
- **Never create a per-email wiki page.** The verbatim already lives in `raw/processed/Correspondence/`; insights distribute onto existing People/Projects/Concepts pages (one thing = one home).

---

## 5. Operation: QUERY

Triggered when the curator asks a question of the wiki.

1. Read `index.md` first to locate candidate pages, then drill into them. (Use `grep`/search over `wiki/` for terms the index doesn't surface.)
2. Synthesize an answer **with citations** back to source files (via the pages' own citations).
3. State plainly when the wiki doesn't contain the answer — don't invent. Offer to find a source or run a web search to fill the gap.
4. **Good answers compound.** If a query produces something worth keeping — a comparison, an analysis, a discovered connection — offer to file it back as a new `concept` page (or a section on an existing one) so it doesn't vanish into chat. Cite the pages it was synthesized from.
5. Optionally log notable queries in `log.md` as `## [YYYY-MM-DD] query | <question>`.

---

## 6. Index & log conventions

**`wiki/index.md`** — content catalog, kept current on every ingest. Organized by the six types; each entry is a link + one-line summary.

```markdown
# Index

## People
- [[jane-doe]] — co-founder, Acme; investor in seed round

## Projects
- [[fundraising-seed]] — *active* — raising seed round

## Concepts
- [[retrieval-augmented-generation]] — RAG techniques and tradeoffs

## Meetings
- [[2026-04-21-acme-intro-call]] — intro call with Jane Doe

## Daily log
- [[2026-06-12]]
```

**`log.md`** — append-only operations ledger. One greppable entry per operation:

```markdown
## [2026-06-12] ingest | Acme intro call transcript
Touched: [[jane-doe]] (new), [[fundraising-seed]], [[2026-04-21-acme-intro-call]] (verbatim). Moved 1 source → processed.
```

Greppable by design: `grep "^## \[" log.md | tail -10` shows recent activity. Prefix is always `## [YYYY-MM-DD] <op> | <title>` where `<op>` ∈ {ingest, query, lint}.

---

## 7. Operation: LINT

Triggered when the curator asks for a health check. Scan `wiki/` and report (do not auto-fix anything destructive without asking):

- **Broken links** — `[[wikilinks]]` pointing to pages that don't exist.
- **Orphans** — pages with no inbound links (candidates for cross-linking or deletion).
- **Duplicates** — two pages describing the same entity (propose a merge; one thing = one home).
- **Contradictions** — claims across pages that disagree; show both with citations.
- **Gaps** — a concept referenced repeatedly but lacking its own page; or "open threads" worth a web search or new source.
- **Missing cross-references** — pages that clearly relate but don't link.
- **Stale claims** — older claims a newer source has superseded.
- **Type drift** — a "concept" page that has grown status/next-steps (should be a project), or a project that's gone inert (should it be `done`/`abandoned`?).
- **Uncited claims** — anything on a page without a `(source: …)`.

Present findings as a checklist; apply fixes the curator approves.

---

## 7b. Name disambiguation (standing)

*(The real vault keeps a list of its actual name collisions here. The entries below are fictional, written to show the shape of the rule.)*

Some entities have confusingly similar names. Maintain this list and never link one to another by name-similarity alone:

- **Atlas** = one of the curator's exploration directions (a project) → `Projects/Atlas.md`. It is *not* a company and *not* the spine. The spine is `Exploration Spine.md` (at the wiki root); Atlas, [[Beacon]], [[Lattice]], and [[Quill]] are sibling experiments that link to it, as does `Accelerator App`.
- **Atlas Chen** = a person → `People/Atlas Chen.md`.
- A source saying "Atlas" refers to the exploration direction; "Atlas Chen" refers to the person. **Never** resolve one to the other's page because the names overlap. When uncertain which is meant, pause and ask.
- **"Atlus"** (appears in some transcriptions) = a misspelling of **Atlas**. **Atlas is the only name** — use it in all synthesized text; never create an "Atlus" page. (Keep verbatim source bodies as-is, but flag.)
- **星语 (Xingyu)** is the correct name of a silent-voice-input product (one transcript rendered it "新语"); use **星语** in all documentation.
- **Rivet** (the company) vs **[[Rivot]]** (the curator's project). **Rivet** = a real recorder company and, since its agentic pivot, this project's **primary competitor** — a source mentioning "Rivet" almost always means the company. **[[Rivot]]** = the curator's own project → `Projects/Rivot.md` (working codename, **one letter off "Rivet"**, high collision risk). Never resolve a "Rivet" mention to the curator's project unless context makes the ownership unmistakable; when uncertain, it's the company.

(Add new collisions here as they surface.)

## 8. Conventions summary

- **Links:** Obsidian `[[wikilinks]]` by page filename (without `.md`). Link liberally; a link to a not-yet-existing page is a fine TODO marker.
- **Citations:** `(source: raw/processed/<subfolder>/<file>)` inline after the claim — use the full processed subpath (processed mirrors inbox, see §1). Images cite the asset path under `raw/processed/assets/`. Records are cited as wikilinks, e.g. `[[Incorporation]]`.
- **Dates:** ISO `YYYY-MM-DD` everywhere (filenames, frontmatter, log).
- **Filenames: Title Case with spaces.** Entities/concepts/projects/records → `Atlas Chen.md`, `Exploration Spine.md`, `AI Hardware Wearables.md`. Acronyms stay uppercase (AI, YC, LLM, MCP, FDE). Dated pages keep an ISO prefix → `2026-04-21 Acme Intro Call.md`. Daily logs stay bare dates → `2026-05-29.md`.
- **Frontmatter:** every page carries it (Dataview-ready). Keep `updated` and `sources` current.
- **Verbatim means verbatim:** meetings and daily logs are never edited; insights flow *out* to other pages, the originals stay intact.
- **`_automation/` is tool config, not wiki knowledge.** `_automation/voice-profile.md` is the email drafter's global voice guide + tone-inference rubric — *synthesized*, not verbatim, so it is NOT a `wiki/` page and NOT a `record`. Don't "correct" it into the wiki. Per-contact tone lives on People pages only as optional `## Email tone` overrides (§2.1); by default the drafter *infers* tone rather than reading a hardcoded label.
