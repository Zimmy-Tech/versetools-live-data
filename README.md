# versetools-live-data

Standalone, hand-maintained runtime data for [VerseTools](https://github.com/Zimmy-Tech/versetools).

**Why this repo exists:** the main site is rebuilt by DigitalOcean on every push, so
correcting a value there means a full rebuild + redeploy. The values here are read by
the live site **at runtime** over `raw.githubusercontent.com`, so a one-file edit here
takes effect **without any site rebuild**. Keep this repo **public** — the app fetches
it unauthenticated; a private repo would 404 and the app would fall back to its
baked-in defaults.

## `exec_hangars_anchor.json`

Overrides the baked-in anchor for the Pyro Executive Hangar door timer on
`/exec-hangars`. The app reads it at:

```
https://raw.githubusercontent.com/Zimmy-Tech/versetools-live-data/main/exec_hangars_anchor.json
```

### Schema

| Field         | Type            | Meaning |
|---------------|-----------------|---------|
| `anchorUtc`   | string (ISO)    | UTC instant of an **observed** first door-open. The whole 185-min cycle is projected from this. |
| `anchorBuild` | string          | The Star Citizen build the observation was made on (trailing digits of the version, e.g. `12302499`). |
| `verifiedUtc` | string \| null  | When a human last confirmed the timer against the real in-game lights. `null` = unverified. |
| `note`        | string          | Free-text; ignored by the app. |

### The safety guard (why a wrong anchor can't show wrong times)

The app compares `anchorBuild` against the live build. If they **don't match**, it shows
**"Timer pending re-sync"** and hides the schedule rather than displaying stale times.
So the failure mode is "no timer", never "wrong timer". The committed placeholder here
uses an old `anchorBuild` on purpose, so the page stays in the safe pending state until
someone enters a real, current-build observation.

## How to re-sync (the whole job)

1. In-game, watch the Pyro Executive Hangar and note the UTC instant the door **opens**.
2. Edit `exec_hangars_anchor.json`:
   - `anchorUtc` → that instant, ISO-8601 UTC (e.g. `2026-07-24T14:05:00.000Z`)
   - `anchorBuild` → the current live build number (trailing digits of the site version)
   - `verifiedUtc` → the instant you confirmed it
3. Commit + push to `main`.

Propagation: the GitHub CDN and the app's own 5-minute cache bucket mean a change is
visible to **new page loads** within ~5 minutes. Already-open tabs pick it up on reload
(the app fetches the anchor once per page load, not on a timer).
