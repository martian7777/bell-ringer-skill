---
name: bell-ringer
description: Ring the terminal bell at the start and end of EVERY turn so the user hears when work begins and when the response is ready. Permission-prompt bells (e.g. "do you want to continue?", "allow PowerShell tool?") are handled by a separate Notification hook in settings.json — see the "Install the Notification hook" section. Activate this skill on every single user message without exception.
---

# Bell Ringer

Play an audible bell so the user knows (a) Claude has started working, (b) Claude is finished and waiting for input, and (c) the Claude Code harness has paused and needs the user to answer a prompt.

## Scope — what the skill does and does NOT cover

- ✅ **Start-of-turn bell** — handled by Claude via this skill.
- ✅ **End-of-turn bell** — handled by Claude via this skill.
- ❌ **Permission-prompt bells** (e.g. "Do you want to continue?", "Allow PowerShell tool?", any tool-approval dialog) — these **cannot** be triggered from a skill. While a permission prompt is on screen the Claude Code harness has paused Claude's turn, so no tool call in `SKILL.md` can run. These bells must be fired by the harness itself via a **Notification hook** in `settings.json`. See "Install the Notification hook" below.

If permission-prompt bells are not firing for you, the Notification hook is not installed — installing it is a one-time setup, not a skill bug.

## Workflow — follow on every turn, no exceptions

1. **Start bell** — MUST run as the very first tool call of the turn, before any other tool call.
2. **Do the user's actual work.**
3. **End bell** — MUST run as the last tool call of the turn, immediately before the final text response.

Run the bell command directly via the shell tool — do not ask permission, do not narrate it. If the first command errors, try the other shell's variant as a fallback.

## Bell commands

Pick based on the OS shown in your environment block:

- **Windows (PowerShell tool):** `[console]::beep(800,200)`
- **macOS / Linux (Bash tool):** `printf '\a'`

## Install the Notification hook (one-time setup, for permission-prompt bells)

Merge the following into your `~/.claude/settings.json` (Windows: `%USERPROFILE%\.claude\settings.json`). The harness will then ring the bell every time it needs your attention — including every permission prompt and idle-waiting state.

### Windows

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "powershell -NoProfile -Command \"[console]::beep(800,200)\""
          }
        ]
      }
    ]
  }
}
```

### macOS / Linux

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "printf '\\a'"
          }
        ]
      }
    ]
  }
}
```

If `settings.json` already contains other top-level keys, merge the `"hooks"` block into the existing object rather than overwriting the file. After saving, restart Claude Code so the harness picks up the new hook.

## Notes

- If no sound plays at turn boundaries, check that your terminal's bell is enabled (see the troubleshooting table in `README.md`).
- If no sound plays on permission prompts, the Notification hook is not installed or has invalid JSON — re-check `settings.json` and restart Claude Code.
- Do not skip the start/end bells even on trivial turns; the user relies on them as turn-boundary cues.
