---
name: degen
description: Find the best straight bet value plays of the day. Default scans NBA, WNBA, NHL, NFL, MLB, and cross-sport signals (Golf always excluded). Pre-filters to leagues with games today. Suggests ~5 straight bets with three-lens research (stats, sharps, public). Tracks hit rate by archetype over time. Use when the user wants daily betting picks or best value bets.
argument-hint: "(optional) sport or game filter, e.g. 'MLB' or 'NBA'. Use 'results' to log outcomes. Use 'history' for hit rate dashboard."
---

# Daily Best Bets — Straight Bet Value Finder

Find the best-value straight bets of the day by scanning ~50 proven situational archetypes across all major sports, then applying three distinct research lenses: pure stats, sharp money signals, and public betting sentiment.

## Arguments

`$ARGUMENTS` — Options:
- Blank: full day card — NBA, WNBA, NHL, NFL, MLB, and cross-sport signals (Golf always excluded)
- Sport filter: `MLB`, `NBA`, `NHL`, `NFL`, etc.
- Team filter: `Lakers`, `Yankees`, etc.
- `update`: Delta check on today's picks — reads tracker.json for today's date, checks current lines vs. recorded odds, scans for breaking injury news on those specific teams only, outputs a compact status table. No archetype scan. Fast and cheap.
- `results [date] [bet #] [W/L/P]`: Log outcome of a previous pick (e.g. `results 2026-04-12 3 W`)
- `history`: Show hit rate summary and last 10 picks from tracker

### `update` output format

| # | Pick | Recorded Odds | Current Odds | Line Move | Injury Flag | Status |
|---|------|--------------|--------------|-----------|-------------|--------|
| 1 | [bet] | [odds] | [odds] | [+/-X pts/cents or —] | [name if flagged, else —] | ✅ Still good / ⚠️ Line moved / ❌ Fade |

**Status rules:**
- ✅ **Still good** — line within 3 cents/pts of recorded odds, no injury news on key players
- ⚠️ **Line moved** — line moved 5+ cents (moneyline) or 1.5+ pts (spread/total) since pick was made; reassess
- ❌ **Fade** — key player on our side announced out/doubtful after pick was recorded, OR line moved sharply against us (10+ cents / 3+ pts)

Only search for injuries and lines on the specific teams in today's picks. Do not re-run the archetype scan or full research framework.

---

## STEP 0 — LEAGUE GAME CHECK

Check today's schedule FIRST to determine which leagues are active. Only load files and scan archetypes for active leagues — this avoids reading irrelevant cached files.

**Default exclusions (always skip unless explicitly requested by argument):**
- Golf — excluded from all default scans

**For each non-excluded league, check today's schedule:**
- NBA: check if any games are scheduled tonight
- NHL: check if any games are scheduled tonight
- NFL: check if any games are scheduled (typically Thu/Sun/Mon in-season; skip entirely in off-season)
- WNBA: **WebFetch `https://www.espn.com/wnba/scoreboard` directly** — do NOT rely on a web search summary, which falsely reports "no games" even when games exist. List every game found (home team, away team, time) before proceeding. Only after this explicit fetch mark WNBA active or inactive.

**Output a brief league status line:**
> Today's active leagues: NBA ✓ | NHL ✓ | NFL ✗ (no games) | MLB ✓
> Loading cached files for: NBA, NHL, MLB

---

## STEP 0a — LOAD CACHED NOTES

Read cached files **only for active leagues identified in Step 0**. Do not read files for excluded or game-less leagues.

- Always read: `~/.claude/skills/degen/leagues/vic.md` — persona and voice for all output
- Always read: `~/.claude/skills/degen/leagues/README.md`
- Always read: `~/.claude/skills/degen/leagues/pending_results.md` — check for any known results on open picks before doing any web searches
- For active NBA: `~/.claude/skills/degen/leagues/nba.md` + `leagues/nba_tracker.md`
- For active NHL: `~/.claude/skills/degen/leagues/nhl.md` + `leagues/nhl_tracker.md`
- For active WNBA: `~/.claude/skills/degen/leagues/wnba.md` + `leagues/wnba_tracker.md`
- For active MLB (if requested): `~/.claude/skills/degen/leagues/mlb.md`
- For active leagues only — coaching context: `~/.claude/skills/degen/leagues/coaching.md` (read only the sections for active leagues)
- For active leagues only — injuries: `~/.claude/skills/degen/leagues/injuries.md` (read only the sections for active leagues)

