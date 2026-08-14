# Daily OFC

A daily [Open-Face Chinese poker](https://en.wikipedia.org/wiki/Chinese_poker) puzzle — one deal a
day, played heads-up against a rival whose cards you get to watch. Wordle-style: everyone gets the
same deal, you get one attempt, and you paste your result to compare.

Single self-contained `index.html`. No build step, no dependencies, no server, no accounts.

## Deploy to GitHub Pages

1. Create a repo and push these files:

```bash
git init && git add . && git commit -m "Daily OFC"
```

2. Create the repo on GitHub and push:

```bash
gh repo create daily-ofc --public --source=. --push
```

3. Turn on Pages:

```bash
gh api -X POST repos/:owner/daily-ofc/pages -f "source[branch]=main" -f "source[path]=/"
```

Or in the web UI: **Settings → Pages → Source: Deploy from a branch → `main` / `root`**.

It goes live at `https://<user>.github.io/daily-ofc/` within a minute or two. Any push to `main`
redeploys.

To use a custom domain, add a `CNAME` file containing the domain and point a DNS `CNAME` record at
`<user>.github.io`.

## How the daily deal works

The deal is derived from the calendar date — `mulberry32` seeded from the day number drives a
Fisher-Yates shuffle. There's no server: every device computes the identical deck offline, so
everyone playing on the same day gets the same 17 cards *and* the same rival. That's what makes
pasted scores comparable — your friends played your hand.

Puzzle #1 is 1 Jan 2026; the day rolls over at local midnight.

## Playing more than one a day

`?d=42` opens puzzle #42. Anything that isn't today is a **practice** puzzle: it keeps its own saved
board, and it never touches your record, streak or stats — so finishing today's deal doesn't leave
you locked out until midnight. A **Play another** button on the result screen jumps to a random one,
and pasted practice results are labelled as such.

Because the deal is a pure function of the puzzle number, `?d=N` is also a stable way to hand
someone a specific hand to try.

## The rival

The rival exists to **turn cards face-up**. It acts first every round, so by the time you commit,
you've seen its reply to the deal — and every card it shows is a card that can no longer come to
you. Chasing a flush is a different decision once four of your suit are dead.

It plays by fixed rules, like a blackjack dealer, so the same deal always produces the same line and
you can learn to read it. It sees only its own cards and the cards you've already placed — never its
own future cards, never your hand. In priority order it keeps its board legal (judging each row by
what it can still *become*, not just what it holds), then takes bankable royalties, then sends high
cards to the bottom, then abandons flush draws as the suit dies in your board.

Measured over 730 deals: **6.2% foul rate**, 0.28 average royalties on clean boards. It is a
competent opponent, not a perfect one — the same squeeze you feel.

Bot strength doesn't affect fairness between friends, since everyone faces the identical rival on
identical cards. It only affects how interesting the decisions are.

## Scoring

Standard heads-up OFC. **+1** per row won, **−1** per row lost; all three rows is a **scoop** for a
further **+3**. Royalties use the real table (top pair 66→AA = 1→9, trips 222→AAA = 10→22; middle
trips/straight/flush = 2/4/8, full/quads = 12/20, straight-flush/royal = 30/50; bottom
straight/flush = 2/4, full/quads = 6/10, straight-flush/royal = 15/25) and are **netted** against the
rival's, so a big hand pays only to the extent it beats theirs. A **foul** surrenders all three rows
and every royalty of yours, and still owes the rival theirs.

## Local state

Three `localStorage` keys, all per-browser — nothing is uploaded:

| Key | Holds |
| --- | --- |
| `ofc-daily-result-v1` | today's finished board |
| `ofc-daily-wip-v1` | the hand in progress |
| `ofc-daily-stats-v1` | played / record / streak / scoops / fouls |
| `ofc-practice-result-<N>` | a finished practice puzzle |
| `ofc-practice-wip-<N>` | a practice hand in progress |

The in-progress hand is saved after every tap and restored exactly as left, including which cards
are still undoable. Without that you could see the deal, close the tab, and reopen to a clean slate
— one puzzle a day would quietly become unlimited retries.

## Running locally

Any static file server, e.g.:

```bash
npx --yes http-server . -p 8044 -c-1
```
