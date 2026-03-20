# Kettle Web UI

Documentation for the Kettle Trading Floor web interface (`data/web/index.html`).  
Last updated: March 2026 (counterfactual lock gain, modal visual refresh, three-tier responsive layout, sticky header/ticker).

---

## Overview

The web UI is an S3-only feature, config-gated via `"web_server": true` in `config.json`. It is served by `KettleWebServer.cpp` using ESPAsyncWebServer + AsyncTCP (mathieucarbou forks) and is accessible at `http://kettle.local` via mDNS.

The UI is a single self-contained HTML file with no external dependencies at runtime. Google Fonts (`Inter Tight`, `JetBrains Mono`) are loaded from CDN on page load.

---

## Architecture

### File locations
| File | Location on LittleFS | Purpose |
|---|---|---|
| `index.html` | `/web/index.html` | Main UI — served as default for all GET `/` requests |
| `data.json` | `/web/data.json` | Rankings, quotes, payoff — written atomically on each market refresh |
| `status.json` | `/web/status.json` | Device health — written on each minute tick |

### Data flow
1. On each successful market refresh, `webserver_write_data_cache()` serializes rankings, payoff, quotes, and ticker data to `/web/data.json` using the atomic write-then-rename pattern.
2. `webserver_write_status_cache()` writes `/web/status.json` on every minute tick (uptime, heap, RSSI, battery, market state).
3. The browser polls both endpoints every 60 seconds via `fetch()` and re-renders all panels in place, preserving tab state on the leaderboard card.

### Auto-update
`httpFetchWebIndexIfChanged()` (called from `httpCheckFiles()` at startup) fetches `index.html` from GitHub using ETag-based conditional GET. If the file has changed, it is downloaded to `/web/index.html.new`, compared byte-for-byte with the existing file, and promoted via rename if different. No reboot required — the server picks up the new file on the next browser request.

> **Note:** `httpCheckFiles()` is skipped when `KETTLE_DEV_MODE 1` is set in `main.cpp`. Set to `0` for production to enable auto-update.

ETag state is persisted in `/etags.json` on LittleFS under the key `web_index`.

---

## API Endpoints

### `GET /api/config`
Returns a sanitized subset of the device's current runtime configuration. Sensitive fields (Wi-Fi credentials, API keys) are never returned. The Pushover user key is never returned; instead, a boolean `pushover_configured` indicates whether one is set.

**Response shape:**
```json
{
  "timezone": "America/Chicago",
  "muted": false,
  "supported_timezones": ["America/New_York", "America/Chicago", "America/Denver", "America/Phoenix", "America/Los_Angeles"],
  "rotation": {
    "enabled": true,
    "grid_delay": 60,
    "view_delay": 15
  },
  "notifications": {
    "wifi_connect": true,
    "new_lock": true,
    "midday_update": true,
    "after_hours_update": true,
    "eod_stats": false,
    "lock_daily_change": {
      "send": true,
      "daily_threshold": 7.5,
      "total_threshold": 60.0
    }
  },
  "pushover_configured": true
}
```

### `POST /api/config`
Accepts a JSON body with any subset of the editable fields, validates and applies them to the live `g_cfg` in memory, and writes the full `config.json` back to SD via the atomic write-then-rename pattern.

**Fields accepted:**
| Field | Type | Notes |
|---|---|---|
| `timezone` | string | Must be one of the `supported_timezones` values |
| `muted` | bool | |
| `rotation.enabled` | bool | |
| `rotation.grid_delay` | int | Must be >= 0 |
| `rotation.view_delay` | int | Must be >= 0 |
| `notifications.wifi_connect` | bool | |
| `notifications.new_lock` | bool | |
| `notifications.midday_update` | bool | |
| `notifications.after_hours_update` | bool | |
| `notifications.eod_stats` | bool | |
| `notifications.lock_daily_change.send` | bool | |
| `notifications.lock_daily_change.daily_threshold` | float | Clamped 0-100 |
| `notifications.lock_daily_change.total_threshold` | float | Clamped 0-500 |

