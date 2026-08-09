---
name: Poll football livescores efficiently
description: Pull today's football fixtures and keep them current using the delta
  endpoint instead of re-pulling the full snapshot.
api: openapi/isports-api-openapi.yml
operations:
- getFootballLivescores
- getFootballLivescoresChanges
- getFootballSchedule
- getFootballScheduleModify
generated: '2026-08-09'
method: generated
source: openapi/isports-api-openapi.yml + https://www.isportsapi.com/en/docs.html
---

# Poll football livescores efficiently

## When to use

Use this skill to show live football scores and keep them current without burning the per-endpoint rate limit.

## Auth

Every call is a `GET` with `api_key` in the query string. There is no header auth and no OAuth.

```
https://api.isportsapi.com/sport/football/livescores?api_key=<YOUR_API_KEY>
```

If `api.isportsapi.com` is slow or unreachable, retry the same path against `api2.isportsapi.com`.

## Steps

1. **Seed the snapshot.** `getFootballLivescores` (`GET /sport/football/livescores`) returns every match in
   play or finished today, with `status`, `homeScore`, `awayScore`, `homeHalfScore`, `awayHalfScore`, cards,
   corners and `updateTime`. Published limit: 1 second/call. Key each match by `matchId`.
2. **Then poll only the delta.** `getFootballLivescoresChanges`
   (`GET /sport/football/livescores/changes`) returns only the matches that changed since the last call.
   Poll this, not the full snapshot.
3. **Interpret `status`.** `0` not started, `1` first half, `2` half-time, `3` second half, `4` extra time,
   `5` penalties, `-1` finished, `-10` cancelled, `-11` TBD, `-12` terminated, `-13` interrupted,
   `-14` postponed.
4. **Compute the clock yourself.** Minutes elapsed = now − `halfStartTime`; add 45 for the second half.
   `halfStartTime` is 0 in any state other than 1 or 3.
5. **Pick up fixture changes separately.** Kick-off times move. `getFootballScheduleModify`
   (`GET /sport/football/schedule/modify`) lists matches whose schedule changed; re-pull those from
   `getFootballSchedule` (`GET /sport/football/schedule`, published limit 60 seconds/call, needs one of
   `date`, `leagueId` or `matchId` — never more than one).

## Rules

- All times are Unix epoch seconds in **GMT+0**. Render in the user's timezone, do not assume local.
- Errors do not use HTTP status. Check `code` in the body: `0` is success, `2` is
  `Invalid [api_key], illegal access.` A wrong path also returns HTTP 200 with `code` 2.
- Respect the published `x-rate-limit` on each operation in `openapi/isports-api-openapi.yml`. Exceeding it
  is reported in `message`, not in a header — there are no `RateLimit-*` headers to back off on.
- Historical `date` lookups on `/sport/football/schedule` are limited to the past month.
