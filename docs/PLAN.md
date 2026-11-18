# polis v2: plan

_Written 2026-11-13 from the team's brief, updated the same day with the team's answers to the decisions. Nothing in this plan is built yet._

This plan turns the brief into 8 phases. Each phase is sized to fit one working session (Phase 6 takes several) and leaves the app working, so you can stop after any phase.

**To start a phase**, open a session on this repository and say: _"Read docs/PLAN.md and do Phase N."_

## The brief

| # | What the team asked for | Phase |
|---|---|---|
| 1 | Sources: only trustworthy outlets (French public media, AFP, large non-partisan newspapers such as Le Monde). No reactionary or partisan media such as BFM or CNews. | 1 |
| 2 | Feed: learn preferences from **themes**, never from parties. A player who likes lots of energy news gets more energy news, with every party still shown on that theme. A player who likes lots of left-wing cards must not end up with only the left. | 4, 6 |
| 3 | Feed looks like a **newspaper**. Swipe cards away: **right = I like it, left = I don't**. Keep the challenges between cards. | 5 |
| 4 | A place to **check the programs**. If a program isn't published, say so. A **keyword search** finds the relevant part of a program (no chatbot). | 7 |
| 5 | Profile: a button that **deletes all of the player's information**. | 2 |
| 6 | Redesign the city to look better, **in the spirit of a mobile strategy game like Clash Royale**, on a smaller scale. | 8 |
| 7 | Overall look: **sober, mostly white, with touches of blue and red** (French flag). No top bar: a **floating avatar** top right opens the profile. Keep dark mode, with a **light / dark choice in the settings**. | 3 |
| 8 | **Liquid glass** bottom tab bar with 4 tabs: **Home, News, Challenges, City**. | 3 |

## Order and size

| Phase | What | Size | Needs |
|---|---|---|---|
| 1 | Sources policy and automatic check | S | — |
| 2 | "Delete all my information" | S | Run one SQL migration in Supabase |
| 3 | White design, liquid glass tab bar (4 tabs), floating avatar, light/dark setting | M | — |
| 4 | Feed algorithm by theme | M | — |
| 5 | Newspaper swipe feed | M–L | Phases 3 and 4 |
| 6 | More news cards (every theme × every camp) | L, several sessions | Phase 1 |
| 7 | Programmes section with keyword search | M + research | Phase 3 |
| 8 | City redesign | L | Decision D5 |

Why this order: the source rules come first because every later card and program must follow them. The data deletion is small and the law expects it, so it goes early. The design tokens come before the new screens, so those are built once in the new style. The algorithm comes before the swipe UI, which plugs into it.

## Decisions

Answered by the team on 2026-11-13, except D5.

| # | Question | Decision |
|---|---|---|
| D1 | Which sources? | **Public media, AFP, official sites, and large non-partisan newspapers such as Le Monde.** No partisan outlet on either side. Details in Phase 1. |
| D2 | Likes in the feed, and the left–right slider | **Keep like / dislike** (swipe right / left), counted per theme. The left–right slider in the profile goes: the new feed doesn't use it, and it was the most sensitive data in the app. |
| D3 | Navigation | **No top bar.** A floating avatar, top right, opens the profile. The bottom bar has **4 tabs: Accueil, Actu, Défis, Ville**. Programmes is reached from the home page and the challenges hub (Phase 7). |
| D4 | Program chatbot | **No chatbot.** A keyword search finds the relevant passages of a program (Phase 7). No API key, no cost, and searches never leave the phone. |
| D5 | City: automatic upgrades, or voix spent on the building of the player's choice? | _Not answered yet._ The plan assumes **automatic, with an "Améliorer" button** that plays the upgrade animation and shows the building's civic fact. Spending voix would mean reworking the levels, badges and `/stats`. |
| D6 | Dark mode | **Keep it, with a choice in the settings:** Automatique (follows the phone, the default), Clair, Sombre. The design is light-first. |

---

## Phase 1: Sources policy (S)

**Goal:** every source shown in the app comes from a trustworthy outlet, and the build refuses anything else.

**Today:** 17 citations break the rule.