The `*_tracker.md` files contain ATS records, O/U trends, and team streaks — use them to skip web searches for historical trend data.

Use this cached data to skip re-fetching season standings, park factors, playoff motivation status, ATS trends, and injury recovery timelines. Only run fresh web searches for:
- Today's specific starting pitchers / confirmed lineups
- Current rosters for teams being bet on — verify key players cited are still on that team (trades happen; always confirm via the official league site before citing a player)
- Sharp money signals and line movement (always fresh)
- Public betting percentages (always fresh)
- Today's weather (always fresh)
- Breaking injuries or scratches announced today

**Source priority — always check official league sites first. Use ESPN/CBS/others as secondary only.**
| League | Rosters · Injuries · Schedules · Stats |
|--------|----------------------------------------|
| NBA | nba.com |
| NHL | nhl.com |
| MLB | mlb.com |
| NFL | nfl.com |
| WNBA | wnba.com |

After loading, note which facts are already known vs. what still needs a fresh lookup.

---

## STEP 0b — POST-RESEARCH: UPDATE CACHED NOTES

After completing research (after Step 2), write any newly discovered facts back to the relevant league file. This keeps the notes current so tomorrow's run starts with better data.

**What to write back:**
- Updated team records and standings
- Pitcher ERA/FIP/WHIP updates (if materially different from cached)
- New injuries or return-from-IL notes
- Playoff/motivation status changes
- Any team-specific trend data discovered (e.g. bullpen ERA, O/U streaks)
- Weather patterns for specific parks if relevant

**How to update:** Edit the relevant `leagues/*.md` and `leagues/*_tracker.md` files directly. Update the "Last updated" date at the top of any file you modify. Be surgical — update specific rows/sections rather than rewriting the whole file. Only write back facts that differ materially from what was cached.

**Mandatory — log discovered results:**
Any time you discover the outcome of a previously pending pick (from research, web searches, or score lookups), you must:
1. Update `tracker.json` — set `result` (W/L/P) and `units_profit` for that pick
2. Update `leagues/pending_results.md` — mark that pick as resolved
Do this immediately when a result is discovered, not deferred. This prevents re-searching the same outcomes in future sessions.

---

## STEP 1 — ARCHETYPE SCAN

Check today's slate against archetypes for active leagues only (per Step 0a). Flag every game that matches one or more. Games matching **multiple archetypes** are highest priority — stacking edges is what creates real value.

