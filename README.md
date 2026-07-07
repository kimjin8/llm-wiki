# I Wired Karpathy's LLM Wiki to Real Life

*Kim Jin, July 2026*

I record most of my working life: meetings, customer calls, events, thinking-out-loud notes. Until recently none of my AI tools knew any of it.

Andrej Karpathy sketched an idea I could not stop thinking about: a personal wiki that an LLM maintains for you ([his idea file](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)). Most people who liked it moved on. I had a problem it fit exactly, so I built the pipeline. A month in, I run my life on it.

## The pipeline

```
  wearable recorder ──┐
                      ├──► inbox/ ──► LLM librarian (nightly ingest)
  Gmail (email feeder)┘                        │
                                               ▼
        ┌───────────────┬────────────────┬──────────────┐
    verbatim         distributed        index +        review
    records          insights           ops log        queue
   (meetings,      (people, project,                 (questions
    daily logs)     concept pages)                    for me)
```

Audio auto-uploads from the recorder and lands as a transcript in an inbox folder. A second feeder drops in the day's noteworthy email. An LLM librarian files each source into a plain-markdown wiki every night. My other agents read that wiki.

## The rules that make it work

1. **Two classes of pages.** Meetings and daily logs are filed verbatim, never summarized. Insights get distributed outward onto people, project, and concept pages. The original is sacred; the synthesis is disposable.
2. **Every claim cites its source file.** When the wiki tells me something surprising, I can open the exact transcript it came from.
3. **One thing, one home.** The librarian updates an existing page instead of spawning a duplicate. A single conversation might touch ten pages.
4. **Ask, don't guess.** Ambiguities go to a review queue for me to resolve rather than getting a confident wrong answer. My favorite failure: a project codename one letter away from a friend's name. The schema now carries a standing disambiguation list.
5. **I never write wiki pages by hand.** I curate sources and ask questions. The librarian does all the filing, cross-referencing, and bookkeeping.

## What it is like to use

Before a call, "brief me on everyone in this meeting" pulls from every prior conversation with them. After a week of events, "who should I follow up with, and why" comes back with citations. And my email now drafts itself: each morning the wiki grounds replies written in my voice, so a draft starts from what a person actually told me last time instead of from a blank box. Anything I write, an investor update, an application, a competitive read, starts from ground truth instead of memory.

A month of daily use: more than 200 recorded sources compiled into a web of 160+ interlinked pages, every claim tied back to its transcript (500+ citations so far), and not one page written by hand.

## Primitive, on purpose

It is a folder of markdown files, one instruction file that holds the schema, and a scheduled nightly ingest. No vector database, no app, no UI beyond Obsidian. The leverage is in the librarian's rules, not the infrastructure. The failure modes are mundane, transcription typos and name collisions, and the fixes are editorial rather than architectural.

## Why I built it

I am exploring ambient capture as a startup direction, on the belief that AI becomes truly personal only when it has the context of your physical life. This wiki is my n-of-1 prototype, and it taught me where the real bottleneck is. It is not the model and it is not the schema. It is capture: getting the physical world into the system without friction. That is a hardware problem, and it is the one I want to solve next.

---

*This repo contains the public write-up, a sanitized copy of the schema the librarian follows (`schema.md`), and a synthetic example pair (`examples/`). The actual vault stays local: it is full of real names and never leaves my disk.*
