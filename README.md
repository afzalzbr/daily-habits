# 📿 Daily Habits

A mobile-first Progressive Web App (PWA) for tracking daily habits — prayers, exercise, learning, reading, journaling, and spending. Data syncs across devices via Supabase.

**Live:** [daily-habits.afzalzubair.com](https://daily-habits.afzalzubair.com/)

## Features

- **Prayer tracking** — tap to toggle Fajar, Zuhar, Asr, Maghrib, Isha, Tilawat (yes / no / unset)
- **Daily activities** — free-text fields for Exercise, Learning, Reading, Notes/Journal, Money Spent
- **Dashboard** — current & best streak, days logged, weekly overview, per-prayer completion %, and a monthly heatmap
- **Cloud sync** — sign in with Google or email magic link; data follows you across devices
- **6 themes** — Slate, Retro, Midnight, Emerald, Rose, Light (in-app picker, saved per device)
- **CSV export** — download all data in the original spreadsheet column format
- **Offline-ready PWA** — installable to home screen, works without a connection
- **History locked** — entries start 1 June 2026; no future-dated logging

## Tech

- Single static `index.html` — vanilla JS, no build step
- [Supabase](https://supabase.com) — Postgres + Auth (Google OAuth + magic link)
- `localStorage` — offline cache + theme preference
- Service worker (`sw.js`) — offline caching
- Deployed on [Vercel](https://vercel.com)

## Project structure

| File | Purpose |
|------|---------|
| `index.html` | Entire app — UI, styles, logic |
| `manifest.json` | PWA metadata |
| `sw.js` | Service worker (offline cache) |
| `icon.svg` | App icon |

## Setup

### 1. Database

Run in Supabase → SQL Editor:

```sql
create table habits (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  date date not null,
  fajar text, zuhar text, asr text, maghrib text, isha text, tilawat text,
  exercise text, learning text, reading text, money text, notes text,
  unique(user_id, date)
);

alter table habits enable row level security;

create policy "own data only" on habits
  for all using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

### 2. Auth

- **Authentication → URL Configuration**
  - Site URL: `https://daily-habits.afzalzubair.com`
  - Redirect URLs: `https://daily-habits.afzalzubair.com/**`
- **Authentication → Providers**
  - Enable **Email** (magic link)
  - Enable **Google** — add OAuth Client ID + Secret from [Google Cloud Console](https://console.cloud.google.com); set the authorized redirect URI to `https://<project>.supabase.co/auth/v1/callback`

### 3. Config

Supabase URL and anon key are set near the top of the `<script>` in `index.html`. The anon key is safe to expose publicly — Row Level Security restricts each user to their own rows.

## Local development

```bash
npx serve .
```

Open the served URL in a browser.

## Data model

| Column | Type | Notes |
|--------|------|-------|
| Fajar, Zuhar, Asr, Maghrib, Isha, Tilawat | `yes` / `no` | Prayer toggles |
| Exercise, Learning, Reading, Notes/Journal, Money Spent | text | Free input |

Export produces a CSV with columns: `Sr. No, Date, Fajar, Zuhar, Asr, Maghrib, Isha, Tilawat, Excercise, Learning, Reading 10 mins, Notes/Journal, Money Spent`.
