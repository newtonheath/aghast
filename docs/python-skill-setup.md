# Python setup for skills

Use one virtual environment per skill that runs Python. Keep `.venv` in the skill root, next to `SKILL.md`, so the skill's scripts use their own Python packages rather than packages installed for another skill or for the host system.

## Skill location

- A checked-in skill lives at `<aghast-checkout>/skills/<skill-name>/`.
- A private skill installed with `./aghast skill add` lives at `~/.local/share/aghast/skills/<skill-name>/` by default.

Run the setup commands from that skill's root directory, the directory containing `SKILL.md`. When using `aghast skill add`, Aghast copies the skill but excludes `.venv`; set up the environment in the installed copy.

You do not need to configure a storage variable for the default location. Aghast supports optional custom locations, but the steps below are the same if you use one.

## Declare dependencies

Put third-party Python dependencies in a `requirements.txt` at the skill root. Pin versions that the skill has been tested with, for example:

```text
requests==2.32.3
```

Do not add a `requirements.txt` if the scripts use only Python's standard library. If the skill requires a particular Python version, say so in its `SKILL.md` and use that version when creating the environment.

## Create the environment

From a POSIX shell, in the skill root:

```sh
python3 --version
python3 -m venv .venv
```

If the skill requires a specific installed version, use its executable instead, such as `python3.12 -m venv .venv`.

Install dependencies with the environment's interpreter so `pip` cannot target a different Python:

```sh
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install -r requirements.txt
```

Skip the last command when the skill has no `requirements.txt`. To confirm which interpreter the environment uses:

```sh
.venv/bin/python -c 'import sys; print(sys.executable)'
```

## Run scripts

Prefer calling the environment's Python directly, from the skill root:

```sh
.venv/bin/python scripts/example.py
```

This avoids accidentally using a global Python even if a different environment is active in the shell. Skill instructions that ask an agent to run a Python script should name the skill-local interpreter (or explain how to locate it) instead of using a bare `python` command.

## Keep environments local

`.venv` is machine-specific and can be recreated from the dependency file. Aghast's `.gitignore` excludes `.venv` from checked-in skills, and `aghast skill add` omits `.venv` when it copies a private skill. Never copy or commit a virtual environment; create it in the skill directory where its scripts will run.
