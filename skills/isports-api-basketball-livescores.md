---
name: Poll basketball livescores and standings
description: Pull basketball fixtures, live scores and league tables, using the delta
  endpoint for live updates.
api: openapi/isports-api-openapi.yml
operations:
- getBasketballLivescores
- getBasketballLivescoresChanges
- getBasketballSchedule
- getBasketballStandingLeague
- getBasketballStats
generated: '2026-08-09'
method: generated
source: openapi/isports-api-openapi.yml + https://www.isportsapi.com/en/docs.html
---

# Poll basketball livescores and standings

## Steps

1. `getBasketballLivescores` (`GET /sport/basketball/livescores`) for today's snapshot, keyed by `matchId`.
2. `getBasketballLivescoresChanges` (`GET /sport/basketball/livescores/changes`) for every poll after that.
3. `getBasketballSchedule` (`GET /sport/basketball/schedule`) for fixtures and results by `date`
   (`yyyy-MM-dd`, GMT+0), `leagueId` or `matchId`.
4. `getBasketballStandingLeague` (`GET /sport/basketball/standing/league`) needs `leagueId`; pass `season`
   to pin a past table.
5. `getBasketballStats` (`GET /sport/basketball/stats`) for team box-score style match statistics; pass
   `cmd=stats` to get the quarter-by-quarter breakdown.

## Rules

- Basketball ids are a different namespace from football ids. A `leagueId` from a football endpoint will
  not resolve here.
- Same envelope, same failure mode: HTTP 200 always, outcome in `code`.
- Respect the published per-endpoint interval; there is no retry-after header to read.
