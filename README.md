# BBX Ilocos — Tournament Tracker

A static, no-login website for tracking **Beyblade X** tournaments in the Ilocos region: live rankings, tournament history, player stats, announcements, and community info.

There is no backend and no build step. The site is plain HTML, CSS, and JavaScript, and it reads its data live from a Google Sheet. Updating the sheet updates the site.

> Beyblade X is a trademark of its respective owners. This is an unofficial community project.

---

## Features

| Page | File | What it does |
|---|---|---|
| **Home** | `index.html` | Hero, latest announcements, community highlights carousel (swipe + auto-rotate), partner logos, quick links |
| **Rankings** | `rankings.html` | Blader of the Week spotlight; **This Week / Swiss Ranking / Award Ranking** leaderboards; a mirrored **Juniors' Corner** (Recent Tournament / Swiss Ranking / Award Ranking); "See All" pagination (20 shown by default) |
| **Tournament History** | `tournaments.html` | Every event, newest first, with a link to its Challonge bracket |
| **Player Stats** | `player.html` | Searchable blader profiles (name or alias), win rate, Swiss record, topcut attendance, career achievements. Shareable via `?name=` |
| **Information** | `information.html` | Community links, teams, and the official rules |

## Tech stack

- **HTML / CSS / vanilla JavaScript**, with no framework and no bundler
- **[PapaParse](https://www.papaparse.com/)** (loaded from cdnjs) for CSV parsing
- **Google Sheets** as the data store, read through the `gviz/tq` CSV endpoint
- **Google Fonts**: Saira Condensed (display), Inter (body), IBM Plex Mono (stats and labels)
- **Netlify** for hosting

## Project structure

```
/
├── index.html          # Home
├── rankings.html       # Rankings + Juniors' Corner
├── tournaments.html    # Tournament history
├── player.html         # Player stats
├── information.html    # Community, teams, rules
└── README.md
```

Each page is self-contained: its own `<style>`, its own `<script>`, and a small `CONFIG` object near the top of the script. All five pages share the same nav, hero style, and design tokens.

## How it works

```
Google Sheet ──(gviz/tq CSV)──▶ fetch() ──▶ PapaParse ──▶ rows → objects ──▶ render to DOM
```

1. On load, each page fetches the CSV export of the tabs it needs.
2. Rows are parsed with PapaParse. **Columns are matched by header name** (trimmed, case-insensitive), so column order in the sheet doesn't matter, but the header text does.
3. Data is sorted, filtered, and rendered client-side.
4. If an image link is missing or broken, the page falls back to a built-in default (initials avatar, SVG logo, plain hero background) instead of showing a broken image.

### Ranking rules

- **This Week** ranks Champion, then Swiss King, then Finalist, then 3rd Place, then Placer, then everyone else by points. Rows with a 0-0 Swiss record **and** 0 points are treated as non-participants and hidden.
- **Swiss Ranking** sorts by total points.
- **Award Ranking** sorts by Championships, then Swiss Kings, then Finalists, then 3rd Places, then Placers, with ties broken alphabetically.
- The **Juniors' Corner** uses the same three views and the same rules.

## Data contract (Google Sheet)

The sheet must be shared as **"Anyone with the link → Viewer"**. Use the `gviz/tq?tqx=out:csv&gid=<GID>` URL form. Do **not** use "Publish to web" links: they fail with CORS errors in real browsers.

| Tab | Columns the site reads |
|---|---|
| **Leaderboard** (Open, all-time) | `Name`, `Team`, `Photo URL`, `Swiss Wins`, `Swiss Losses`, `Topcut Attendance`, `Total Points`, `Championships`, `Finalists`, `3rd Places`, `Placers`, `Swiss Kings`, *(optional)* `Alias` |
| **Weekly Leaderboard** | `Name`, `Team`, `Photo URL`, `Swiss Wins`, `Swiss Losses`, `Topcut Attendance`, `Award`, `Swiss King`, `Points` |
| **Juniors All-Time** | Same as Leaderboard. The achievement columns and `Alias` are optional. |
| **Juniors Recent Tournament** | Same as Weekly Leaderboard |
| **Tournaments** | `Tournament Title`, `Date`, `Challonge Link` |
| **Announcements** | `Date`, `Title`, `Body`, `Photo URL`, `Link URL`, `Link Label`, `Week` |
| **Partners** | `Partner Name`, `Logo URL`, `Link URL` |
| **Image Repository** | `Key`, `Image URL` |

Recognised `Award` values (case-insensitive): `Champion`, `Finalist`, `3rd Place`, `Placer`. `Swiss King` is `Yes` or blank.

### Image Repository keys

| Key | Used for |
|---|---|
| `site_logo` | Nav logo on every page |
| `hero_bg_landing`, `hero_bg_rankings`, `hero_bg_tournaments`, `hero_bg_player_stats`, `hero_bg_information` | Hero backgrounds |
| `highlight_1` … `highlight_5` | Home page highlights carousel |
| `team_logo_8nineties`, `team_logo_ilocos_elites`, `team_logo_isur`, `team_logo_mmsu` | Team logos on Information |
| `facebook_icon`, `messenger_icon` | Community card icons on Information |

Any key left blank simply uses the default fallback.

## Configuration

Every page has a `CONFIG` object at the top of its `<script>`:

```js
const SHEET_ID = "<your-sheet-id>";
const CONFIG = {
  leaderboardCSV: `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:csv&gid=<GID>`,
  // ...one entry per tab the page reads
};
```

To point the site at your own sheet, change `SHEET_ID` and each `gid` in **all five pages**. A tab's `gid` is the number after `#gid=` in the sheet's URL when that tab is open.

## Local development

There is nothing to install or build. Serve the folder with any static server so that `fetch()` behaves as it does in production:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Deployment (Netlify)

The site needs no environment variables and no secrets.

- **Drag and drop:** upload the folder in Netlify's deploy UI.
- **From GitHub (continuous deployment):** connect the repo with the build command left **empty** and the publish directory set to `.` (the repo root). Optionally commit a `netlify.toml`:

  ```toml
  [build]
    publish = "."
  ```

Netlify serves the pages as-is, and no redirects or functions are needed.

## Design

Dark theme with a grey and orange palette. Colours, radii, and fonts are CSS custom properties declared in `:root` at the top of each page (`--bg`, `--surface`, `--orange`, `--text`, `--display`, `--body`, `--mono`, and so on). The layout is responsive, and the leaderboards collapse into stacked cards below 860px.

## Roadmap

- Tournament seasons: separate seasons, with partial data (top 10 plus achievements) for past ones
- Team achievements and team tournament results
- Alias support on Player Stats (code is in place; needs sheet columns)
- Juniors award tracking (code is in place; needs sheet columns)
