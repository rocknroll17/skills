# skills

Claude Code plugin marketplace for research and study workflows.

## Install the marketplace

```
/plugin marketplace add rocknroll17/skills
```

## Available plugins

| Plugin | What it does |
| --- | --- |
| [`paper-study`](./paper-study) | Drop-in briefing agent for reading a research paper — scaffolds `CLAUDE.md`, `notes/`, `glossary/`, and bundles paper-specific slash commands. |

## Install a plugin

```
/plugin marketplace add rocknroll17/skills
/plugin install paper-study@rocknroll17-skills
```

Then, from any folder, pass an arXiv ID, a URL, or a local PDF — each paper gets its own keyword-named subfolder:

```
/paper-study:new-paper XXXX.XXXXX                        # arXiv ID
/paper-study:new-paper https://arxiv.org/abs/XXXX.XXXXX  # arXiv / PDF URL
/paper-study:new-paper ./mypaper.pdf                     # local PDF
```

## Update an installed plugin

```
/plugin marketplace update rocknroll17-skills
/plugin update paper-study@rocknroll17-skills
/reload-plugins
```

Refresh the catalog, install the new version, then reload it into the running session — no restart needed. Or enable auto-update: `/plugin` → **Marketplaces** → **Enable auto-update** ([docs](https://code.claude.com/docs/en/discover-plugins#configure-auto-updates)).

See each plugin's README for details.

## Repository layout

```
skills/
├── .claude-plugin/
│   └── marketplace.json     # this repo as a marketplace
├── paper-study/             # plugin 1
│   ├── .claude-plugin/plugin.json
│   ├── skills/              # slash commands shipped by this plugin
│   │   └── new-paper/templates/   # scaffold files copied into each paper folder
│   ├── README.md
│   └── README.ko.md
└── LICENSE
```

## License

MIT — see [LICENSE](./LICENSE).
