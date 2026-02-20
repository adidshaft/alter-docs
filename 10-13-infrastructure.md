---
title: "Infrastructure & Scheduling"
parent: Algorithm Reference
nav_order: 13
---

## Section 13: Infrastructure & Scheduling

### 13.1 Nightly Matchmaker

| Component | Detail |
|---|---|
| Runtime | AWS Lambda (`alter-matchmaker`) |
| Schedule | EventBridge `cron(11 23 * * ? *)` = 11:11 PM UTC |
| Trigger | Nightly, no arguments |
| Output | Upserts into `matches` table |

### 13.2 Scouting Log Cron

Three pg_cron jobs call `generate-scouting-logs` via `pg_net.http_post`:

| Job name | Schedule (UTC) | Local time (IST) |
|---|---|---|
| `scouting-logs-noon` | 12:00 | 17:30 |
| `scouting-logs-afternoon` | 16:00 | 21:30 |
| `scouting-logs-evening` | 20:00 | 01:30 +1 |

### 13.3 Authentication Patterns

| Context | Auth method |
|---|---|
| User-triggered (onboarding, dossier refresh) | Bearer JWT (user's `access_token`) |
| Cron-triggered functions | `x-admin-secret` header |
| Matchmaker Lambda → Supabase | Service role key (env var) |
| iOS → Storage upload | Bearer JWT + `apikey` anon key |
| iOS → `analyze-profile` | Anon key (Edge Function uses service role internally) |

