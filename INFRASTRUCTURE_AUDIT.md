# Infrastructure audit — Russian build

Date: 2026-11-16

## Scope and safety

This audit covers only this repository (`reaktsiya-russia`). The international repository and its branch are not modified. No cloud project, paid service, database, secret, or production setting was changed.

## Current frontend configuration

- The game HTML currently points to the Supabase project `rgxlnzhjdpmhteioosem` at `eu-central-1` (Frankfurt, Germany).
- The browser contains a Supabase publishable key. This is a public client key, not a service-role secret.
- Online leaderboard reads `public.leaderboard_scores` through the Supabase REST API.
- Score submissions go to the `submit-score` Edge Function.
- The local browser storage also keeps nickname, settings, coins, skins, achievements, XP, and local records.

## Data flow found

1. On opening the online leaderboard, the browser requests nickname, score, difficulty, and timestamp from the Supabase table.
2. On score submission, the browser sends nickname, score, and difficulty to the Edge Function.
3. The function reads the request's forwarded IP address, hashes it with SHA-256, and calls `allow_score_submission` to enforce a two-second submission cooldown.
4. The function inserts nickname, score, and difficulty; the database supplies the timestamp.
5. The local game progress and preferences are stored in the player's browser via localStorage.
6. VK Bridge is loaded from unpkg and used for VK app initialization and window resizing. The inspected code does not request a VK user ID.

A hashed IP is not automatically anonymous in every legal context; whether it is personal data depends on whether a person can be identified by the data and surrounding information. Nicknames can also identify a player.

## Existing backend inventory (international project; read-only inspection)

- Supabase project: `reaction-arcade`
- Organization plan: Free
- Region: `eu-central-1` (Frankfurt)
- Tables: `leaderboard_scores`, `leaderboard_rate_limits`
- Edge Function: `submit-score` (active, version 6)
- Database RPC: `allow_score_submission(text)`

No data was copied, deleted, or modified during this audit.

## Budget-first conclusion

Supabase's published region list does not include Russia. Creating another Supabase project, even on a free plan, would therefore not make this a Russian-hosted database. It would also introduce another project to maintain.

The lowest-cost temporary design is to make the Russian build local-only: retain local scores and game progress in the player's browser, and disable online leaderboard reads and submissions. This avoids sending leaderboard nickname/score/IP-derived data to the existing Frankfurt project. It also means there is no shared online leaderboard between players until a suitable backend is selected.

If a shared leaderboard is a must-have for the Russian release, a separately hosted backend in Russia needs to be selected and priced first. Do not assume a free trial or promotional credit is a permanent zero-cost service. The backend would need a database and a small server/API endpoint for safe score submission and rate limiting.

## Next implementation step

Before changing game behavior, confirm whether the Russian release may temporarily use a local-only leaderboard (zero new hosting cost), or whether a shared online leaderboard is mandatory even if it requires a paid Russian-hosted service.

## Important

Repository separation alone does not change where data is stored. Do not publish a privacy notice claiming Russian-only storage while the game still calls the Frankfurt Supabase project.
