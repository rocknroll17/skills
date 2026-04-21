# skills

Claude Code plugin marketplace for research and study workflows.

## Install the marketplace

```
/plugin marketplace add rocknroll17/skills
```

## Available plugins

| Plugin | What it does |
| --- | --- |
| [`paper-study`](./paper-study) | Drop-in briefing agent for reading a research paper — scaffolds `CLAUDE.md`, `notes/`, `glossary/`, and bundles six paper-specific slash commands. |

## Install a plugin

```
/plugin install paper-study@rocknroll17-skills
```

Then, in an empty folder:

```
/paper-study:new-paper 2508.00298
```

See each plugin's README for details.

## Repository layout

```
skills/
├── .claude-plugin/
│   └── marketplace.json     # this repo as a marketplace
├── paper-study/             # plugin 1
│   ├── .claude-plugin/plugin.json
│   ├── skills/              # slash commands shipped by this plugin
│   ├── scaffold/            # template files (also embedded in /new-paper skill)
│   ├── README.md
│   └── README.ko.md
└── LICENSE
```

## License

MIT — see [LICENSE](./LICENSE).
