# Aghast

**Agent Generic Harness for AGENTS.md, Skills & Tools**

Aghast keeps personal instructions and reusable agent skills in one Git checkout, then links each supported harness to those source files. The model provider can change independently: Aghast manages the harness context, not API keys or model endpoints.

## Install

Clone this repository somewhere you intend to keep it, then run the installer from the checkout:

```sh
./aghast install --dry-run
./aghast install
```

The installer uses absolute symlinks to this checkout. If you move the checkout, run `./aghast install` again to update them.

On first install, existing `~/.claude` and Codex home directories (`~/.codex` by default) are copied beside themselves to timestamped `*.aghast-backup-*` paths. The original directories stay in place. Live Unix sockets are skipped because they cannot be copied and are recreated by their applications as needed. The installer links `CLAUDE.md` and `AGENTS.md` to this checkout, leaving the other files, settings, history, caches, credentials, and plugins at their existing paths. It adds a small `.aghast-managed` marker and the skill links described below. A full copy can take extra disk space; re-running install does not make another full copy of an already managed harness directory. OpenCode's config directory is kept in place; an existing global `AGENTS.md` is backed up before it is linked.

Existing skill directories Aghast manages, `~/.agents/skills` and `~/.claude/skills`, are also included in the initial snapshots before Aghast adds links. The `~/.claude/skills` directory is covered by the full Claude snapshot when it is a regular directory; if it is a symlink, Aghast copies its target contents separately. `~/.agents/skills` gets its own copy. The active skill directories stay in place. Aghast adds one symlink per skill there. If a skill name already exists, that entry is moved into a sibling `*.aghast-backups/` directory outside skill discovery so the Aghast link can use the name.

Each link points to the whole skill directory so its references, scripts, and assets stay together. Codex has a reported discovery limitation when only `SKILL.md` is symlinked ([issue](https://github.com/openai/codex/issues/17344)).

## What gets linked

| Harness | Instructions | Skills |
| --- | --- | --- |
| Codex | `~/.codex/AGENTS.md` by default | `~/.agents/skills/<skill>` |
| Claude Code | `~/.claude/CLAUDE.md` | `~/.claude/skills/<skill>` |
| OpenCode | `~/.config/opencode/AGENTS.md` | `~/.agents/skills/<skill>` |

All three instruction files point to the checkout's root `AGENTS.md`. Codex and OpenCode share the Aghast skill links under `~/.agents/skills`; Claude Code receives links under its native skill directory.

Project-level instructions remain in each project's own `AGENTS.md` (and any harness-specific project instruction files). Aghast installs only user-level guidance.

## Skills

Put skills you want to track with Aghast in `skills/<skill-name>/SKILL.md`. A skill can include supporting files such as `references/`, `scripts/`, and `assets/`.

To install a skill from another directory without checking its files into this repository:

```sh
./aghast skill add /path/to/my-skill
```

This copies the skill into `~/.local/share/aghast/skills/<skill-name>/` by default and links it into the harness skill directories. Any `.venv` directory in the source is excluded; create the environment in the copied skill directory using [`docs/python-skill-setup.md`](docs/python-skill-setup.md). The source folder can then be moved independently. Use `./aghast install` after adding or removing tracked skills so the links stay in sync.

Skill folder names must be lowercase kebab-case and contain a `SKILL.md` file. Aghast rejects duplicate names between checked-in and locally installed skills.

## Commands

```sh
./aghast install [--dry-run]  # copy existing Claude/Codex homes and link Aghast files
./aghast status               # show managed links and local skill installs
./aghast skill add PATH       # copy an external skill into Aghast's local skill store
```

By default, Aghast stores private skills and link state under `~/.local/share/aghast`, and uses `~/.codex` as the Codex home. Optional custom-location settings are available, but most setups can use these home-directory defaults without configuring anything.

## Backups and Git

Initial Claude and Codex snapshots are copies outside this repository, beside the original directories. Replacing an existing instruction file is covered by that full snapshot. Other file and skill-name conflicts are backed up outside their active discovery paths. Review backup names printed by the installer before restoring files manually.

Local skills added with `aghast skill add` are stored outside the checkout and are never committed by Aghast. Tracked skills under `skills/` are normal Git content.
