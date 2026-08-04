---
name: fetch-gsc
description: Fetch Google Search Console data for the current project using the seo-strategy skill's GSC integration scripts. Use when you need to refresh GSC data before generating an SEO strategy report.
---

# Fetch GSC Data

Fetch Google Search Console performance data and save it locally for use with `/seo-strategy`.

## Usage

```
/fetch-gsc                    Fetch performance data (queries, pages, positions)
/fetch-gsc --submit-sitemap   Resubmit the sitemap so Google recrawls the URL listing
```

## Check Invocation Mode

If invoked with `--submit-sitemap` or `submit-sitemap`, skip to the **Submit Sitemap Command** section below. Otherwise, continue with the data-fetch steps immediately following.

## Steps

1. **Locate Skill Configuration:**
   - The skill will automatically use its installation location
   - Possible locations:
     - Personal: `~/.claude/skills/seo-strategy/config.json`
     - Project-specific: `.seo-content/config.json`
   - If config.json not found, inform the user:
     ```
     ❌ Error: config.json not found in seo-strategy skill directory

     Please create config.json from config.template.json.

     For personal installation:
     cd ~/.claude/skills/seo-strategy
     cp config.template.json config.json

     For project-specific installation:
     cd .claude/skills/seo-strategy
     cp config.template.json config.json

     Then edit config.json and set:
     - "gscIntegration.enabled": true
     - "gscIntegration.property": "https://yoursite.com"
     ```
   - Exit if no config found

2. **Verify GSC Integration Enabled:**
   - Read skill's `config.json` and check `gscIntegration.enabled`
   - If `false` or missing, inform the user:
     ```
     ℹ️  GSC integration is disabled in config.json

     To enable GSC data fetching:
     cd ~/.claude/skills/seo-strategy
     Edit config.json and set:
     - "gscIntegration.enabled": true
     - "gscIntegration.property": "https://yoursite.com"

     Then run /fetch-gsc again
     ```
   - Exit if disabled

3. **Verify Fetch Script:**
   - Use the same skill directory location from step 1
   - Check for `scripts/fetch-gsc-data.js` in the skill directory
   - If script not found, inform the user:
     ```
     ❌ Error: GSC fetch script not found

     Expected locations:
     - ~/.claude/skills/seo-strategy/scripts/fetch-gsc-data.js
     - .claude/skills/seo-strategy/scripts/fetch-gsc-data.js

     Please ensure the seo-strategy skill is properly installed.
     ```
   - Exit if script not found

4. **Check Script Dependencies:**
   - Check if `node_modules` exists in the scripts directory
   - If not found, inform the user they need to install dependencies:
     ```
     ⚠️  Script dependencies not installed

     First-time setup required:
     cd ~/.claude/skills/seo-strategy/scripts
     npm install

     Then run /fetch-gsc again.
     ```
   - Exit if dependencies not installed

5. **Display Pre-Fetch Information:**
   - Show the property that will be fetched:
     ```
     🔍 Google Search Console Data Fetch

     Property: [property URL from config]
     Data will be saved to: [dataDir from config]
     Date range: [dateRangeDays from config] days

     Starting fetch...
     ```

6. **Execute Fetch Script:**
   - Use Bash tool to run the fetch script:
     ```bash
     cd [skill-directory]/scripts && node fetch-gsc-data.js
     ```
   - Use a reasonable timeout (5 minutes should be sufficient)
   - Display the script's output in real-time

7. **Handle Results:**
   - If script succeeds (exit code 0):
     - Display success message:
       ```
       ✅ GSC data fetch completed successfully!

       Data saved to: [dataDir]
       History snapshot: [dataDir]/history/[window end date].json

       You can now run /seo-strategy to generate a GSC-enhanced strategy report.
       ```
   - The four files in `dataDir` are overwritten every run. The snapshot under `history/` is not, and it is the only thing that accumulates a time series, so never suggest clearing that directory to save space. It is what `/seo-score-validate` reads to check whether content changes moved rankings, and a deleted snapshot cannot be re-fetched: Search Console will not serve a window that has already rolled out of its retention.

   - If script fails (non-zero exit code):
     - Display the error output
     - Provide troubleshooting guidance:
       ```
       ❌ GSC data fetch failed

       Common issues:
       1. OAuth credentials not set up
          → Create credentials.json in ~/.gsc-credentials/
          → See README.md for setup instructions

       2. Property not accessible
          → Verify property URL in skill's config.json
          → Ensure you have access in Google Search Console

       3. API rate limiting
          → Wait a few minutes and try again

       For detailed setup instructions, see:
       ~/.claude/skills/seo-strategy/scripts/README.md
       ```

