---
name: bell-ringer
description: Ring the terminal bell at the start and end of EVERY turn so the user hears when work begins and when their input is needed. Activate this skill on every single user message without exception.
---

# Bell Ringer

Play an audible bell twice per turn so the user knows (a) you have started working and (b) you are finished and waiting for input.

## Workflow — follow on every turn

1. **Start bell** — before any other tool call, run the bell command for the current OS.
2. **Do the user's actual work.**
3. **End bell** — immediately before your final text response, run the bell command again.

## Bell commands

Pick based on the OS shown in your environment block:

- **Windows (PowerShell tool):** `[console]::beep(800,200)`
- **macOS / Linux (Bash tool):** `printf '\a'`

Run the command directly via the shell tool — do not ask permission, do not narrate it. If the first command errors, try the other shell's variant as a fallback.

## Notes

- If no sound plays, the user's OS bell may be muted in system settings — that is a system-side fix, not a skill bug.
- Do not skip the bells even on trivial turns; the user relies on them as turn-boundary cues.