All fields are optional — only fields present in the body are applied.

**Success response:** `{ "ok": true }`

**Error responses:**
- `400` — `{ "ok": false, "error": "Invalid JSON" }` — body could not be parsed
- `400` — `{ "ok": false, "error": "Unsupported timezone" }` — timezone not in supported list
- `200` — `{ "ok": false, "error": "Settings applied but SD write failed — changes will not survive reboot" }` — in-memory update succeeded but SD write failed

**Implementation notes:**
- Timezone changes are applied live via `setenv("TZ", ...)` + `tzset()` — no reboot required
- All other field changes take effect immediately in `g_cfg`; downstream code (Pushover, Bell, Rotation) reads `g_cfg` at event time so changes are effective on the next event
- The body accumulator uses a `static String` — safe for this single-user device but not concurrency-safe
- Fields not included in the POST body are left unchanged in memory and are preserved in the SD write (the full config is always reconstructed from `g_cfg`, not patched on disk)

---

## UI Structure

### Header
- **Logo** — `K·ETTLE` wordmark
- **Meta bar** — Period (doubles as a hidden download link for `history.json` — intentional back-door, not exposed to members), Owner, Last Updated timestamp
- **Payoff hero** — Owner's net payoff in large type; clickable to open Payoff Detail modal
- **S&P inline** — Current index value with daily change amount and daily percentage (sourced from FMP `change` / `changePercentage` fields via `g_spChange` / `g_spChangePct` globals — not the period-based computed change)
- **Market status badge** — Animated blinking dot + OPEN / static circle CLOSED
- **Gear icon** — Opens the Settings modal

### Ticker
- Scrolling ticker bar across the top showing all watchlist symbols with price and daily % change
- Duplicated content for seamless loop
- Speed: 105s per cycle; Font: 12px JetBrains Mono

### Main grid
3 rows x 2 columns:

| Left | Right |
|---|---|
| Leaderboard / Bet Summary | S&P Prediction |
| Gainer | Loser |
| Lock | Basket |

### Leaderboard card
- Tab strip: Leaderboard | Bet Summary
- Leaderboard tab: rank, member name, points (gold), basket % with mini bar
- Bet Summary tab: sorted by net position; shows bets, fines, net per member
- Tab state persists across data refreshes — switching to Bet Summary and leaving it there is safe

### Pick cards (Gainer, Loser, Lock)
- Rank, member, ticker (Yahoo Finance link), open price (hidden on narrow screens), current price, % gain
- Lock badge LK shown for locked picks; split badge SP shown when a split adjustment has been applied
- **Counterfactual tap (Lock card only):** For members who have locked their pick, the gain cell is tappable. Tapping toggles between the locked gain (used for scoring) and the hypothetical gain had they not locked, calculated as (current_live_price - season_open) / season_open. The flipped state is shown in muted italic with a reverse-arrow indicator. Cursor changes on hover; no underline — intentionally subtle. Resets to locked gain on the next 60-second data refresh.
- `live_price` is emitted in `data.json` only for locked lock entries, via a dedicated write path in `KettleWebServer.cpp` (separate from the shared `write_pick_array` lambda used for Gainer/Loser).

### Basket card
- Rank, member, three ticker symbols (each a Yahoo Finance link), average % gain with mini bar

### S&P Prediction card
- Rank, member, season target value, % distance from current index

---

## Modals

All modals share the same visual scheme: blue-gradient dark background, blue glow border, bouncy entrance animation. All dismiss on Escape or click-outside.

### Payoff Detail modal
- Triggered by clicking the payoff hero number in the header
- Shows owner's net position and itemised breakdown (ante, category wins/losses, fines)

### Member Card modal
- Triggered by clicking the owner's name anywhere it appears (header meta bar, or any highlighted row across all six panels)
- Shows owner's rank and performance value in each of five categories: Leaderboard (points, bar scaled to season max of 35), Gainer, Loser, Lock, Basket
- Each row: category label, rank (gold/silver/bronze coloured), proportional bar, value
- Leaderboard bar is always green (points are never negative); scaled to 35pt maximum
- Pick category bars are scaled relative to the best performer in that category
- Owner-only by design. Other members' names are not clickable.

