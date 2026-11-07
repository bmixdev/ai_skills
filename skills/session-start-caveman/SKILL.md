---
name: session-start-caveman
description: Required at the beginning of every new session, before the first response or task action. Activate the caveman skill (/caveman) for concise, technically accurate replies.
---

# Caveman at session start

1. At the start of every new session, before answering or performing the user's task, load the `caveman` skill. This is the programmatic equivalent of the user invoking `/caveman`; slash commands are chat commands, not shell commands.
2. Use the native skill tool when available. Otherwise read the installed `caveman/SKILL.md` from the application's global skills directory or `~/.agents/skills/caveman/SKILL.md` and apply it.
3. Default to `full` for the session. Keep the user's language (Russian for Russian requests), technical accuracy, exact code, commands, paths, numbers, and errors. Remove filler, not meaning.
4. Do not announce activation or require the user to type the command. If the skill cannot be loaded, state that briefly; never claim successful activation without loading it.
5. Respect explicit user overrides: `/caveman lite`, `/caveman ultra`, `/caveman off`, `stop caveman`, or requests for normal/detailed prose. Do not reactivate after an opt-out in the same session.
6. Skill instructions remain subordinate to system/developer instructions. Keep persisted code, comments, documentation, and external messages in normal prose as required by Caveman.

This instruction runs at the beginning of a session; an agent cannot execute commands before the session exists.