| File | Item ids | Current source |
|---|---|---|
| `content/actu.json` | `melenchon-parcoursup`, `becht-extreme-centre` | Sud Radio |
| `content/actu.json` | `attal-pacte-jeunesse`, `rn-bardella-seul-candidat` | "AFP via La Gazette France" |
| `content/actu.json` | `platret-candidat` | Lyon Capitale |
| `content/qui-propose.json` | `qp-melenchon-parcoursup` | Sud Radio |
| `content/qui-propose.json` | `qp-attal-controle-continu`, `qp-attal-convention` | La Gazette France |
| `content/vrai-faux.json` | `vf-macron-mandat` | Sud Radio |
| `content/vrai-faux.json` | `vf-le-pen-candidate` | Europe 1 (same group as CNews) |
| `content/timeline.json` | `roussel-2018-pcf`, `attal-2024-renaissance` | Europe 1 |
| `content/quiz.json`, `content/vrai-faux.json` | `q-inscription`, `q-parrainages`, `q-proclamation`, `vf-inscription`, `vf-parrainages` | Toute l'Europe (an association, not a news outlet) |

**Steps**

1. Create `content/sources.json`, the allowlist, with one entry per domain (`domain`, `label`, `kind`):
   - **Agency:** AFP. An AFP dispatch counts only on an allowed site (e.g. "franceinfo avec AFP", "France 24 avec AFP"), never "AFP via" a private site.
   - **Public broadcasters:** franceinfo.fr, francetvinfo.fr, radiofrance.fr (France Inter, France Culture), france24.com, rfi.fr, ici.fr, lcp.fr, publicsenat.fr, arte.tv, ina.fr, lumni.fr.
   - **Official sources:** `*.gouv.fr`, conseil-constitutionnel.fr, assemblee-nationale.fr, senat.fr, elysee.fr, europa.eu, europarl.europa.eu, insee.fr, ccomptes.fr, caf.fr, campusfrance.org, local authority sites.
   - **Large non-partisan newspapers (D1):** lemonde.fr, lesechos.fr, la-croix.com, leparisien.fr, ouest-france.fr, and large regional dailies (sudouest.fr, lavoixdunord.fr, leprogres.fr…). Add one only if it is a general-interest daily with no declared political line.
   - **Candidates and parties (programs only):** official campaign and party sites. They are allowed in `content/programmes/` only, never as the source of a news card.
   - **Left out on purpose**, and listed in the file with the reason so nobody adds them back by mistake: outlets with a political line on either side. On the right: CNews, BFMTV, Europe 1, JDD, Sud Radio, Valeurs actuelles, Frontières, Boulevard Voltaire. On the left: L'Humanité, Politis, Le Média. Le Figaro and Libération lean clearly to one side each, so they are left out **as a pair**. If the team wants them one day, add both together.
2. Add `scripts/check-sources.mjs`. It walks every `content/**/*.json`, checks each URL's host against the allowlist, and exits with an error listing the offenders. Wire it as `"prebuild"` in `package.json`, so a bad source stops the Vercel deploy and the live site keeps the previous version.
3. Replace the 17 citations. Find the same facts on an allowed site (franceinfo, Public Sénat, LCP, France 24, ici, Le Monde…). If no allowed site has them, **remove the item**. Never keep an unsourced fact.
4. Log every change in `content/actu-notes.md` and `content/content-notes.md`, then update "Editorial rules" in the README.
5. In the "Pourquoi ces actus ?" sheet, add one line: "Nos sources : médias publics, AFP, sites officiels et grands quotidiens sans ligne partisane."

**Done when:** `npm run build` passes, the check script finds 0 offenders, and adding a `bfmtv.com` URL to any content file makes the build fail.

---

## Phase 2: Delete all my information (S)

**Goal:** one button in the profile erases the player's data **on the phone and on the server**.

**Today:** "Effacer toutes mes données sur ce téléphone" (`app/profil/ProfileClient.tsx`) only clears `localStorage`. These Supabase rows stay:

