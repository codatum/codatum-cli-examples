---
name: cdm
description: Use this skill whenever you are about to read or edit any *.cnb.md notebook file. It verifies the cdm CLI and PAT are usable, then loads the cdm agent workflow guide before any change is made.
---

# Editing cdm notebook files

This skill applies whenever you are about to read or edit a `*.cnb.md` (Codatum notebook) file. A `*.cnb.md` file is **not** plain Markdown — it has its own schema and directives — so always go through the cdm CLI and the workflow guide instead of editing it like an ordinary Markdown file.

## Steps

1. Verify that the `cdm` command is available and the PAT is valid:

   ```sh
   cdm --version
   cdm auth whoami
   ```

   - If `cdm --version` fails with "command not found", **stop** and ask the user to install `cdm`. Do **not** try to install it yourself. Point them to the "Getting Started" page in the cdm docs.
   - If `cdm auth whoami` exits with a non-zero status (no profile registered, or the PAT is invalid), **stop** and ask the user to register / re-register the PAT via `cdm auth login`.

   Do not start editing until **both** commands succeed.

2. Fetch the agent workflow guide via the cdm CLI:

   ```sh
   cdm doc show /guide/for-agents.md
   ```

3. Follow the workflow defined in that guide — it is the source of truth and will point you to additional doc pages (`cdm doc show <path>`) for the specific edit you're making (chart settings, grid layout, SQL blocks, parameters, resource IDs, etc.). Note that some commands require **explicit user consent** before you run them (server writes, expensive bulk runs); follow the consent rules in the guide and never run them unprompted.

4. After editing, run `cdm notebook format <file>` to auto-repair and validate the file. Never hand-format a `*.cnb.md` file — its format differs from ordinary Markdown.
