---
name: fetch-gsc
description: Fetch Google Search Console data for the current project using the seo-strategy skill's GSC integration scripts, or resubmit the project's sitemap to Google (--submit-sitemap). Use when you need to refresh GSC data before generating an SEO strategy report, or when asked to resubmit a sitemap or get Google to recrawl the site.
---

# Fetch GSC Data

Fetch Google Search Console performance data and save it locally for use with `/seo-strategy`, or resubmit the sitemap.

## Usage

```
/fetch-gsc                    Fetch performance data (queries, pages, positions)
/fetch-gsc --submit-sitemap   Resubmit the sitemap so Google recrawls the URL listing
```

If invoked with `--submit-sitemap` or `submit-sitemap`, skip to **Submit Sitemap Command** below.

## How the scripts resolve config (both modes)

The scripts live in `~/.claude/skills/seo-strategy/scripts/` and own config resolution. Do not re-implement it; their error messages are authoritative, so relay them rather than paraphrasing. For reference, they check, in order:

1. `.seo-content/config.json` in the current working directory (the project). If this file has a `gscIntegration` block, it decides: `enabled: false` stops the run with an error.
2. `~/.claude/skills/seo-strategy/config.json` (personal fallback), used only when the project has no config or its config has no `gscIntegration` block.

Because both config and output paths resolve against the cwd, **always run the scripts from the project root by absolute path. Never `cd` into the skill directory.** The scripts refuse to run from there.

## Steps

1. **Check dependencies:** if `~/.claude/skills/seo-strategy/scripts/node_modules` is missing, tell the user and stop:
   ```
   ⚠️  Script dependencies not installed

   First-time setup (--prefix keeps you in the project directory):
   npm install --prefix ~/.claude/skills/seo-strategy/scripts

   Then run /fetch-gsc again.
   ```
   If `~/.claude/skills/seo-strategy/scripts/fetch-gsc-data.js` itself is missing, say the seo-strategy skill is not installed and stop.

2. **Run the fetch from the project root, in the background:**
   ```bash
   node ~/.claude/skills/seo-strategy/scripts/fetch-gsc-data.js
   ```
   - Run it with `run_in_background`, not as a blocking call. On the first run for a property (or after a token is revoked), the script opens a browser for Google OAuth consent and waits on a local callback until the user approves. A blocking call would just sit until its timeout.
   - Read its output as it comes in. If it prints an authorization URL, show that URL to the user and tell them to approve it in the browser; the run continues on its own afterwards.
   - The script prints which config it loaded, the property, the date range and the output directory. Don't read the config yourself to show these beforehand.

3. **On success (exit 0):**
   ```
   ✅ GSC data fetch completed successfully!

   Data saved to: [dataDir from script output]
   History snapshot: [dataDir]/history/[window end date].json
   ```
   The four files in `dataDir` are overwritten every run. The snapshot under `history/` is not, and it is the only thing that builds a time series, so never suggest clearing that directory to save space. `/seo-score-validate` reads it to check whether content changes moved rankings, and a deleted snapshot cannot be re-fetched: Search Console will not serve a window that has rolled out of its retention.

   Then suggest next steps:
   ```
   💡 Next steps:

   1. Generate SEO strategy with GSC data:     /seo-strategy
   2. View Quick Wins from latest strategy:    /seo-strategy --quick-wins
   3. Re-fetch GSC data (recommended weekly):  /fetch-gsc
   4. Check whether scored content moved:      /seo-score-validate
   ```
   Weekly re-fetching matters: each run adds a point to the history series, which is what makes ranking movement answerable later.

4. **On failure (non-zero exit):** show the script's error output first; it usually names the exact fix. Then, if it helps:
   ```
   ❌ GSC data fetch failed

   Common issues:
   1. No config, or GSC disabled for this project
      → Create/edit .seo-content/config.json in the project root:
        "gscIntegration": { "enabled": true, "property": "https://yoursite.com" }
        (see ~/.claude/skills/seo-strategy/config.template.json for all options)

   2. OAuth credentials not set up
      → Put credentials.json in ~/.gsc-credentials/
      → See ~/.claude/skills/seo-strategy/scripts/README.md

   3. Property not accessible / signed in with the wrong Google account
      → Check the property URL, then re-authorize:
        node ~/.claude/skills/seo-strategy/scripts/fetch-gsc-data.js --reauth

   4. API rate limiting
      → Wait a few minutes and try again
   ```

Do NOT generate a strategy report. This command only fetches GSC data; `/seo-strategy` generates strategy reports.

---

## Submit Sitemap Command

**What this does and does not do:** Google's terms limit the Indexing API to `JobPosting` and `BroadcastEvent` structured data, so it is not a legitimate way to force-index a regular article. Resubmitting the sitemap via `sitemaps.submit` is the sanctioned programmatic step: it tells Google the URL listing changed and should be recrawled, but does not guarantee immediate indexing of any single URL. To get a specific URL indexed sooner, the user still needs the manual "Request Indexing" button in GSC's URL Inspection tool (no public API exists, and it's rate-limited to roughly 10-12 URLs/day).

1. **Check prerequisites:**
   - Same dependency check as step 1 above.
   - If `~/.claude/skills/seo-strategy/scripts/submit-sitemap.js` is missing, say the seo-strategy skill needs updating and stop.
   - If the project's `.seo-content/config.json` exists and has no `gscIntegration.sitemapUrl`, tell the user before running (this avoids an unnecessary OAuth prompt):
     ```
     ❌ Error: no sitemapUrl configured

     Add it to .seo-content/config.json:
       "gscIntegration": { ..., "sitemapUrl": "https://yoursite.com/sitemap.xml" }

     Domain properties (sc-domain:...) can't derive this automatically,
     since they don't pin a scheme or www/non-www.
     ```
     (The script checks this too, so if there is no project config, just run it.)

2. **Run from the project root, in the background:**
   ```bash
   node ~/.claude/skills/seo-strategy/scripts/submit-sitemap.js
   ```
   This script requests a read-write scope and keeps its own token file, so it needs one-time OAuth consent per property even if the data fetch is already authorized. Handle any authorization URL the same way as in step 2 above.

3. **Handle results:**
   - Success: show the script's confirmation (submission timestamp, warnings/errors count from `sitemaps.get`).
   - `insufficient permission`: the Google account has only Restricted (read-only) access to the property in Search Console, not Full or Owner. This is a GSC permission-tier issue, not an OAuth scope problem. The script's error output includes the users-and-permissions URL and the commands to delete the stale token and retry with another account; relay it.
   - Other failures: relay the script's error output.

Do NOT run the data fetch in this mode, and do NOT generate a strategy report.
