# Product notes

Detailed choices behind polis. The [README](../README.md) has the overview; this file has the details.

## Profile and personalised feed

- `/bienvenue` (onboarding): name, "can you vote in France?", habits (follows the news, voted, registered), an optional 3-question level test, topics, a 3-screen tutorial, result. Every step can be skipped ("Passer" / "Tout passer"). The account email is optional and saved in `accounts` through `/api/account`.
- Level = right answers in the test + how often they follow the news → débutant / intermédiaire / avancé. Beginners get the "bases" cards first; advanced players go straight to the news. Not registered (or not sure) → the registration card comes first.
- Feed rule (`lib/feed.ts`): camps take turns (gauche, centre, droite, extrême droite) with a neutral card after every 2; the player's topics first inside each camp; 👍 on a camp → the next cards come from the camps with the fewest 👍; if the player gave a position, the camps farthest from it come first. Nothing is ever hidden: every camp is always shown.
- The left–right position is optional, set in the profile, and **never leaves the phone**.

## Challenges

- **Answers always explained**: after every answer, the player sees right/wrong, the right answer, why, and the source, then taps "Question suivante".
- **Difficulty**: every question has `difficulty` 1 (easy), 2 (medium) or 3 (hard). The player picks Progressif (default: starts easy, +1 level after 2 right answers in a row, −1 after a mistake), Facile, Moyen or Difficile. Logic in `lib/difficulty.ts`.
- A friend's link replays exactly the same questions (`item_ids` in `challenges`). Team matches use the same questions for everybody (easiest first).
- "Joue avec un ami" explains the 3 steps in the challenges page, the share sheet and the friend's landing page.

## International players and English

- Players who can't vote at the presidential election (EU / non-EU / not sure) get the "qui peut voter / pourquoi ça te concerne / élections étudiantes" cards (`audience: "intl"` in `feed.json`) and no registration reminders.
- FR/EN switch in the top bar and in the profile; default = browser language. UI strings are written inline as `t("français", "English")` (`lib/i18n.ts`). Content translations live in `content/en/overlay.json`, matched by id; anything missing falls back to French.

## Privacy

- The political position is never in any event. The server strips `position`, `ideas`, `leaning`, `opinion`, `side`, `pour` and `contre` from every event.
- The Pour / Contre answer in "Défends l'autre camp" stays in the component state: never stored, never sent.
- Feed 👍 / 👎 (`feed_reaction`) are stored **without session id or tester tag**: likes per camp can be counted, but never linked to a player.
- The account has an email and no password (one row per email, linked to the last session that used it). It is never used to write to players: the "prochains défis" emails (`emails` table) need the consent box ticked. First name optional.
- No cookies, no third-party trackers (only `localStorage`).

## Implementation choices not written in the spec

- `challenges` and `match_players` have an extra `total` column (to show "6/8").
- "Your team wins a match" (+50 voix) is granted when both teams have at least 2 finished players and your team's average is strictly higher, once per match.
- "A friend plays your challenge" (+50 voix) is checked each time the app opens.
- If a page is reloaded, the last result is kept for the tab only (`sessionStorage`).
