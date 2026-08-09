---
name: Load and cache iSports reference data
description: Load the slow-moving league, team, player, country and bookmaker reference
  data once and keep it fresh from the modify feeds.
api: openapi/isports-api-openapi.yml
operations:
- getFootballLeague
- getFootballLeagueBasic
- getFootballTeam
- getFootballPlayer
- getFootballCountry
- getFootballBookmaker
- getFootballTeamSearch
- getFootballPlayerSearch
generated: '2026-08-09'
method: generated
source: openapi/isports-api-openapi.yml + https://www.isportsapi.com/en/docs.html
---

# Load and cache iSports reference data

## When to use

Before any live integration. Match, event and odds payloads carry only ids — they are meaningless until the
reference tables are loaded.

## Steps

1. **Leagues.** `getFootballLeagueBasic` (`GET /sport/football/league/basic`) for the lightweight list, or
   `getFootballLeague` (`GET /sport/football/league`) for the full profile. Published interval is long
   (30 minutes/call class); the recommended pull is once a day.
2. **Countries.** `getFootballCountry` (`GET /sport/football/country`).
3. **Bookmakers.** `getFootballBookmaker` (`GET /sport/football/bookmaker`) — required to read any odds feed.
4. **Teams and players.** `getFootballTeam` (`GET /sport/football/team`) and `getFootballPlayer`
   (`GET /sport/football/player`); pass `cmd=teamlist` to the player endpoint for a team's squad.
5. **Lookup by name.** `getFootballTeamSearch` (`GET /sport/football/team/search?name=xxx`) and
   `getFootballPlayerSearch` (`GET /sport/football/player/search?name=xxx`).
6. **Keep it fresh.** Re-pull only what the modify feeds flag rather than re-downloading everything.

## Rules

- Cache aggressively. These endpoints carry the loosest rate limits and the provider's own recommended
  frequency is 1 day/call — hitting them per request is the main way integrations get throttled.
- Names come back in English. For localised names use the language-pack endpoints
  (`/sport/languageth`, `/sport/languagevn`, `/sport/languageidn`, `/sport/languagekr`,
  `/sport/languagebra`, `/sport/languagetc`), each of which takes `sport=football` or `sport=basketball`.
- Every plan gates a different set of endpoints. A `code` 2 on a valid key usually means the plan does not
  include that product, not that the key is wrong.
