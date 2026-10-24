# GTM Health Agent: Weekly Tag Health Report

Owner: VP Analytics Marketing · Runs: every Monday 8:00 AM (local) · Data source: GTM MCP connector (read-only)

## What it does
1. **Flags tags, variables and containers that are not firing or cannot fire** (see checks below)
2. **Sends a report with a recommended fix** for each issue, ranked by business impact
3. **Explains likely causes** each week and compares them with last week's report to show trends

## A note on "not firing"
The GTM API shows configuration, not live hit data. The agent finds the **configuration reasons a tag won't fire**: paused tags, missing triggers, broken references, unpublished changes, blocked consent. To confirm with real traffic, connect GA4 (Windsor.ai GA4 skill). When it's connected, the agent compares key event counts week over week and flags drops above 30%.

## Checks

### Containers
- Live version older than 90 days while the workspace has unpublished changes, meaning work is stuck
- Workspace has unpublished changes older than 7 days
- Live version published in the last 7 days, which is the most likely cause of a new break
- Container has no tags or no live version
- Container domain list is empty or missing production domains

### Tags
- Paused tags (`paused: true`), especially conversion tags
- No firing trigger, or a trigger ID that doesn't exist anymore
- Blocking trigger that matches the firing trigger, so the tag never fires
- Schedule (`scheduleEndMs`) has expired
- Conversion tags (GA4 purchase/lead, Google Ads, Meta, Floodlight) with an empty or hard-coded ID/label instead of a variable
- GA4 event tags with no measurement ID or Google tag config
- Duplicate tags (same type, ID and trigger), which cause double counting
- Custom HTML tags with `document.write`, or third-party scripts with no consent check
- Tag sequencing that depends on a paused or deleted setup tag

### Triggers
- Trigger used by no tag (orphan)
- Custom Event triggers whose event name has a typo or case mismatch with the dataLayer spec
- Page URL / Click conditions using `equals` where `contains`/regex is needed, or hard-coded staging URLs
- Click/Form triggers with "wait for tags" or "check validation" set wrong

### Variables
- `{{Variable}}` references in tags or triggers that don't exist (broken references)
- Data Layer Variables with no default value on revenue, transaction_id or currency
- Lookup / RegEx tables with no default output
- Unused variables (clutter and risk)
- Built-in variables used but not enabled (Click URL, Page Path, and so on)

### Governance and privacy (VP add-ons)
- **Consent Mode v2**: Ads/Meta/Floodlight tags with no consent settings (`ad_storage`, `ad_user_data`, `ad_personalization`). EU revenue risk.
- **PII leakage**: variables or tags that capture email, phone or `?email=` URL parameters
- **Naming standard**: tags not following `Platform – Type – Detail` (for example `GA4 – Event – purchase`)
- **Change log**: who published what this week, with version notes missing flagged
- **Access**: users with Publish rights who are not on the approved list
- **Server-side**: GA4 tags that should be routed through sGTM (if a server container exists)

## Severity
| Level | Meaning | Example |
|---|---|---|
| P0 Critical | Revenue or conversion data lost now | Purchase tag paused; Ads conversion missing label |
| P1 High | Data wrong or at risk | Duplicate tags; no consent on Ads tags |
| P2 Medium | Hygiene that will cause bugs | Orphan triggers; broken variable reference on a non-conversion tag |
| P3 Low | Housekeeping | Naming, unused variables |

## Report format
1. **Health score** (0–100) per container, with change versus last week
2. **Top 5 actions this week**: owner role, effort (S/M/L), business impact
3. **Issue table**: Severity · Container · Item · Problem · Likely cause · Fix
4. **Weekly reasoning**: patterns and likely root causes (recent publish, dev release changed dataLayer, consent banner update, agency edit)
5. **Trend**: new / fixed / still open since last week
6. **Change log** for the past 7 days
