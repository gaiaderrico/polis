<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="public/brand/polis-lockup-white.png">
    <img src="public/brand/polis-lockup.png" alt="polis" width="320">
  </picture>
</p>

<p align="center">
  <b>Comprends la présidentielle 2027 en 5 minutes.</b><br>
  A short, neutral game that helps students follow the 2027 French presidential campaign.
</p>

<p align="center">
  <a href="https://polis-2027.vercel.app"><b>▶ Play: polis-2027.vercel.app</b></a>
  &nbsp;·&nbsp;
  <a href="polis-demo-105bpm.mp4">Demo video</a>
  &nbsp;·&nbsp;
  <a href="https://www.instagram.com/polis_app/">Instagram</a>
</p>

---

## What it is

polis is a student project (ESSEC, Innovation Process course, 2026–2027). Players read short news cards about the campaign, play quick challenges, grow their own city with the points they earn, and challenge their friends.

It tests one assumption: **students want to learn about the 2027 campaign through a short game.** Pass threshold (Test Card v2): at least 40 % of testers finish at least one challenge.

**Principles:** facts only, a source for every item, every political camp shown equally. The app never recommends a candidate, and the player's political position never leaves their phone.

## Features

| | |
|---|---|
| 📰 **Feed** | Short, sourced news cards. Camps take turns (left, centre, right, far right) so nobody sees only one side. |
| 🎯 **Challenges** | Mini-quiz, Vrai ou Faux, Qui propose ça ?, Défends l'autre camp, Pronostics, plus a daily challenge. Every answer is explained with its source. |
| 🏙️ **Ma ville** | Points ("voix") grow a city. Each building explains what it does and who decides. |
| 👥 **Play with friends** | Send a challenge link (same questions for your friend) or play team matches. |
| 🗳️ **Voting guide** | `/voter`: deadlines with a countdown, checklists, official sources, calendar export. |
| 🌍 **FR / EN** | Full English version, and a mode for players who can't vote in France. |

## Tech stack

Next.js 16 (App Router) · TypeScript · Tailwind CSS 4 · Supabase (free tier, server-side only) · Vercel

## Run locally

```bash
npm install
cp .env.example .env.local   # fill it in, see below
npm run dev                  # http://localhost:3000
```

Without Supabase the app still runs: you can read the feed, play every solo challenge and grow your city. Nothing is saved on the server. Friend links, team matches and `/stats` need Supabase.

### Environment variables

| Variable | What |
|---|---|
| `NEXT_PUBLIC_SITE_URL` | Public URL of the site, used in challenge links (e.g. `https://polis-2027.vercel.app`). Baked in at build time: redeploy after changing it. |
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service key (server only, never commit it) |
| `STATS_PASSWORD` | Password for the `/stats` page |

### Set up Supabase (10 minutes)

1. Create a free project at <https://supabase.com>.
2. In **SQL Editor › New query**, paste `supabase/schema.sql` and click **Run**.
3. In **Project Settings › API**, copy the **Project URL** into `SUPABASE_URL` and the **service_role** key into `SUPABASE_SERVICE_ROLE_KEY`.

Row Level Security is on for every table, with no policy. The browser never talks to Supabase: every read and write goes through `app/api/*` with the service key, so `accounts`, `emails`, `events` and `predictions` can never be read from the client.

## Deploy

The repository is connected to Vercel: **every push to `main` redeploys <https://polis-2027.vercel.app>** (about 1 minute).

- Run `npm run build` before pushing. If the build fails locally, the deploy fails too and the live site keeps the previous version.
- To try a change without touching the live site, push to another branch: Vercel creates a separate preview link.
- Secrets live in Vercel and in each person's `.env.local`. Never commit them.

To deploy a fresh copy: import the repo on <https://vercel.com/new> (framework: Next.js, nothing to change), add the 4 environment variables, deploy.

## Edit the content (no code)

Everything players read is in `content/*.json`. Edit, commit, push: Vercel redeploys.