**For NBA/NHL playoff games:** Before scanning totals archetypes, check the Series Pace Log in `nba_tracker.md` / `nhl_tracker.md` for each active series game. Note: (1) which game number in the series it is, (2) the O/U result of each prior game. If ≥2 straight unders have hit, run Series Adjustment Over (#58). If a team faces elimination in a non-G7, run Series Must-Win Pace-Up (#59).

---

### MLB ARCHETYPES

**Pitching & Starter Edges**
1. **Home Dog Superior Starter** — Home team is an underdog despite having a starter with ERA 2+ runs lower than the opposing starter. Public is overvaluing the road team's brand.
2. **Dominant Road Starter vs Struggling Starter** — Road team's starter has ERA under 3.00 and WHIP under 1.10; opposing starter ERA over 5.00. Stats strongly favor the dog.
3. **FIP vs ERA Regression (Fade)** — Starter's ERA is 1.5+ runs ABOVE their FIP, meaning their ERA is inflated by bad luck/defense. Bet on them — regression is coming.
4. **FIP vs ERA Regression (Bet Against)** — Starter's ERA is 1.5+ runs BELOW their FIP, meaning they've been lucky. Fade them before regression hits.
5. **Extra Rest vs Short Rest** — One starter on 5+ days rest, opposing starter on 3 days (short rest). Rest advantage is meaningful in ERA suppression.
6. **Ace Return from IL** — Team's ace is back from injured list, first start. Often rusty — first start ERA historically elevated. Fade the ace in this spot.
7. **Rookie First Road Start** — Rookie starter making their first road start (no crowd they're familiar with). ERA expectations inflate; lean against them.
8. **Bullpen Depletion Spot** — Opponent used 3+ relievers yesterday in a long game. Today their bullpen is short; lean toward the offense-heavy side or over.
9. **Day Game After Night Game (DANG)** — Home team plays night game, then a day game with the same pitching staff. Opponent had full rest. Lean against the home team.

**Situational / Scheduling Edges**
10. **Dead Cat Bounce** — Team off a 5+ run blowout loss, playing the same opponent at home. Teams bounce back at home after embarrassing losses at a documented rate.
11. **Series Finale Desperation** — Home team is down 0-2 in a 3-game series. Desperation + home crowd + manager burning aces in this spot historically.
12. **Return Home After Long Road Trip** — Team back home after 6+ game road trip. Home comfort + crowd energy, lean toward home team in this spot.
13. **West-to-East Travel Fatigue** — Road team traveled 2+ time zones east for an early game (before 1pm local). Circadian disadvantage suppresses performance.
14. **Better Record Mispriced as Underdog** — One team's record is 3+ games better in W-L but priced as underdog. Market is reacting to name value, not current performance.
15. **Lefty-Righty Platoon Mismatch** — Starting pitcher faces a lineup that historically crushes their handedness. Check opposing team's L vs R splits.
16. **Interleague DH Disadvantage** — AL team playing in NL park without DH (or vice versa in current rules), disrupting their normal lineup configuration.

**Weather / Environment**
17. **Wind-In Under** — Wind blowing IN at 10+ mph at an outdoor park with ERA 4.00+ starters. Suppresses power, lean under.
18. **Wind-Out Over** — Wind blowing OUT at 12+ mph with weak pitching matchup (both starters ERA 4.50+). Ball carries, lean over.
19. **Cold Weather Under** — Temperature below 45°F, outdoor park, both offenses average or below. Cold suppresses bat speed and carry.

**Total-Based Edges**
20. **Both Starters Struggling Over** — Both starters have ERA over 5.00 and WHIP over 1.50. High-scoring games statistically expected.
21. **Pitcher's Duel Under** — Both starters under 2.50 ERA, under 1.05 WHIP. Lean under even if total looks low.

---

### NBA ARCHETYPES

**Rest & Fatigue**
22. **Back-to-Back Road Favorite Fade** — Road team playing second night of a back-to-back, priced as favorites. Tired road chalk is consistently overpriced by the market.
23. **Rested Home Dog vs Road B2B** — Home team has 2+ days rest against a road team on a back-to-back. Stack rest + home edge.
24. **3rd Game in 4 Nights** — Team playing their 3rd game in 4 nights. Fatigue compounding; lean against them especially if they're road favorites.
25. **Altitude Disadvantage** — Road team visiting Denver on 1 day rest. Altitude + fatigue = consistent underperformance vs market expectations.
26. **4th Game in 5 Nights (Road)** — Road team's 4th game in 5 nights. Extreme fatigue spot; strongly lean against regardless of record.

**Motivation & Situational**
27. **Last Day Regular Season Motivation Mismatch** — Clinched team vs team fighting for play-in positioning. The desperate team covers at a high rate as clinched teams may rest stars.
28. **Play-In Home Dog** — Play-in game at home for the underdog. Crowd intensity + desperation = strong cover spot historically.
29. **Trap Game Fade** — Big favorite coming off an emotional upset win, now facing a weak opponent, before a marquee game next. Classic letdown/sandwich game; fade the chalk.
30. **Revenge Game Home** — Team lost by 10+ in the last H2H meeting, now hosting the same opponent. Home crowd revenge energy documented across sports.
31. **Under — Two Elite Defenses** — Both teams rank top-8 in defensive efficiency rating. These matchups historically go under the total at a strong rate.
32. **Under Playoff Game 7** — Game 7 historically produces the lowest-scoring game of any series. Both teams tighten up defensively; lean under.
33. **Star Load Management Not Priced** — Road team's star announced as out or doubtful late and the line hasn't fully adjusted. Fade the road team ML.
34. **Under in Seeding Must-Win** — Both teams fighting for positioning on a meaningful game; intensity spikes defensively. Lean under.
35. **Home Dog in Division Rival Game** — Division rivals know each other too well; large spreads rarely hold up. Fade the road favorite.
36. **Interim/Motivated Coach Spot** — Team recently fired their coach, interim is in; teams historically play hard for new coaches in first 5 games.

**Playoff Series Trends (NBA, NHL & WNBA)**
58. **Series Adjustment Over** — Triggers when ≥2 consecutive prior games in a series both went UNDER. Both staffs have film on each other's defensive scheme; offensive counter-adjustments land by G3/G4. Defenses are harder to sustain over a series than offenses are to fix. Confirm with: actual game totals trending up vs. the posted line, pace stats (possessions/game rising). Unit bump if total line is flat or lower than prior games despite rising actual scores. Check `series_prior_ou` in nba_tracker.md / nhl_tracker.md / wnba_tracker.md Series Pace Log.
59. **Series Must-Win Pace-Up** — Non-G7 elimination game: G4 (down 0-3), G5 (down 1-3), G6 (down 2-3). Trailing team deliberately up-tempos to force extra possessions; intentional fouls late extend games. Favors the OVER. Distinct from #32 "Under Playoff Game 7" — that archetype covers G7 defensive lockdown, opposite dynamic. Reduce confidence if trailing team's identity is elite defense + slow pace.

---

### WNBA ARCHETYPES

**Travel & Fatigue**
60. **Commercial Travel B2B** — WNBA teams fly commercial, not charter. Back-to-back with cross-country travel (2+ time zones) is more fatiguing than any NBA equivalent. Lean against this team, especially on the road.
61. **3rd Game in 5 Nights (WNBA)** — Short 40-game season compresses scheduling; a third game in five nights on thin rosters hits harder than NBA equivalents. Fade this team on the road.

**Roster & Injury**
62. **Thin Roster Star Scratch** — Late-announced star unavailable (A'ja Wilson, Caitlin Clark, Breanna Stewart, Sabrina Ionescu). With 11-12 active players and no true depth replacement, the market rarely adjusts fully in time. Fade the depleted team's ML.
63. **Olympic/FIBA Window Depletion** — National team windows pull 3+ players simultaneously. Teams losing multiple starters to FIBA/Olympics are significantly undermanned and often mispriced vs. unaffected opponents.

**Motivation & Situational**
64. **Commissioner's Cup Motivation Mismatch** — Cup games run June 1–17; championship June 30. East: 7 teams × 6 games; West: 8 teams × 7 games. Standings order: (1) best Cup record, (2) point differential (first tiebreaker), (3) head-to-head. Every Cup game except the final also counts toward regular-season standings — dual stakes. Prize pool: $500K total (~$30K/player winner, $10K/player loser). **Betting angles:** (a) Lean toward the Cup-contending team vs. an eliminated opponent on ML/spread. (b) Lean OVER when a contending team is chasing point differential in a tight conference race — they have explicit incentive to run up the score, not just win. The further they are from the leader in differential, the stronger the over lean.
65. **Rivalry Home Dog** — Home underdog in heated rivalries (NY Liberty/Connecticut Sun, Las Vegas Aces/LA Sparks, Indiana/Chicago). Division familiarity tightens games; large road spreads historically fade well.
66. **Return Home After Road Swing** — Team back home after 4+ game road trip in a 40-game season. Effect is amplified vs. NBA; lean home.

**Market**
67. **National Spotlight Public Fade** — Teams with outsized national TV attention (Indiana Fever/Clark, Las Vegas Aces dynasty brand) routinely receive 70%+ public action regardless of matchup quality. Apply Public Darling Fade logic (#55) but expect the distortion to be more extreme than other leagues.

**Playoffs**
68. **WNBA First Round G2 Lower Seed Home Elimination** — WNBA first round is best-of-3; lower seed hosts G2 (higher seed always hosts G1 and G3). If higher seed wins G1, lower seed faces elimination at home in G2 — maximum crowd intensity + desperation. Unique structure not found in NBA. Lean home dog ML in G2 elimination spots; lean OVER as pace and urgency spike. Also applies archetypes #58 (Series Adjustment Over) and #59 (Series Must-Win Pace-Up) — check wnba_tracker.md Series Pace Log for prior game O/U results.
69. **WNBA Playoff Seeding Must-Win** — No play-in in WNBA; bubble teams (seeds 7-8) are fighting for inclusion, not just seeding. A team on the playoff bubble in final 2 weeks of regular season with a home game vs. an eliminated opponent = maximum desperation + home court. Lean home team regardless of spread size.

---

### NHL ARCHETYPES

**Goalie & Scoring**
37. **Strong Recent Over Trend** — Team has hit the over in 70%+ of their last 10 games. Trend-based edge, especially when opponent also has elevated over rate.
38. **Elite vs Backup Goalie Mismatch** — One team starts an elite goalie (.920+ save%), other team starts a backup (.890- save%). Massive edge in expected goals.
39. **Goalie Bounce Back** — Starting goalie allowed 4+ goals in prior start. Historically goalies rebound the following game; lean on their team and the under.
40. **Under — Both Top-5 Goals Against** — Both teams rank top-5 in fewest goals allowed per game. These games go under at a very high rate.
41. **Over — Both Bottom-5 Save%** — Both teams have poor save percentages and high shot volumes. High-scoring game expected.
42. **Power Play Mismatch** — One team's PP% is 20%+ vs other team's PK at 75% or lower. Significant power play edge that often tips close games.

**Fatigue & Schedule**
43. **3rd Game in 4 Nights (NHL)** — Team on their 3rd game in 4 days. NHL fatigue compounds faster than other sports; fade this team especially on the road.
44. **5th Game in 7 Nights (Road)** — Road team's 5th game in 7 nights. Extreme fatigue; lean against road team regardless of record.
45. **Division Dog in Tight Game** — Division rivals historically play tighter games; large spreads and heavy favorites fade well in division matchups.

---

### NFL / CFB ARCHETYPES

46. **Off-Bye-Week Home Dog** — Home underdog coming off their bye week. Documented ~58% ATS historically — the single strongest situational edge in NFL betting.
47. **Home Dog in Divisional Game** — Division games are historically close regardless of record gap. Road favorites rarely cover big spreads vs division opponents.
48. **Letdown Spot Road Favorite** — Road team coming off massive emotional win (upset of a top-5 team), now favored against a weak opponent. Classic sandwich/letdown fade.
49. **Under Cold Outdoor Game** — Temperature below 25°F, outdoor stadium, pass-heavy offenses. Cold weather consistently suppresses scoring and passing games.
50. **Under — Both Top-10 Defensive DVOA** — Both teams rank top-10 in defensive DVOA or yards allowed per play. Lean hard under in these matchups.
51. **Road Dog Revenge (Playoff or Rematch)** — Team that lost by 14+ in regular season meeting now playing a rematch (playoff or same-year rematch). Motivated underdog spot.
52. **Primetime Road Favorite Fade** — Public massively overloads on primetime road favorites (Monday Night, Sunday Night games). Market inflates favorites; fade in spots with 70%+ public.

---

### CROSS-SPORT / MARKET ARCHETYPES

53. **Reverse Line Movement (RLM)** — Line moves in the OPPOSITE direction from where the majority of public bets are going. Classic sharp money indicator. The bigger the gap (e.g. 70% public on Team A, line moves toward Team B), the stronger the signal.
54. **Steam Move** — Line moves 5+ cents across multiple books within a 30-minute window. Coordinated sharp/syndicate action. High confidence if confirmed across 3+ books.
55. **Public Darling Fade** — Team receiving 70%+ of public bets AND 70%+ of public money, priced at -150 or heavier. Market is distorted by crowd; sharp value is on the other side.
56. **Blowout Loser Public Over-Fade** — Team just got blown out; public hammers them next game as a fade. Market overcorrects. Buy the team that got embarrassed — especially at home.
57. **Closing Line Value Validator** — After picking a side, verify that the closing line moves your way before game time. If you took +120 and it closed at +105, you beat the market = value confirmed. Track this.

---

## STEP 2 — RESEARCH FRAMEWORK

For each game flagged by the archetype scan, gather data across **three strictly separated lenses**:

### Lens 1 — PURE STATS (objective, bias-free)
- Team record, last 10 game form, home/away splits — **never conflate a team's overall record with their home or away record. Only state a home/away split if you have actually looked it up; do not infer it from overall record.**
- Sport-specific stats: ERA/WHIP/FIP (MLB), offensive/defensive efficiency (NBA), save%/GF/GA (NHL), DVOA/yards per play (NFL)
- Rest days, travel direction, head-to-head record this season
- Weather conditions for outdoor games
- Injury report

### Lens 2 — SHARP SIGNALS (informed money, potentially biased)
- Opening line vs current line — direction and magnitude of movement
- Reverse line movement: line moving against the public = sharp signal
- Steam moves / syndicate reports from Action Network, BetQL, Covers sharp
- Note: sharps are right more than they're wrong, but they have their own biases and may already be fully priced in

### Lens 3 — PUBLIC SENTIMENT (crowd behavior)
- % of bets and % of money on each side (Action Network, Covers, DK Network)
- Is this a public darling game (prime time, big market, national TV)?
- Media narrative consensus
- Note: public isn't always wrong — but 70%+ public lean on a favorite is a meaningful fade indicator

---

## STEP 3 — VALUE FILTER

Only recommend bets that:
- Price at **-130 or better**
- Match **at least 1 named archetype** from the list above
- Have at least **2 of 3 lenses** pointing the same direction
- OR have an exceptionally strong single lens + named archetype match

---

## STEP 4 — BET SELECTION

Select ~5 bets for the day:
- **3-4 core plays**: -130 to +150, stats + at least one other lens aligned
- **1-2 value plays**: +150 to +250, only when the long shot is backed by archetype match + stats
- Mix bet types (ML, spread, total) and sports when possible
- Games matching **multiple archetypes** get priority over single-archetype matches

---

## OUTPUT FORMAT

### For each bet:

---

### Bet [#]: [Team/Side/Total] ([Bet Type]) — ~[odds] — **[X]u**
**Sport/Game:** [League — Team A vs Team B, Time ET]
**Tier:** Core / Value
**Archetype(s) Triggered:** [e.g. "Home Dog Superior Starter + Reverse Line Movement"]
**Unit Size:** [X]u — [reason: e.g. "All 3 lenses + 2 archetypes → 1.5u" or "Conflicted stats lens → 0.5u"]

#### Research Breakdown

| Lens | Signal | Direction |
|------|--------|-----------|
| **Stats** | [key stats — ERA, record, efficiency, rest, weather, etc.] | LEAN [side] |
| **Sharps** | [line movement — opened X, now Y, direction vs public] | LEAN [side] / NEUTRAL / CONFLICTED |
| **Public** | [bet % and money % on each side] | FADING [side] / WITH [side] |

**Our lean:** [One sentence: which side and the core reason]
**Alignment:** Stats + Sharps / Stats + Public Fade / All three / Stats only

---

*(repeat for all ~5 bets)*

---

### Daily Card Summary

| # | Game | Bet | Type | Odds | Units | Tier | Archetype | Stats | Sharps | Public | Pick |
|---|------|-----|------|------|-------|------|-----------|-------|--------|--------|------|
| 1 | [Team A vs Team B] | [bet] | ML/Spread/Total | [odds] | [Xu] | Core | [archetype] | [lean] | [lean/—] | [fade/with] | [side] |
| 2 | ... | | | | | | | | | | |
| 3 | ... | | | | | | | | | | |
| 4 | ... | | | | | | | | | | |
| 5 | ... | | | | | | | | | | |
| | | | | **Day total** | **[X]u** | | | | | | |

---

### Tracker Update

After displaying picks, read `~/.claude/skills/degen/leagues/tracker.json`. If it doesn't exist, create it:

```json
{
  "picks": [],
  "summary": {
    "our_record": {"W": 0, "L": 0, "P": 0},
    "sharp_agreement_record": {"W": 0, "L": 0},
    "sharp_disagreement_record": {"W": 0, "L": 0},
    "public_fade_record": {"W": 0, "L": 0},
    "public_with_record": {"W": 0, "L": 0},
    "by_archetype": {},
    "by_series_game_num": {
      "G1": {"W": 0, "L": 0},
      "G2": {"W": 0, "L": 0},
      "G3": {"W": 0, "L": 0},
      "G4": {"W": 0, "L": 0},
      "G5": {"W": 0, "L": 0},
      "G6": {"W": 0, "L": 0},
      "G7": {"W": 0, "L": 0}
    }
  }
}
```

Each pick entry:
```json
{
  "date": "YYYY-MM-DD",
  "bet_num": 1,
  "game": "Team A vs Team B",
  "game_detail": "BOS (4-9) @ STL (8-5) · 2:15 PM ET · Busch Stadium",
  "bet": "Cardinals ML",
  "odds": "+120",
  "tier": "Core",
  "archetypes": ["Home Dog Superior Starter", "Reverse Line Movement"],
  "stats_lean": "Cardinals",
  "stats_detail": "Punchy stat summary for card display.",
  "sharp_lean": "NEUTRAL",
  "sharp_detail": "Punchy sharp signal summary for card display.",
  "public_lean": "FADE Red Sox",
  "public_detail": "Punchy public sentiment summary for card display.",
  "our_lean": "One punchy sentence for card display.",
  "our_pick": "Cardinals",
  "units": 1.0,
  "result": null,
  "units_profit": null,
  "sharp_agreed": null,
  "public_was_faded": true,
  "series_game_num": null,
  "series_prior_ou": null
}
```

**Unit sizing rules (apply every pick):**
- **0.5u** — Conflicted lens, single archetype with uncertainty, Value tier with caveats
- **1.0u** — Standard: 1-2 archetypes, stats + 1 other lens aligned
- **1.5u** — Strong: 2+ archetypes, 2-3 lenses all aligned, Core tier
- **2.0u** — Exceptional: 3+ archetypes, all lenses, clear market inefficiency
- **3-5u** — Extreme: almost never. Reserve for rare, multi-edge no-brainers.

When result is logged: set `units_profit` to actual P/L (e.g. 1u win at -110 = +0.91, 1u win at +145 = +1.45, any loss = -units wagered).

```json
```

### Card Format Constraints

The card renderer uses fixed pixel budgets per section. Write content to fit — not paragraph prose.

| Field | Hard limit | Pixel budget |
|---|---|---|
| `game_detail` | 52 chars (1 line) | hero row |
| `archetypes` | max 3 items, 24 chars each | hero row |
| `stats_detail` | 200 chars (~5 lines) | 130px lens |
| `sharp_detail` | 200 chars (~5 lines) | 130px lens |
| `public_detail` | 200 chars (~5 lines) | 130px lens |
| `our_lean` | 150 chars (~3–4 lines) | 110px lean box |

**Write punchy card copy, not paragraphs:**
- ❌ "No reverse line movement has been confirmed at this time. The books are currently pricing Boston at -122 on the road with a 9.00 ERA starter, which is a market anomaly driven by brand recognition rather than matchup quality."
- ✅ "No RLM. BOS priced -122 on road with a 9.00 ERA starter — market anomaly, no sharp agreement."

**`stats_lean` / `sharp_lean` / `public_lean`** are short signal labels (1–3 words) used in the signal pill bar: team name, "NEUTRAL", "LEAN Cardinals", "FADE Red Sox", "LEAN UNDER". Keep these ≤ 14 chars.

The `by_archetype` summary object tracks W/L record for each archetype name so we can see which archetypes are actually hitting over time.

**When `results` argument is used:** Update the matching pick's `result` and `units_profit` fields, recalculate all summary stats including by_archetype and unit P/L, save the file, then display:

---

### Hit Rate Dashboard

| Category | W | L | Push | Win% | Units Net |
|----------|---|---|------|------|-----------|
| **Our overall record** | [W] | [L] | [P] | [%] | **[+/-Xu]** |
| Bets where sharps agreed | [W] | [L] | — | [%] | [+/-Xu] |
| Bets where we differed from sharps | [W] | [L] | — | [%] | [+/-Xu] |
| Bets fading the public | [W] | [L] | — | [%] | [+/-Xu] |
| Bets with the public | [W] | [L] | — | [%] | [+/-Xu] |
| **Pending (open bets)** | — | — | — | — | [Xu at risk] |

**Unit Sizing Breakdown** (resolved bets only):

| Size | Bets | W | L | Units Net |
|------|------|---|---|-----------|
| 0.5u | [n] | [W] | [L] | [+/-Xu] |
| 1.0u | [n] | [W] | [L] | [+/-Xu] |
| 1.5u | [n] | [W] | [L] | [+/-Xu] |
| 2.0u+ | [n] | [W] | [L] | [+/-Xu] |

**Archetype Performance** (min 3 bets to show):

| Archetype | W | L | Win% | Units Net |
|-----------|---|---|------|-----------|
| [archetype name] | [W] | [L] | [%] | [+/-Xu] |
| ... | | | | |

**Series Game Position** (playoff bets only, O/U bets where series_game_num is set):

| Game # | O/U Bets | W | L | Win% |
|--------|----------|---|---|------|
| G1 | [n] | [W] | [L] | [%] |
| G2 | [n] | [W] | [L] | [%] |
| G3 | [n] | [W] | [L] | [%] |
| G4 | [n] | [W] | [L] | [%] |
| G5+ | [n] | [W] | [L] | [%] |

**Key insight:** [One sentence on the most useful pattern emerging — e.g. "Home Dog Superior Starter is 4-1 (80%), +3.2u net — lean harder on pitching archetypes. 1.5u bets outperforming 1u bets."]

---

**When `history` argument is used:** Read tracker.json, display the full Hit Rate Dashboard + Archetype Performance table + last 10 picks with results.

---

> **Disclaimer:** Verify exact odds on DraftKings, FanDuel, or BetMGM before placing. Lines move — check current odds before placing. No bet is guaranteed.

---

## TEXT EXPORT

After displaying picks, write only the Daily Card Summary table to:
`/Users/kurtkurt/Documents/Dev - claude/degen-YYYY-MM-DD.txt`

Use today's date in the filename. No bet cards, no research breakdowns — summary table only.

---

## Examples

- `/degen` — full day card: NBA, WNBA, NHL, NFL, MLB (active leagues only; Golf excluded from default)
- `/degen MLB` — MLB only
- `/degen NBA` — NBA only
- `/degen results 2026-04-12 3 W` — log Bet #3 from April 12 as a win
- `/degen history` — show cumulative hit rate dashboard

## Tracker Merge Command (canonical)

tracker.json is a flat JSON array. Merge mobile-session entries with:

```bash
jq -s 'if (.[0]|type)=="array" then .[0] + .[1] | group_by([.date, .bet_num]) | map(last) else error("tracker.json is not an array - STOP, do not write") end' tracker.json new_entries.json > tmp.json && mv tmp.json tracker.json
```

Never use `jq '. + input'` — on object input it key-merges and silently fragments pick history (caused the June 2026 data loss).
