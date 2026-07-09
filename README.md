# 🎈 Balloon Tap!

A tap-to-keep-alive balloon game with a solo mode, a 2-player online volleyball
mode (peer-to-peer, room codes), combo multipliers, and an optional global
leaderboard.

**Play it:** open `index.html` in any browser, or visit the GitHub Pages site.

## Modes

- **Solo** — Keep the balloons off the ground. A new balloon joins every
  10 seconds. Chain taps within 2 seconds to build a combo (×2, ×3, …) —
  each tap scores its combo value. 3 lives.
- **2 Player** — One balloon, a net in the middle, first to 5 points.
  One player hosts and shares a 6-character room code; the other joins
  from their own device. Connection is peer-to-peer (PeerJS) — no server.

## Leaderboard setup (optional, one-time)

The leaderboard stores aliases + scores in a free [Supabase](https://supabase.com)
project. Until configured, the leaderboard button shows a "not set up" notice
and everything else works normally.

### 1. Create a Supabase project

Sign up at [supabase.com](https://supabase.com) (free tier is plenty) and
create a new project.

### 2. Create the scores table

In your project's **SQL Editor**, run:

```sql
create table public.scores (
  id uuid primary key default gen_random_uuid(),
  alias text not null check (char_length(alias) between 1 and 12),
  score int not null check (score >= 0 and score < 1000000),
  created_at timestamptz not null default now()
);

alter table public.scores enable row level security;

-- anyone can read the leaderboard
create policy "public read" on public.scores
  for select using (true);

-- anyone can submit a score (anonymous inserts)
create policy "public insert" on public.scores
  for insert with check (true);
```

### 3. Paste your keys into the game

In `index.html`, near the top of the `<script>` block:

```js
const SUPABASE_URL      = 'https://YOUR-PROJECT-REF.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';
```

Both values are in your Supabase dashboard under
**Project Settings → API**. The anon key is safe to publish — it only allows
what the policies above permit (read scores, insert scores).

That's it. After a solo game ends, players can enter a name (max 12 chars)
and save their score; the 🏆 Leaderboard screen shows the global top 20.

> **Note on cheating:** since scores are submitted from the browser, a
> determined person could post a fake score with curl. For a casual
> friends-and-family leaderboard this is fine. If it becomes a problem,
> the fix is a Supabase Edge Function that validates submissions.
