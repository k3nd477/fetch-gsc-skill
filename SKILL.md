---
name: fetch-gsc
description: Fetch Google Search Console data for the current project using the seo-strategy skill's GSC integration scripts. Use when you need to refresh GSC data before generating an SEO strategy report.
---

# Fetch GSC Data

Fetch Google Search Console performance data and save it locally for use with `/seo-strategy`.

## Usage

```
/fetch-gsc
```

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

       You can now run /seo-strategy to generate a GSC-enhanced strategy report.
       ```

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
     ```

Do NOT generate a strategy report. This command only fetches GSC data; strategy generation is handled by `/seo-strategy`.
