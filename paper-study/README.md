# paper-study

A Claude Code plugin that turns any empty folder into a **briefing agent for a specific research paper**.

Install it, run `/paper-study:new-paper <arxiv_id>` in an empty folder, and Claude downloads the PDF, scaffolds `CLAUDE.md`, `notes/`, and `glossary/`, then waits for your first question.

> 한국어 사용 안내: [README.ko.md](./README.ko.md)

---

## Install

```
/plugin marketplace add rocknroll17/skills
/plugin install paper-study@rocknroll17-skills
```

Then from any empty folder:

```
/paper-study:new-paper 2508.00298
```

Local PDF works too:

```
/paper-study:new-paper ./mypaper.pdf
```

## Requirements

- [Claude Code CLI](https://code.claude.com/docs) ≥ 1.0 (with plugin support)
- `curl`
- `pdfinfo` (optional — from `poppler-utils`, used to extract title/author)

## What `/paper-study:new-paper` does

1. Downloads the PDF into `./pdf/paper.pdf`
2. Creates `CLAUDE.md`, `notes/` (6 skeletons), `glossary/terms.md` in the current folder
3. Extracts title / authors / venue and fills `CLAUDE.md` §1
4. Reads the first ~4 pages and writes a one-line overview into `notes/00-overview.md`
5. Extracts figures into `figures/` (run as a background job)
6. **Calibrates to you** — while figures extract, it asks a few questions to gauge your background, then sets per-concept explanation depth in `CLAUDE.md` §2
7. Fully analyzes every section into `notes/` + `glossary/`, then reports back

`CLAUDE.md` is auto-loaded every session; Claude now speaks in **briefing mode** tuned to this paper and your level.

## Slash commands (all scoped under `paper-study:`)

| Command | What it does |
| --- | --- |
| `new-paper <arxiv_id \| pdf_path \| url>` | Bootstrap a paper environment in CWD |
| `explain-equation <eq>` | 5-block equation walkthrough (intuition → symbols → diagram → counterfactual → worked example) |
| `glossary <term \| list>` | Add/look up a term in the glossary |

## What gets scaffolded in your folder

```
my-paper/
├── CLAUDE.md                 # tutor personality; auto-loaded
├── pdf/paper.pdf             # the paper
├── notes/                    # 00-overview.md … 06-limitations.md
└── glossary/terms.md         # grows as new terms appear
```

## Customizing the tutor

`CLAUDE.md` is the tutor's personality. Defaults:

- Answers in **Korean**, briefing style (short, conclusion-first).
- A **reader profile** (`§2`) is filled by the calibration step and drives per-concept explanation depth.

Edit these sections in your project's `CLAUDE.md`:

- `§2 Reader profile` — your field/formal-tools level and per-concept depth
- `§3–§9` — tone, jargon policy, analogy use, length, what to avoid
- `§13` — whether the tutor may ask follow-up questions on its own

Claude reloads the file every session, so edits are live immediately.

## Install without the plugin system

If your Claude Code CLI doesn't have plugin support, or you just want a copy to modify:

```bash
git clone https://github.com/rocknroll17/skills.git /tmp/skills-src
cp -r /tmp/skills-src/paper-study/skills/new-paper/templates/. ~/my-paper
cp -r /tmp/skills-src/paper-study/skills ~/my-paper/.claude/skills
cd ~/my-paper
claude
```

`skills/new-paper/templates/` holds the same files `/paper-study:new-paper` writes (single source of truth). Copy them directly and everything works — the slash commands live under `.claude/skills/` exactly like plugin-distributed ones.

## Update

```
/plugin marketplace update rocknroll17-skills
```

Your existing per-paper folders are untouched. The next `/paper-study:new-paper` uses the refreshed scaffold.

## How it compares

| Approach | Pros | Cons |
| --- | --- | --- |
| `paper-study` plugin | One-command install, scoped slash commands, MCP-like feel | Requires plugin-capable Claude Code |
| `paper-study` cloned manually | Works anywhere, easy to fork | Copy step is manual |
| Plain chat + PDF upload | Zero setup | No persistent notes, glossary, or skills |

## Contributing

Issues and PRs welcome at https://github.com/rocknroll17/skills.

## License

MIT — see [LICENSE](../LICENSE) at the repo root.

## Credits

Style rules distilled from work on LLM tutoring and cognitive-load theory. The **briefing mode** default adopts a staff-to-professor framing — Claude does the reading and reports back, instead of running a classroom.
