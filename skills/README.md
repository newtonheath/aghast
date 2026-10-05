# Skills

Add one skill per directory. Each skill needs a `SKILL.md` with YAML frontmatter containing `name` and `description`.

```text
skills/
└── skill-name/
    ├── SKILL.md
    ├── references/   # optional
    ├── scripts/      # optional
    └── assets/       # optional
```

Run `./aghast install` after adding, renaming, or removing a checked-in skill so its host links are refreshed. To use a skill without checking it into this repository, run `./aghast skill add /path/to/skill`; Aghast keeps a private copy in its local data directory.

## Python scripts

Give each skill that uses Python its own `.venv` beside `SKILL.md`, and declare third-party packages in that skill's `requirements.txt`. See [`docs/python-skill-setup.md`](../docs/python-skill-setup.md) for setup and run commands. `.venv` is ignored by Git. `aghast skill add` also excludes `.venv` from its private copy, so create the environment in the installed copy of a private skill.