8. **Display Next Steps:**
   - After successful fetch, suggest next actions:
     ```
     💡 Next steps:

     1. Generate SEO strategy with GSC data:
        /seo-strategy

     2. View Quick Wins from latest strategy:
        /seo-strategy --quick-wins

     3. Re-fetch GSC data (recommended weekly):
        /fetch-gsc

     4. Check whether scored content actually moved:
        /seo-score-validate
     ```
   - Weekly re-fetching matters more than it used to: each run adds a point to the history series, and the series is what makes ranking movement answerable later.

Do NOT generate a strategy report. This command only fetches GSC data; strategy generation is handled by `/seo-strategy`.

---

## Submit Sitemap Command

If invoked with `--submit-sitemap` or `submit-sitemap`:

**What this does, and what it does not do:** Google's Indexing API is restricted by Google's own terms to `JobPosting` and `BroadcastEvent` structured data, so it is not a legitimate way to force-index a regular article page. Resubmitting the sitemap via `sitemaps.submit` is the actual sanctioned programmatic step: it tells Google the URL listing changed and should be recrawled, but it does not guarantee immediate indexing of any single URL. If a specific URL needs to be indexed sooner, that still requires the manual "Request Indexing" button in GSC's URL Inspection tool (no public API exists for that, and it's rate-limited to roughly 10-12 URLs/day).

1. **Locate Skill Configuration and Script:**
   - Same config resolution as the data-fetch flow above: `.seo-content/config.json` (project) first, then `~/.claude/skills/seo-strategy/config.json` (personal)
   - Check for `scripts/submit-sitemap.js` in the seo-strategy skill directory
   - If script not found, inform the user:
     ```
     ❌ Error: sitemap submission script not found

     Expected at: ~/.claude/skills/seo-strategy/scripts/submit-sitemap.js

     Please ensure the seo-strategy skill is up to date.
     ```
   - Exit if script not found

2. **Verify `sitemapUrl` is configured:**
   - Read `gscIntegration.sitemapUrl` from the resolved config
   - If missing, inform the user:
     ```
     ❌ Error: no sitemapUrl configured

     Add it to .seo-content/config.json:
       "gscIntegration": { ..., "sitemapUrl": "https://yoursite.com/sitemap.xml" }

     Domain properties (sc-domain:...) can't derive this automatically
     since they don't pin a scheme or www/non-www -- it must be set explicitly.
     ```
   - Exit if missing (the script also checks this itself, but surfacing it here avoids an unnecessary OAuth prompt)

3. **Check Script Dependencies:**
   - Same as step 4 of the data-fetch flow: check for `node_modules` in the scripts directory, prompt `npm install` in `~/.claude/skills/seo-strategy/scripts` if missing

4. **Execute Submit Script:**
   - Use Bash tool to run:
     ```bash
     cd [skill-directory]/scripts && node submit-sitemap.js
     ```
   - **This may require one-time interactive OAuth consent** (the script requests a different, read-write scope than the data-fetch script, so it keeps its own token file the first time it runs for a property). If the script prints an authorization URL, surface it to the user and let the run continue in the background rather than blocking synchronously, since approving it happens in their browser, not the terminal.
   - Use a reasonable timeout; if waiting on user consent, prefer running the command in the background and checking back rather than a short blocking timeout

5. **Handle Results:**
   - Success: display the script's own confirmation output (submission timestamp, warnings/errors count from `sitemaps.get`)
   - Failure with `insufficient permission`: this means the authenticated Google account only has Restricted (read-only) access to the property in Search Console, not Full or Owner -- a GSC permission-tier issue, not an OAuth scope problem. The script's own error output includes the Search Console users-and-permissions URL and the exact commands to delete the stale token and retry with a different account. Relay that guidance rather than re-deriving it.
   - Other failures: relay the script's error output directly

Do NOT run the data-fetch flow when in this mode, and do NOT generate a strategy report.
