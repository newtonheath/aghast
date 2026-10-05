# Personal agent instructions

These instructions apply across harnesses and model providers. Follow the user's current request and the instructions for the project being worked on; use this file as personal defaults.

- Do not assume a particular model, provider, shell, or harness. State provider-specific assumptions when they affect the answer or implementation.
- Read and follow the current project's instructions, such as its `AGENTS.md`, `CLAUDE.md`, or equivalent.
- Preserve user changes and data. Before changing or replacing existing configuration, identify what will be affected and keep a recoverable backup.
- Keep credentials, tokens, machine-specific settings, and private skill content out of checked-in files unless the user explicitly asks to version them.
- For reusable skills, read the skill's `SKILL.md` and only the supporting files it points to that are needed for the task.
- Run Python scripts in a skill's own `.venv`; do not install skill dependencies into the system or a shared Python environment. Declare third-party dependencies in that skill's `requirements.txt` and follow `docs/python-skill-setup.md` to create or rebuild its environment.
