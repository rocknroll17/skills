# paper-study

A drop-in project template for **reading a research paper with Claude Code**.

Spin up a new folder, point it at an arXiv ID, and Claude turns into a dedicated briefing agent for that paper — with pre-wired slash commands for summarizing sections, explaining equations, comparing prior work, and building a glossary as you go.

> 한국어 사용 안내는 [README.ko.md](./README.ko.md)

---

## What you get

When you clone this template into an empty folder:

- `CLAUDE.md` — auto-loaded every session. Holds paper metadata, a user profile, and the style rules Claude follows (briefing mode: concise, conclusion-first, jargon-aware).
- `notes/` — skeleton files (`00-overview.md` … `05-limitations.md`). Claude fills them as you ask about each section.
- `glossary/terms.md` — grows every time a new term appears.
- `.claude/skills/` — six slash commands, paper-agnostic.

Claude reads the PDF, populates `CLAUDE.md` metadata, writes a one-line overview, and waits for your first question.

## Requirements

- [Claude Code CLI](https://code.claude.com/docs) ≥ 1.0
- `git`, `curl`
- `pdfinfo` (optional, from `poppler-utils`) — used by `/new-paper` to extract title/author

## Quickstart

```bash
# 1) Copy the template into a new folder for your paper
git clone https://github.com/rocknroll17/skills.git /tmp/skills-src
cp -r /tmp/skills-src/paper-study ~/my-paper
cd ~/my-paper

# 2) Launch Claude Code
claude
```

Inside Claude, run:

```
/new-paper 2508.00298
```

That's it. Claude downloads the PDF, extracts metadata, writes a one-line overview into `notes/00-overview.md`, and asks where you'd like to start.

Local PDF instead of arXiv ID:

```
/new-paper ./mypaper.pdf
```

### One-liner install (without cloning the full repo)

```bash
mkdir ~/my-paper && cd ~/my-paper
curl -sL https://api.github.com/repos/rocknroll17/skills/tarball \
  | tar xz --strip=2 --wildcards '*/paper-study/*'
```

## Slash commands

| Command | What it does |
| --- | --- |
| `/new-paper <arxiv_id \| pdf_path \| url>` | Bootstrap the environment for a new paper |
| `/summarize-section <name>` | 5-layer briefing of a section (TL;DR, technical, deep dive, critique, genealogy) |
| `/explain-equation <eq>` | 3-step equation walkthrough (symbols → intuition → tiny example) |
| `/compare-prior <paper>` | Comparison table vs. one or more prior works |
| `/glossary <term \| list>` | Add/look up a term in the glossary |
| `/quiz <topic>` | Bloom-taxonomy quiz to self-check understanding *(only when you ask)* |

## Directory layout

```
my-paper/
├── CLAUDE.md
├── pdf/              # paper PDFs live here
├── notes/            # 00-overview.md … 05-limitations.md
├── glossary/terms.md
└── .claude/skills/   # six slash commands
```

## Customizing the tutor

The default `CLAUDE.md` assumes:

- You are **new to the paper's field** (the tutor introduces jargon inline).
- You want **briefing-style** answers in Korean — short, conclusion-first, no self-appointed Socratic quizzes.

To change language, tone, or depth, just edit `CLAUDE.md`:

- Section **§2 User** — your background and preferred language.
- Section **§3–§7 Style rules** — tone, jargon policy, length, what to avoid.
- Section **§11 Working principles** — whether the tutor may ask follow-up questions on its own.

Claude reloads the file each session, so edits are live immediately.

## Updating the template

```bash
cd /tmp/skills-src && git pull
```

Existing per-paper folders are untouched. Your next `/new-paper` uses the refreshed template.

## How it compares

| Approach | Pros | Cons |
| --- | --- | --- |
| `paper-study` (this repo) | Zero setup, works in any folder, skills bundled | Manual clone/copy step |
| Claude Code plugin | One-command install from a marketplace | Doesn't scaffold project files on install |
| Plain chat + PDF upload | No setup | No persistent notes, no glossary, no skills |

## Contributing

Issues and PRs welcome. See `skills/` root for other templates.

## License

MIT — see [LICENSE](../LICENSE) at the repo root.

## Credits

Style rules distilled from published work on LLM tutoring, cognitive-load theory, and practitioner guides (Feynman technique, Socratic prompting, Bloom's taxonomy). The **briefing mode** default adopts a "staff-to-professor" framing — Claude does the reading and reports back, instead of running a classroom.
