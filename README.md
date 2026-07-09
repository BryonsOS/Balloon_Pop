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

## Leaderboard

The leaderboard is **already set up and live**. Scores are stored in the
`balloon_scores` table of the Supabase project, and the project URL +
publishable key are configured at the top of the `<script>` block in
`index.html`. The publishable key is safe to ship — row-level security
policies only allow reading scores and inserting new ones.

After a solo game ends, players can enter a name (max 12 chars) and save
their score; the 🏆 Leaderboard screen shows the global top 20.

The table was created with this migration (for reference):

```sql
create table public.balloon_scores (
  id uuid primary key default gen_random_uuid(),
  alias text not null check (char_length(alias) between 1 and 12),
  score int not null check (score >= 0 and score < 1000000),
  created_at timestamptz not null default now()
);

alter table public.balloon_scores enable row level security;

create policy "public read" on public.balloon_scores
  for select using (true);

create policy "public insert" on public.balloon_scores
  for insert with check (true);
```

> **Note on cheating:** since scores are submitted from the browser, a
> determined person could post a fake score with curl. For a casual
> friends-and-family leaderboard this is fine. If it becomes a problem,
> the fix is a Supabase Edge Function that validates submissions.