| File | What |
|---|---|
| `actu.json` | **Current news cards** (main feed). `camp`: `neutre`, `gauche`, `centre`, `droite`, `extreme-droite`. `topic`: an id from `topics.json`. Optional `why`, `sides`, `quiz`. |
| `feed.json` | "Les bases" cards (`kind: "bases"`, shown by level) and some candidate cards (`kind: "actu"`). |
| `topics.json` | Topics (label + emoji) and camp labels. |
| `onboarding.json` | Profile questions, level test, level texts. |
| `city.json` | City buildings: what each does, who decides, "pour toi", source, tap area. |
| `quiz.json` | Mini-quiz questions (`answer` = index of the right option, from 0). |
| `vrai-faux.json` | Vrai ou Faux statements (`answer: true` = Vrai). |
| `qui-propose.json` | Qui propose ça ? (`answer` = index; options are shuffled on screen). |
| `autre-camp.json` | Défends l'autre camp: one argument `pour`, one `contre`, one `offTopic` and its correction. |
| `pronostics.json` | Prediction questions. |
| `levels.json` | City levels and level-up messages. |
| `defis.json` | Challenge hub cards, durations, modes, result lines. |
| `badges.json` | Badges (trophées). Unlock rules are in `lib/progress.ts`, matched by `id`. |
| `guide.json` | Voting guide (`/voter`): deadlines (ISO dates), checklists, sources. Update `checked` when you re-verify it. |
| `timeline.json` | Campaign timeline (`/frise`). |
| `site.json` | App name, Instagram link, **team contact email for the privacy page (still to fill in)**. |
| `en/overlay.json` | English translations, matched by id. |

Every question has a `difficulty` from 1 (easy) to 3 (hard). Sources and fact-checks are logged in `content/actu-notes.md` and `content/content-notes.md`.

**Editorial rules:** facts only, a source for every item, as many items about the left as about the right and centre, nothing still uncertain. **Re-check every card the day before a test.**

## Measuring the test

- `/stats` (password = `STATS_PASSWORD`) shows every metric, with a CSV export of sessions.
- `?t=T03` on a link tags the session for an in-person test.
- `?src=instagram` (or `qr`, `whatsapp`…) records where people came from, e.g. `https://polis-2027.vercel.app/?src=instagram` for the Instagram bio. Never use `?t=` on a public channel.

| Metric | How it is computed |
|---|---|
| **Completion rate** (primary, pass ≥ 40 %) | Sessions with at least one `challenge_finish` ÷ sessions that opened the app. Computed on tagged testers when some sessions have a `?t=` tag. |
| **Share rate** (secondary, ≥ 40 %) | Sessions with a `challenge_finish` **and** a `share_click` ÷ sessions with a `challenge_finish`. |
| **Real sends** | `friend_landing_view` and `friend_finish` per challenge code. |
| **Engagement** | Players active on 2+ days, daily challenge started/finished, badges unlocked, player feedback (1–5 + comment). |

## Project structure

```
app/                 pages and API routes (app/api/*)
components/          UI: City (SVG), ChallengePlayer (challenge engines), ShareSheet, TopBar…
lib/                 content loader, points rules, feed rule, difficulty, i18n, tracking
lib/server/db.ts     Supabase client (service key, server only)
content/             all editable content (JSON)
supabase/schema.sql  database tables + RLS
public/brand/        logos
docs/                decision log, design handoff, brand, product notes
```

## More documentation

- [docs/PRODUCT_NOTES.md](docs/PRODUCT_NOTES.md): feed rule, onboarding, difficulty, privacy, implementation choices
- [docs/DECISION_LOG.md](docs/DECISION_LOG.md): decisions and the evidence behind them
- [docs/DESIGN_HANDOFF.md](docs/DESIGN_HANDOFF.md): design handoff
- [docs/brand/README.md](docs/brand/README.md): logo files and how to use them
