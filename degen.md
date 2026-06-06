---
name: degen
description: Find the best straight bet value plays of the day. Scans NBA, WNBA, NHL, NFL, MLB. Suggests ~5 straight bets with three-lens research. Use for daily betting picks.
argument-hint: "(optional) sport filter, e.g. MLB or NBA. Use results to log outcomes. Use history for hit rate dashboard."
---

## Pre-flight: Regenerate Project Prompt

Before running the main degen workflow, silently regenerate `~/.claude/skills/degen/leagues/project-prompt.md`:

1. Read `~/.claude/skills/degen/leagues/SKILL.md`, `~/.claude/skills/degen/leagues/vic.md`, and `~/.claude/skills/degen/leagues/degen.md`
2. Produce a single markdown file that contains:
   - Vic's full persona and voice from vic.md
   - All archetypes and the full research framework from SKILL.md
   - All local file path references swapped for GitHub raw URLs: `~/.claude/skills/degen/leagues/` → `https://raw.githubusercontent.com/thejoyofmissingout/degen-trackers/main/`
   - The mobile read-only note: tracker files are read via GitHub URLs; no write-back; end every session with a formatted Tracker Update Summary block
   - Trigger phrases: "hey Vic", "degen", "/degen", "what are we betting today"
3. Write the result to `~/.claude/skills/degen/leagues/project-prompt.md`
4. Compare the new `project-prompt.md` to what was there before. If any of the following changed, output a visible reminder at the END of the session (after all picks): any archetype added, removed, or materially changed; Vic's persona or voice rules changed; research framework or value filter changed; unit sizing rules changed. Format the reminder as:
   ⚠️  SYNC VIC — archetypes/persona updated. Paste project-prompt.md into the Claude Project:
   https://raw.githubusercontent.com/thejoyofmissingout/degen-trackers/main/project-prompt.md
5. Do not mention this step to the user unless the reminder fires.

Then proceed with the normal degen workflow below.

---

Read and execute the full instructions from `~/.claude/skills/degen/leagues/SKILL.md` with arguments: $ARGUMENTS
