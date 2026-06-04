# CLAUDE.md — Faceless AI-News YouTube Channel (Agent Pipeline)

This file is the **coordinator** for an AI-news YouTube channel built as a Claude
Code agent pipeline. The main Claude Code session reads this file and orchestrates
three subagents to take a video from idea → script → edit plan.

> **Audience & format:** Faceless AI-news channel for a **US audience**, covering AI
> tools, model launches, and "how to use AI / make money with AI." Production is
> screen recordings + stock b-roll + AI voiceover — **no on-camera host**.

---

## Repository layout

```
.
├── CLAUDE.md                       # ← you are here (the coordinator)
├── README.md
├── .claude/
│   ├── agents/
│   │   ├── yt-researcher.md         # Stage 1 — find the next topic (5 ranked ideas)
│   │   ├── yt-scriptwriter.md       # Stage 2 — write the full VO script
│   │   ├── yt-editor.md             # Stage 3 — edit plan + SRT + (optional) rough cut
│   │   └── youtube-ai-news-research.md  # legacy single-shot researcher (see note)
│   └── commands/
│       └── next-video.md            # legacy slash command → youtube-ai-news-research
├── research/                        # Stage 1 output: dated topic briefs (*.md)
├── scripts/                         # Stage 2 output: full VO scripts (*.md)
├── assets/
│   └── clips/                       # YOU drop b-roll/screen-recording clips here
│   └── voiceover.mp3                # YOU drop the generated narration here (not committed)
└── output/                          # Stage 3 output: edit plans, *.srt, rough-cut *.mp4
```

There is **no application code** in this repo — it is a Claude Code configuration
project. The "product" is the agents, this coordinator, and the artifacts they write
into `research/`, `scripts/`, and `output/`.

---

## The pipeline (run-order)

```
yt-researcher  →  [user picks an idea]  →  yt-scriptwriter  →  yt-editor
   research/<date>-<slug>.md               scripts/<slug>.md     output/<slug>-*
```

| Stage | Agent | Reads | Writes | Tools |
|------|-------|-------|--------|-------|
| 1. Research | `yt-researcher` | web | `research/<YYYY-MM-DD>-<slug>.md` | WebSearch, WebFetch, Write |
| 2. Script | `yt-scriptwriter` | `research/*.md` | `scripts/<slug>.md` | Read, Write, WebSearch |
| 3. Edit | `yt-editor` | `scripts/*.md`, `assets/*` | `output/<slug>-editplan.md`, `output/<slug>.srt`, `output/<slug>-roughcut.mp4` | Read, Write, Glob, Bash |

**Slug convention:** the researcher picks a short kebab-case slug for the chosen
idea; the scriptwriter and editor reuse the **same slug** so artifacts line up across
`research/`, `scripts/`, and `output/`.

---

## How to run (what the coordinator should do)

When the user says **"run the pipeline"**, **"make my next video"**, or similar:

1. **Research** — Launch `yt-researcher`. It web-searches the last 7 days, returns 5
   ranked ideas + a `MAKE THIS FIRST →` pick, and (after a choice) saves a brief to
   `research/`.
2. **Pick** — Ask the user which idea to build.
   - If they say **"auto"** (e.g. *"make my next video — auto"*) or don't choose,
     default to the `MAKE THIS FIRST` pick and proceed without waiting.
3. **Script** — Launch `yt-scriptwriter`, passing the brief path from Stage 1. It
   writes the full VO script to `scripts/`.
4. **Edit** — Launch `yt-editor`, passing the script path. It always produces an edit
   plan + SRT, and assembles a rough cut **only if** `assets/voiceover.mp3` and clips
   in `assets/clips/` already exist.
5. **Report** — Tell the user what was produced (paths) and **what's left** — the
   manual steps (voiceover + b-roll) the agents can't do.

Run stages sequentially (each depends on the previous one's output). Within a stage,
the agent may parallelize its own web searches.

---

## HONEST NOTES — read this before expecting magic

- **Research + scripting are fully automated.** They work today, no extra cost, no
  API keys (just WebSearch).
- **Editing is the hard part.** A Claude Code agent **cannot** generate voiceover or
  create/download b-roll by itself. So `yt-editor` does three *real* things:
  1. a shot-by-shot **edit plan**,
  2. timed **captions (SRT)**, and
  3. **only if** you've already dropped `assets/voiceover.mp3` + clips into
     `assets/clips/`, it stitches a **rough cut** with ffmpeg.

  It does **not** do creative editing or produce a polished, publish-ready video.

For the pieces the agent can't do, two paths:

- **Manual (recommended to start):** generate the voiceover yourself (e.g.
  ElevenLabs), record/grab b-roll, drop both into `assets/`, then let `yt-editor`
  assemble the rough cut. Polish in a normal editor.
- **Fuller automation (later):** wire in paid APIs (TTS + an AI video generator) so
  the editor can pull voice and footage too. That needs API keys, costs money per
  video, and is a real build — only worth it once the channel is working.

**Don't over-engineer before the channel proves out.** Get 5–10 videos live with the
manual path first; automate the expensive last mile only if it's working.

---

## Conventions for AI assistants

- **Never fabricate** a launch, benchmark, date, price, or quote. Verify against web
  search; if it can't be verified, drop it or mark it `⚠️ verify`.
- **Keep artifacts in their folders** and reuse the slug so a video's files line up.
- **Write for the ear** in scripts — short sentences, contractions, ~150 wpm.
- **Stay honest about limits** in the edit stage; never claim a polished video was
  produced when only a plan/SRT/rough-cut exists.
- Treat anything older than ~7 days as context, not "fresh news."
- `assets/voiceover.mp3` and rendered media are user-supplied / generated — don't
  commit large binaries; keep the repo to config + text artifacts.

### Legacy files
`.claude/agents/youtube-ai-news-research.md` and `.claude/commands/next-video.md` are
the original single-shot researcher (7 ideas) and its `/next-video` command. They
still work standalone, but the **3-agent pipeline above is the current setup**.
`yt-researcher` supersedes the legacy researcher for end-to-end runs.

---

## Setup & requirements

One-time setup (already scaffolded in this repo):

1. `CLAUDE.md` in the project root.
2. `.claude/agents/` holding the three pipeline agents.
3. Empty working folders: `research/`, `scripts/`, `assets/clips/`, `output/`.
4. Open the folder in Claude Code and **restart the session** (agents load at
   startup). Run `/agents` to confirm `yt-researcher`, `yt-scriptwriter`, and
   `yt-editor` all appear.

Requirements:

- **Claude Code** with **WebSearch enabled** (for the researcher/scriptwriter).
- **ffmpeg** installed locally (for the editor's optional rough-cut assembly).
