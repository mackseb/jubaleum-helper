# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file wedding day helper app (`index.html`) for Julia & Sebastian's wedding (27 June 2026). All HTML, CSS, and JavaScript lives in one file — no build tools, no dependencies, no framework.

## Running locally

```bash
python3 -m http.server 4178
# open http://localhost:4178
```

The `.claude/launch.json` configures this server. To preview a specific point in time, append `?t=2026-06-27T14:10` to the URL — this bypasses the real clock. The live banner auto-refreshes every 60 seconds in real-time mode only.

## Architecture

The file is split into two clearly marked zones:

**Data zone** (`GROUPS`, `PEOPLE`, `SECTIONS`, `SCHEDULE` — marked "DATEN — hier alles bearbeiten") — intended for non-technical edits. No programming knowledge required to add/remove tasks or people.

**Logic zone** (everything after — marked "LOGIK — ab hier nichts ändern") — the rendering engine; not meant to be changed for content updates.

### Data model

- `GROUPS` — role groups with a display name and hex color
- `PEOPLE` — persons with their group memberships (one person can belong to multiple groups)
- `SECTIONS` — ordered day/phase buckets (`key` ties to `SCHEDULE.sec` and `SEC2DAY`)
- `SCHEDULE` — ordered array of tasks: `sec`, optional `time` (HH:MM), `title`, `who` (array of names; `"?"` means unassigned), optional `details`, optional `open: true`

### Timestamp logic

Timed items (`it.time` set) anchor the schedule. Untimed items between two anchors get timestamps interpolated linearly. Untimed items before any anchor are placed 1 minute before the first anchor. `it.endTs` is derived from the next item's `ts` (or +30 min for the last). Items with times past midnight (e.g. `2:00`) that follow afternoon times are advanced by one day automatically.

`SEC2DAY` maps section keys to calendar day keys (`di`/`mi`/`do`/`fr`/`sa`/`so`). `ALLDAY` sections (`di`, `mi`, `do`, `fr`, `so`) have no timestamps — their status is day-level only.

### Status & rendering

`statusOf(item, now)` returns `"done"` / `"current"` / `"upcoming"`. `render()` is the single function that redraws everything from the `filter` state object and the current time — call it to reflect any state change. `setFilter(type, value)` updates the filter and calls `render()`.

Filter types: `"all"`, `"group"` (by group key), `"person"` (by name), `"open"` (unassigned tasks).
