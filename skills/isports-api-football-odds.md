---
name: Read football odds and track line movement
description: Fetch pre-match, in-play and historical football odds by bookmaker and
  follow line moves through the changes feed.
api: openapi/isports-api-openapi.yml
operations:
- getFootballOddsMain
- getFootballOddsMainChanges
- getFootballOddsMainHistory
- getFootballOddsInplay
- getFootballOddsAll
- getFootballOddsAllChanges
- getFootballBookmaker
generated: '2026-08-09'
method: generated
source: openapi/isports-api-openapi.yml + https://www.isportsapi.com/en/docs.html
---

# Read football odds and track line movement

## When to use

Use this skill to price or display football markets: Asian handicap, over/under and 1x2, pre-match and in-play.

## Steps

1. **Resolve bookmakers first.** `getFootballBookmaker` (`GET /sport/football/bookmaker`) maps `companyId`
   to a bookmaker name. Every odds record is keyed by `matchId` + `companyId`; without this map the feed is
   unreadable.
2. **Choose the breadth you are paying for.** `getFootballOddsMain`
   (`GET /sport/football/odds/main`) covers the main 18 agencies; `getFootballOddsAll`
   (`GET /sport/football/odds/all`) covers the full set. Both take `matchId` and `companyId` filters and
   accept comma-separated batches.
3. **Track movement with the delta feeds, not repeated full pulls.** `getFootballOddsMainChanges`
   (`GET /sport/football/odds/main/changes`) and `getFootballOddsAllChanges`
   (`GET /sport/football/odds/all/changes`) return only what moved.
4. **In-play is a separate endpoint.** `getFootballOddsInplay` (`GET /sport/football/odds/inplay`) carries
   the running match state alongside the price.
5. **Backfill from history.** `getFootballOddsMainHistory`
   (`GET /sport/football/odds/main/history`) takes a `date` in `yyyy-MM-dd` (GMT+0).

## Rules

- Batch instead of looping: `matchId=322964610,322964611` and `companyId=8,19` are supported.
- Odds endpoints carry the tightest published intervals in the catalogue (as low as 1 second/call). Read
  `x-rate-limit` on the operation before choosing a poll loop.
- A handicap value is signed: positive means the home team gives goals, negative means it receives them.
- Never treat HTTP 200 as success. Check `code == 0` first.