| Table | Linked by | What it holds |
|---|---|---|
| `accounts` | `session_id`, `email` | email, first name |
| `emails` | `session_id`, `email` | email, consent |
| `events` | `session_id` | every tracked event |
| `challenges` | `session_id` | first name, scores |
| `predictions` | `session_id` | answers |
| `match_players` | nothing | first name, scores (can't be linked to a player today) |

**Steps**

1. Add `app/api/me/route.ts` with `DELETE`. Body: `{ sessionId, email? }`. It deletes the `session_id` rows in `events`, `challenges`, `predictions`, `emails` and `accounts`. It also deletes `accounts` and `emails` rows for that email **only where the session id matches too**, so nobody can erase someone else's email by typing it. It returns what was deleted. Read the Next.js 16 route handler guide in `node_modules/next/dist/docs/01-app/` first.
2. Migration `supabase/migrations/2026-11-match-players-session.sql`, also folded into `schema.sql`: add `session_id` to `match_players`, fill it from the join and match routes, and include it in the deletion. Someone has to run this SQL in Supabase (Project › SQL Editor).
3. Profile: rename the button to **"Supprimer toutes mes informations"**. Show it on every profile screen, including the one for players who haven't finished their profile. The confirmation sheet lists what will be erased (on this phone: voix, city, trophies, profile; on our server: account, email, game history). On confirm: call the API, wipe local data (the existing `wipe()`), then go to `/`. If the server can't be reached, say so and keep the local data, so the player can try again.
4. Anonymous feed reactions can't be linked to anyone, so they can't be deleted. Say so on `/donnees-personnelles`, and add a line on how to ask for deletion by email (the contact email in `content/site.json` must be filled in first).
5. If a friend opens a challenge link whose creator deleted their data, check that their landing page still works ("ce défi n'existe plus").

**Done when:** after a full play-through with an email, pressing the button leaves no row with that session id in any table, and the app starts fresh.

---

## Phase 3: White design, liquid glass tab bar, floating avatar, light/dark setting (M)

**Goal:** sober, mostly white, blue for actions, red as a small accent. No top bar.

**Steps**

1. **Tokens** (`app/globals.css`): `--paper` becomes pure white. Cards are white with a hairline `--line` border and very light shadows, instead of white on grey. Keep blue `--accent` for actions and red `--rouge` for small accents (kickers, countdown, tricolour hairline). Dark mode keeps its own values.
2. **Remove the big navy blocks.** `bg-hero` appears 15 times in 9 files: `HomeClient`, `FeedClient`, `HubClient`, `OnboardingClient`, `VoterClient`, `FriseClient`, `ProfileClient`, `InlineChallenge`, `BadgeToast`. Replace each with a white card: blue title, red kicker, tricolour hairline.
3. Stay recognisably French without looking like a government app: no Marianne, no copy of the State's design system (DSFR).
4. **Liquid glass tab bar** (`components/BottomNav.tsx`) with **4 tabs: Accueil, Actu, Défis, Ville** (D3):
   - a floating capsule, 16px from the sides and 12px above the safe area, fully rounded;
   - `backdrop-filter: blur(20px) saturate(180%)` over a translucent white, a 1px light inner border, a soft shadow, and a top highlight gradient for the "glass" edge;
   - the active tab sits in a glass "lens" that slides between tabs with a spring animation;
   - extra refraction with an SVG displacement filter where Chromium supports it, and plain blur elsewhere (Safari);
   - honour `prefers-reduced-transparency` (solid white bar) and `prefers-reduced-motion`;
   - update every height that assumes the old bar (the feed's `h-[calc(...4.25rem...)]`, the spacer `div`).
5. **No top bar (D3).** Remove `components/TopBar.tsx` from `app/layout.tsx` and give each of its jobs a new home:

   | Top bar today | New home |
   |---|---|
   | Profile access (from the Profil tab) | **Floating avatar**, top right, in the same glass style as the tab bar, on every screen except onboarding. It opens `/profil`. |
   | Voix counter and "+10 voix" animation | A thin ring around the avatar shows progress to the next city level. When voix are earned, a "+10 voix" bubble pops out of the avatar for about 1.5 s. The total appears on the home page and in Ville. |
   | Back button on sub-pages (`/voter`, `/frise`, `/equipes`, `/quiz`, `/programmes/…`, `/donnees-personnelles`) | **Floating back button**, top left, same glass style. |
   | FR / EN switch | Settings, in the profile. |
   | Logo | The home page only. |
   | Tricolour stripe | A hairline at the very top of the home page. |

   The avatar is chosen from a set of drawn avatars (for example the city's buildings), with the first letter of the name as the default. **No photo upload:** a photo would be personal data to store and protect. Content must leave room for the floating buttons (top padding on every page).
6. **Settings** in the profile: Langue (Français / English), **Apparence: Automatique (default, follows the phone) / Clair / Sombre** (D6), and the "Supprimer toutes mes informations" button from Phase 2.
   - Store the choice in `localStorage` (`p27.theme`) and set `data-theme="light"` or `"dark"` on `<html>`.
   - Apply it **before the first paint**, with a tiny inline script in `app/layout.tsx`, so the page never flashes the wrong colours. Read the Next.js 16 guide on scripts in the root layout first (`cacheComponents` is on).
   - CSS: dark values under `@media (prefers-color-scheme: dark)` scoped to `:root:not([data-theme="light"])`, and again under `:root[data-theme="dark"]`.
7. Update `docs/DESIGN_HANDOFF.md` (tokens table, tab bar, avatar, no top bar, no navy blocks, theme setting).

**Done when:** iPhone-size screenshots of every tab (FR and EN, light and dark) show white screens with blue and red accents, the tab bar and avatar float as glass over scrolled content, the theme setting switches instantly with no flash on reload, and the build passes.

---

## Phase 4: Feed algorithm by theme (M)

**Goal:** preferences are learned **per theme**, and inside every theme all camps get equal turns.

**Today** (`lib/feed.ts`, `pickNext`): camps take turns, a 👍 on a camp pushes the *other* camps forward, and the optional left–right position decides which camp comes first. Themes only break ties inside a camp.

**New rule**

1. **Theme scores**, kept on the phone only (`p27.topicScores`): swipe right +1, swipe left −1, opening a card's details +0.5. Topics picked at onboarding start at +2. A swipe never affects anything about the camp.
2. **Choosing the next card:**
   1. *Theme:* chosen at random, weighted by score, so liked themes come up more often. Two guard rails: at least 1 card in 4 comes from outside the top 3 themes, and never more than 3 cards in a row on the same theme.
   2. *Camp, inside that theme:* the camp shown least on that theme so far. Ties go to the camp shown least overall.
   3. Keep a neutral card after every 2 or 3 camp cards. Never 2 cards in a row from the same camp when there is a choice.
   4. If the theme has no card from the camp whose turn it is, take that camp's card from another liked theme, so the camp balance holds across the session.
3. Likes stay (D2) but count for the theme only: remove the camp-based steering. Remove the left–right slider too (`lib/profile.ts`, profile and onboarding screens, the `ideas` text in `content/onboarding.json`), and delete the stored position from phones that have one.
4. Topics (`content/topics.json`): add **Énergie** (fuel taxes, electricity, nuclear) and review the other 12 so that no card is filed under a vague topic.
5. **Tests:** add `lib/feed.test.ts`, run with `node --test` through `tsx` (a new dev dependency, needed so the `@/` imports resolve). It simulates 300 swipes:
   - always right on left-wing cards: each camp still gets 20–30 % of camp cards;
   - always right on energy: energy reaches ≥ 40 % of cards, and the camps inside energy stay within ±1 card of each other.
6. "Pourquoi ces actus ?" sheet: show the player's top themes and a "Remettre à zéro" button.
7. Tracking: `feed_reaction` keeps no session id and now also carries `topic`. In `/stats`, show likes per topic next to likes per camp.

**Done when:** the tests pass, and both rules (theme preference and camp balance) are written in `docs/PRODUCT_NOTES.md`.

---

## Phase 5: Newspaper swipe feed (M–L)

**Goal:** each card looks like a newspaper clipping. Swipe right if you like it, left if you don't.

**Card design**
- White paper, a thin double rule at the top, a masthead line "polis · L'édition du jour · 13 nov. 2026".
- The theme as a small-caps section label in red; the date and a neutral camp chip.
- A **serif headline** (e.g. Newsreader or Source Serif 4, through `next/font/google`, which serves the font from our own domain: no third-party request) and sans-serif body text.
- The source as a byline: "Source : franceinfo".
- Tap the card to open the full article in a sheet: "Pour toi", what each camp says, the link to the voting guide, the source.

**Interaction**
- A stack of 3 cards. The top card follows the finger, tilts with the drag, and shows a "J'aime" (right) or "Pas pour moi" (left) stamp. Past 30 % of the width, or with a fast flick, it flies off and the next card comes forward.
- Buttons under the stack do the same for one-handed and accessible use: ✕ (left), ↺ (undo the last swipe), ♥ (right). The ← and → keys work too.
- Both directions count as "read" (+1 voix). Only the theme score changes (Phase 4).
- The quick challenge every 3 cards becomes a card in the stack that can't be swiped until it is answered or skipped. Keep `InlineChallenge`.
- On the first visit, a short coach mark explains right and left. Light haptics on each swipe.
- An "Fin de l'édition" card at the end, like today.
- No animation library: pointer events and CSS transforms. Under `prefers-reduced-motion`, a swipe becomes a fade.

**Files:** rewrite `app/feed/FeedClient.tsx` and split it (`components/feed/NewsCard.tsx`, `SwipeStack.tsx`). Remove the up and down arrows and the snap scroll.

**Check:** `/stats` still gets `feed_card_view` and `feed_reaction`, so the completion rate stays comparable with earlier tests.

**Done when:** a Playwright script swipes through 10 cards with mouse drags and buttons on an iPhone viewport, and the screenshots look like a newspaper.

---

## Phase 6: More news cards (L, several sessions)

**Goal:** enough cards for Phase 4 to work: **every theme × every camp**.

**Today:** 27 news cards, with large gaps. Several topics have no card at all from some camps:

| Topic | Neutral | Left | Centre | Right | Far right |
|---|---|---|---|---|---|
| election | 1 | 3 | 2 | 2 | 1 |
| budget | 1 | 1 | 1 | – | 1 |
| pouvoir-achat | 1 | 1 | 1 | – | – |
| etudes | 2 | 1 | 1 | – | – |
| securite | – | – | – | 2 | – |
| emploi | – | – | 1 | 1 | – |
| institutions | – | – | – | – | 2 |
| logement | 1 | – | – | – | – |
| climat, sante, immigration, europe, (énergie) | – | – | – | – | – |

**Target:** at least 1 card per theme × camp, and 2 for the themes students care most about (pouvoir d'achat, logement, études, emploi, climat/énergie). That is about 60–70 new cards.

**Rules:** allowed sources only (Phase 1). One fact per card, quoted or dated. Every card checked by opening its source and logged in `content/actu-notes.md`. Add the English version to `content/en/overlay.json`. If a camp hasn't said anything on a theme, **don't invent a position**: the algorithm already handles the gap.

**Optional:** a weekly scheduled session that drafts new cards for a person to review. Never publish a card automatically.

---

## Phase 7: Programmes section with keyword search (M + research)

**Goal:** one place to read each candidate's program, saying clearly when it isn't out yet, and a search that finds the relevant passages.

**Where it lives (D3):** a "Les programmes" card on the home page and in the challenges hub (next to the voting guide and the timeline). A news card about a candidate links to that candidate's program ("Voir son programme").

**Pages**
- `/programmes`: the search box on top, then every declared candidate (from `content/timeline.json`). Show them in alphabetical order with a note saying so, or shuffle them on each visit, so no camp is always on top. Each candidate has a status badge:
  - **Programme publié** (date, link to the official document);
  - **Premières propositions** (sourced proposals, nothing official yet);
  - **Pas encore publié** ("Dernière vérification : <date>"), with an optional "Préviens-moi quand il sort" (the existing email capture).
- `/programmes/[candidat]`: the program by theme, using the same topics as the feed. Each excerpt is quoted word for word, with its page and a link. If the program says nothing on a theme: "Le programme ne parle pas encore de ce thème."
- `/programmes/theme/[topic]`: every candidate's excerpt on one theme, one after the other, in the same order rule.

**Keyword search (D4, instead of a chatbot)**
- Two places: across all programs (on `/programmes`), and inside one program (on `/programmes/[candidat]`).
- The player types a word ("loyer", "retraite", "Parcoursup"). Results are the matching excerpts, grouped by candidate in the same neutral order, with the matching words highlighted and the page number. A tap opens the excerpt in the program, scrolled into view and highlighted, with a link to the official document at that page.
- Forgiving matching: accents and capitals ignored ("etudiant" finds "Étudiants"), word starts ("retrait" finds "retraite"), small typos. A small synonym list in `content/programmes/synonymes.json` (e.g. loyer → logement, APL; bourse → études; essence → carburant, énergie) widens the search.
- Suggested searches under the box: the feed's topics as chips (Logement, Pouvoir d'achat, Études…).
- If a candidate has no match: "Rien sur « loyer » dans le programme de X" rather than hiding them, so the absence is visible too. If the program isn't published, the candidate is listed with "programme pas encore publié".
- Built in the browser, from the program files: no server, no API key, no cost. Use a small search library such as MiniSearch (about 7 kB, handles prefixes, typos and accent folding), loaded only on these pages.
- Privacy: **search words never leave the phone.** Only a `programme_search` event with the candidate id and the number of results is tracked, never the words typed, because they can reveal an opinion.

**Data:** `content/programmes/<candidat>.json`:

```json
{
  "candidate": "glucksmann",
  "status": "publie | propositions | non-publie",
  "publishedOn": "2027-01-20",
  "document": { "label": "Programme officiel (PDF)", "url": "https://…" },
  "checked": "2026-11-13",
  "sections": [
    { "id": "logement-1", "topic": "logement", "title": "…", "text": "verbatim excerpt", "page": 12 }
  ]
}
```

**Research step:** for each candidate, find out whether a program is published (official site, then public media coverage). Extract the text from the PDF (the PDF skill can do this) and split it into excerpts by theme. Expect most programs to come out between January and March 2027, so plan a re-check before every user test.

**Done when:** every declared candidate has a file, a status and a "checked" date, the pages work in FR and EN, and searching "logement", "etudiant" (no accent) and "retrait" (a word start) returns the right excerpts for every published program.

---

## Phase 8: City redesign (L)

**Goal:** make the city feel like a real mobile game, with chunky buildings, levels and satisfying upgrades, while staying in the sober bleu-blanc-rouge look.

**Direction:** take the *feel* of a mobile strategy game, not its art. All drawings are original. No characters, assets, names or look-alikes from Clash Royale or any other game.

- A small **2.5D isometric island** of tiles, with soft shadows and gentle idle animations (flags waving, smoke, people walking).
- Each building has **levels 1–3**, drawn as 3 visual variants, with a level badge and a progress bar toward the next level.
- An **upgrade moment**: the "Améliorer" button glows when the threshold is reached. Tapping it plays a bounce and sparkle animation (+ haptic), the building grows, and its card opens with what that building does, who decides, and the source (from `content/city.json`).
- A **level road**: the path from Hameau to Capitale as a vertical road with the next rewards visible.
- Palette: white stone, slate-blue roofs, red flags and awnings, green trees. Night and dusk skies stay.

**Mechanics:** as in D5. Voix stay a score that never goes down, and buildings unlock at thresholds (`content/levels.json`). Upgrades are a moment the player triggers, not a purchase.

**Files:** rewrite `components/City.tsx` as `components/city/` (one SVG component per building and level), plus `app/ma-ville/CityClient.tsx`, `components/LevelUp.tsx`, and `content/city.json` (add per-level text). Keep the shared PNG export (`cityPng`) working.

**Done when:** screenshots at each city level look finished, the upgrade animation runs at 60 fps on a mid-range phone, and the shared image still renders.

---

## Rules for every phase

- Work on a branch (`claude/…`). **Never push to `main` without asking:** a push there redeploys the live site within a minute. Each branch gets its own Vercel preview link to try first.
- Before writing Next.js code, read the matching guide in `node_modules/next/dist/docs/` (Next.js 16 differs from older versions; `cacheComponents` is on).
- Run `npm run build` before every push.
- Take iPhone-size screenshots in FR and EN of every screen the phase touches.
- Add the English version of every new string (`t("…", "…")` or `content/en/overlay.json`).
- Never show a candidate or camp more prominently than another. Never infer or store a player's political leaning.
- At the end of a phase, add a line to `docs/DECISION_LOG.md` and update the README and `docs/PRODUCT_NOTES.md` or `docs/DESIGN_HANDOFF.md` where they describe what changed.
