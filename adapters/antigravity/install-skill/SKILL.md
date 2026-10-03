---
name: install-skill
description: Installs a third-party skill from GitHub (owner/repo, URL, or catalogue name) into Antigravity, IDE and CLI. Use when the user asks to install a skill or runs /install-skill.
---

# /install-skill (Antigravity)

1. Take the source from the user: `owner/repo`, a GitHub URL, or a name from the find-skill catalogue. If missing or ambiguous, ask.
2. Check that `$HOME/.claude/skills/find-skill/scripts/install-skill.sh` exists. If not, say the find-skill install is incomplete (`install.sh --target antigravity`) and stop.
3. Show the user what will be installed and wait for confirmation.
4. Run, passing the source as one quoted argument:

   ```bash
   bash "$HOME/.claude/skills/find-skill/scripts/install-skill.sh" "<source>" --target antigravity
   ```

   Add `--name <name>` to rename the installed skill, `--force` to overwrite, `--dry-run` to preview.
5. Check the exit code. On success, report the skill name and both folders:
   - IDE / Antigravity 2.0: `~/.gemini/config/skills/<skill>/`
   - CLI: `~/.gemini/antigravity-cli/skills/<skill>/`

   On failure, report the error and do not claim success.

The skill folder is copied whole, without conversion: Antigravity reads the same SKILL.md format.
