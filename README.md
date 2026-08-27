# Coast Clash

A two-player East-vs-West rap debate game. Draft five artists, deploy one per
category, make your case, let the judge score it. About 15 minutes a match.

## Run it

Open `coast-clash.html` in a browser. That's it — no install, no server, no
build step, no network. Works from a `file://` path or any static host.

Add `#test` to the URL (`coast-clash.html#test`) to run the built-in rule tests.

## How it plays

Five rounds: **Superstar → Hitmaker → Numbers → Rap Ability → Legacy**.
Each artist can be played in **one round only**, so the draft is about timing,
not just talent. Each player gets **one Hit Card** per match — a signature song
that is worth more in Hitmaker or Superstar than in Rap Ability.

The judge is deterministic and shows all of its arithmetic:
`card rating + argument bonus + hit card bonus`. The argument bonus (max +6)
scores the *form* of your case — did you name the opposing artist, did you cite
something specific — not whether the claim is true. The rubric is printed on the
argument screen. Rounds within 1.5 points are a draw; a tied match goes to a
Final Boss round where burned artists come back.

## Editing the artists

All 40 artists live in the `ARTISTS` array near the top of the `<script>`:

```js
A("nas","Nas","east","Queensbridge, NY",[78,72,76,97,95],
  [["Made You Look",2002,true],["One Mic",2001,true]]),
```

Ratings are `[superstar, hitmaker, numbers, rapAbility, legacy]`, 40–99.
Songs are `[title, year, iconic, hot100Peak]` — leave the peak off unless you
actually know it.

## On the ratings

The ratings are **provisional and editorial** — informed opinion, not
measurement, and the app says so on every screen. Chart peaks appear only where
they're known; no sales or streaming figures are estimated anywhere.

## Not in this alpha

AI judging, artist images, era modes, expanded rosters. See `BUILD.md`.