### Settings modal
- Triggered by the gear icon in the header
- Fetches current config from GET /api/config on open; renders form fields populated with live values
- Sections: General (mute audio, timezone), Display Rotation (auto-rotate, grid dwell, view dwell), Pushover Notifications (per-event toggles), Lock Change Alerts (enable + thresholds)
- Pushover configured status is shown as informational text (key is never displayed)
- SAVE button POSTs the collected form state to POST /api/config
- Status line shows Saved. (green) on success, or the error message (red) on failure
- SD write failure is reported to the user; in-memory changes remain applied for the current session

---

## Styling

### Typography
| Context | Font | Size |
|---|---|---|
| Body / tables | Inter Tight | 16px base, td 15px |
| Monospace (values, meta, ticker) | JetBrains Mono | varies |
| Card titles | Inter Tight | 12px, 700, uppercase |
| Ticker | JetBrains Mono | 12px (11px mobile) |

### Colour palette
```
--bg:        #111213   Page background
--bg2:       #161718   Card / header background
--bg3:       #1c1d1f   Table header rows
--line:      #252628   Borders
--text:      #e8e9ea   Primary text
--muted:     #7c7f87   Secondary / labels
--dim:       #4a4d54   Tertiary
--green:     #22c55e   Positive values
--red:       #ef4444   Negative values
--blue:      #3b82f6   Accent / owner highlight
--gold:      #f59e0b   Rank #1 / points
```

### Rank colouring
Applied consistently across all panels and the member card:
- #1 — gold (#f59e0b)
- #2 — silver (#94a3b8)
- #3 — bronze (#cd8b3b)
- #4+ — muted

### Owner row highlighting
Owner rows have a blue left-edge accent and a dark blue tint background. Owner names are rendered as clickable spans — cursor changes on hover, no underline, consistent with other hidden interactive elements (period download link, counterfactual tap).

---

## Responsive behaviour

Three layout tiers, all validated on real devices:

### Desktop (>1024px)
- 3×2 grid layout, all columns visible
- Full header: logo, meta bar (single row), payoff hero, S&P inline, market badge, gear icon
- Ticker and header are sticky — both remain visible while scrolling the main content
- Basket card shows avg gain value + mini bar

### Tablet (641px–1024px)
- 3×2 grid layout retained (no collapse)
- Header: logo shrinks to 18px; meta bar wraps to two lines (Period + Owner on line 1, Updated centred on line 2); S&P number shrinks to 18px; "tap for detail" hint hidden; padding tightened
- Open price column hidden in Gainer, Loser, Lock cards
- Basket mini bar hidden (value only)
- Ticker and header remain sticky

### Mobile (≤640px)
- Single column grid (all cards stack vertically)
- Header reflows to three rows: logo + controls (row 1), meta bar (row 2), payoff + S&P (row 3)
- Market badge and gear icon grouped in top-right corner; gear scales down to 36px
- Open price column hidden in pick tables
- Ticker and header are sticky
- Further font size reduction at ≤380px

---

## Known open items

- **Member drill-down** — currently owner-only; no plans to extend to other members
- **History bump chart** — planned for when sufficient daily history has accumulated; will use Chart.js rendered inline; history files at /history/YYYYMMDD.json on LittleFS
- **Bet Summary surfacing** — currently a tab on the leaderboard card; final placement TBD

---

## File update checklist

When `index.html` is updated:
1. Place updated file at `data/web/index.html` in the firmware repo
2. Commit and push to GitHub
3. The device will pick up the change at next boot (with KETTLE_DEV_MODE 0) via ETag check
4. Confirm in serial log: `Web: index.html updated from GitHub.`
5. If ETag is stale (304 returned despite changed content), delete `/etags.json` from LittleFS to force re-download
