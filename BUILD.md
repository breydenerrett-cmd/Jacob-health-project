# Coast Clash — Alpha Build Spec

Execution-ready. No decisions left open. Build exactly this.

## Goal

A playable East-vs-West rap debate game. Two friends, one device, ~15 minutes.
Alpha quality: correct rules, readable UI, zero polish.

## Hard constraints

1. **Output is ONE file: `coast-clash.html`.** Inline `<style>` and `<script>`. No modules, no imports, no CDN, no build step, no npm, no server.
2. Must run by double-clicking the file from `file://`. If it needs a server, it's wrong.
3. No external network calls of any kind.
4. Plain JS. No framework. No TypeScript.
5. No images. Artists render as a colored tile with their initials.
6. Mobile-first single column, max-width ~700px. Dark background. Big tap targets.
7. Total file target: under ~1200 lines. If a feature would blow that, cut the feature.

## Locked defaults — do not prompt the user for these

- Era: **2000–present**, all listed artists eligible. No era toggle.
- Scoring: **Full Card** — always play all 5 rounds.
- Categories, fixed order: `superstar → hitmaker → numbers → rapAbility → legacy`
- Roster size: 5 per player. Draft: alternating picks, P1 first.
- Default names: `Brey` and `Jake`, both editable.
- Judge: **deterministic only**. No AI, no API key, no fetch.

## Data

One `const ARTISTS = [...]` array at the top of the script. 20 East + 20 West.

**East:** Jay-Z, Nas, 50 Cent, DMX, Jadakiss, Cam'ron, Juelz Santana, Fabolous,
Busta Rhymes, Fat Joe, Ja Rule, Nicki Minaj, A$AP Rocky, Joey Bada$$, Pop Smoke,
Cardi B, Meek Mill, Mac Miller, Lil Uzi Vert, Pusha T

**West:** Snoop Dogg, Dr. Dre, Ice Cube, Xzibit, The Game, Kendrick Lamar,
ScHoolboy Q, Nipsey Hussle, YG, E-40, Too $hort, Mac Dre, DJ Quik, Nate Dogg,
Tyler the Creator, Vince Staples, Roddy Ricch, G-Eazy, Larry June, Baby Keem

Each entry:

```js
{ id, name, coast: 'east'|'west', city,
  r: { superstar, hitmaker, numbers, rapAbility, legacy },   // 40–99
  songs: [ { title, year, iconic: true|false, peak: <Hot100 peak or null> } ] }
```

Rules for the data:
- 1–3 signature songs per artist. Song titles only — never lyrics.
- `peak`: fill in ONLY where confident. Otherwise `null`. **Never invent a chart number.**
- Ratings are editorial. Spread them — nobody should be 90+ across all five.
  Specialists are the point: Larry June low `numbers`, high `rapAbility`;
  Ja Rule high `hitmaker`, low `rapAbility`; DJ Quik low `superstar`, high `legacy`.
- Add `const RATINGS_VERSION = 'provisional-v1'` and show the word **provisional**
  once in the UI footer. These are opinions, and the app should say so.

## Engine — pure functions, no DOM access

```
newGame(p1Name, p2Name, p1Coast)
canDraft(g, playerIdx, artistId)      // right coast, unowned, roster < 5
draft(g, playerIdx, artistId)
canPlay(g, playerIdx, artistId)       // on roster, not burned (ignored in Final Boss)
lockSelection(g, playerIdx, {artistId, hitSongId|null, argument})
bothLocked(g)
resolveRound(g, judgment)             // award point, burn both artists, once only
advanceRound(g)
isComplete(g)
needsFinalBoss(g)                     // after 5 rounds, scores equal
recapMarkdown(g)
```

`state` is one plain serializable object. Every mutation returns/updates it, then
`save()` writes `JSON.stringify(state)` to `localStorage['coastclash']`.
On load, if a saved in-progress game exists, show a **Resume / New Match** choice.

## Judge — `judgeRound(g, round)`

All arithmetic, no randomness. Show every term to the players.

```
total = base + argBonus + hitBonus
base  = artist.r[category]
draw  if |totalA - totalB| <= 1.5
```

**argBonus, 0–6.** Scores argument FORM, not quality. Print this rubric on the
argument screen so players know how to earn it:

| pts | test |
|-----|------|
| +2 | argument text contains the opponent's artist name |
| +2 | contains a 4-digit year, a digit, a `"quoted title"`, or any song title in ARTISTS |
| +1 | contains >= 2 keywords from that category's keyword list |
| +1 | length between 40 and 400 chars |

Empty/skipped argument = 0. Cap at 6.

**hitBonus.** Only if the player spends their one Hit Card that round.
`songScore` = 50, +25 if `iconic`, +25 if `peak <= 10` (if peak is null, ignore).

```
weight = { superstar:.08, hitmaker:.08, numbers:.05, legacy:.04, rapAbility:.02 }
hitBonus = Math.round(songScore * weight[category])
```

Returns `{ winner: 0|1|null, draw, totals:{a,b}, breakdown, explanation }`.
`explanation` is a **1–2 sentence template**, host-toned, naming the decisive term.
Example: `"West takes Superstar — Snoop's 92 card holds off 50 Cent's 88, and the Hit Card added +7 in a round that rewards mainstream reach."`

**Final Boss** (only if `needsFinalBoss`): burn rules off, any roster artist,
category is a blended `(legacy + superstar) / 2`. Winner takes the match. No draws —
if tied, higher `legacy` wins.

## Screens — one `render()` off `state.screen`

`setup → coast → draft → roster → pick(P1) → pass → pick(P2) → reveal → result → [x5] → final → recap`

- **pick**: shows only the active player's name, their unburned roster, an argument
  textarea (maxlength 400), a Hit Card toggle + song dropdown for the selected artist,
  and **Lock**. Opponent's data never rendered to the DOM on this screen.
- **pass**: full-bleed neutral screen, `PASS TO {name}` + a button. Nothing else.
- **reveal**: both cards side by side, category, both arguments, **Judge Round** button.
- **result**: winner, explanation, full score math, `X is burned`, score, **Next Round**.
- **recap**: rendered summary + a `<textarea>` holding the Markdown + **Copy** button.
  Recap format follows handoff §26. Also compute MVP (biggest win margin) and
  Best Hit Card (largest hitBonus on a won round).
- **Rematch** button resets to `setup` with names kept.

## Self-test — no test framework

If `location.hash === '#test'`, skip the game and run ~12 assertions in-page,
printing `PASS`/`FAIL` lines into the body:

- can't draft opposite coast, can't draft 6, can't draft a taken artist
- can't play a burned artist
- judge can't run before both lock
- point awarded once; `resolveRound` twice doesn't double-score
- draw awards nobody a point
- hitBonus in rapAbility < hitBonus in hitmaker for the same song
- argBonus caps at 6 and is 0 for empty text
- 5 rounds completes the match
- `recapMarkdown` contains both rosters and the final score

## Done when

Open the file, play `Brey` vs `Jake` start to finish without touching devtools,
and `#test` prints all PASS. Then write a 5-line `README.md`: what it is, how to
run it, how to edit the artist list.

## Explicitly NOT in alpha

AI judge, images, animations, era modes, card tiers as mechanics, power cards,
online play, accounts, source citations, sales figures beyond chart peaks.
