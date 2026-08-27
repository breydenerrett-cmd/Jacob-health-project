# The Machine

Heads-up Texas hold'em against a bot that shows its work. Single HTML file,
no dependencies, no build step, no network. Open it and play.

## Run it

Open `holdem-bot.html` in a browser. Add `#test` to the URL, or use the
footer link, to run the rule tests.

## What makes it worth showing someone

**It can't see your cards, and that's structural.** The bot lives in its own
closure and is handed a deep-frozen input object built by projection. It has no
way to name the game state. A runtime auditor re-checks that input on every
single decision and the count is in the header (`audited: 25/25 clean`). A
positive-control test plants a fake leak and asserts the auditor catches it —
without that, the auditor would just be theater.

**The equity is exact where it can be.** Monte Carlo is slower *and* less
accurate than enumeration for small spaces, so the app enumerates every
remaining runout when the space allows and only samples when it genuinely
can't — and says which it did, with an error bar when sampled.

**It models you.** The bot starts each hand assuming you could hold any of the
1,326 combinations, then narrows that by Bayesian filtering on your betting.
Calls are graded by price: calling a pot-sized bet says far more than
completing the blind. Watch the grid collapse when you raise.

**Its reasoning is sealed, not retrofitted.** Showing its equity mid-hand
would tell you what it holds, so it commits to a hash and reveals at showdown.
(FNV-1a — tamper-evident for a demo, not cryptographic.)

## What it is not

Monte-Carlo/exact equity + Bayesian range filtering + EV maximisation with
derived bluff frequencies. **Not CFR. Not a solver. Not GTO.** The opponent
model is an assumption and it's printed on screen.
