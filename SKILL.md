---
name: account-digest-for-reps
description: Build connector-first account intelligence digests for reps from CRM lists, CRM searches, Salesforce reports/list views, or CSV account lists; normalize signals from approved connectors; render compact Slack-ready summaries; and gate Slack delivery plus CRM note writeback behind explicit approval.
---

# Account Digest For Reps

Use this skill to run a repo-agnostic account intelligence digest workflow. The skill defines the operating contract. It must not assume a local repo, local CLI, local environment variables, or Freckle-specific paths.

## Default Posture

- Connector-first and repo-agnostic.
- RevOps/manager-run by default.
- Draft/preview before live delivery.
- Slack delivery only after confirming the exact destination.
- CRM note writeback only after Slack QA and explicit approval.
- No automated tasks in v1 unless the user explicitly asks for them.
- Do not imply a connector was used unless that connector produced the signal.

## Start Here

Before extracting data, ask or infer the account source:

```text
How will you provide the account list?
- HubSpot list
- Salesforce report or list view
- CSV
- connected CRM search
```

If the user chooses CRM, check whether the relevant CRM connector/MCP is available in the current environment:

- HubSpot connector for HubSpot lists, companies, contacts, and notes.
- Salesforce connector for Salesforce reports, list views, accounts, contacts, and notes/tasks.
- Other CRM connector only if it exposes enough account identity, owner, domain, and record URL data.

If no CRM connector is available, ask the user to connect it or provide a CSV fallback. Do not fall back to a local repo CLI unless the user explicitly asks for a repo-specific adapter.

## Data Model

Resolve every account into this portable account shape:

```json
{
  "name": "Example Co",
  "domain": "example.com",
  "owner": "Rep Name or owner id",
  "crm": "hubspot",
  "crm_record_id": "123",
  "crm_url": "https://app.hubspot.com/contacts/PORTAL/record/0-2/123"
}
```

Normalize every signal into this portable signal shape:

```json
{
  "group": "product_signups",
  "summary": "Organization signed up on 2026-04-24.",
  "source_title": "HubSpot",
  "source_url": "https://app.hubspot.com/contacts/PORTAL/record/0-2/123",
  "date": "2026-04-24",
  "confidence": "high"
}
```

The digest payload should use:

```json
{
  "rep_name": "Rep or segment name",
  "accounts": [
    {
      "account": {
        "name": "Example Co",
        "domain": "example.com",
        "owner": "Rep Name",
        "crm": "hubspot",
        "crm_record_id": "123",
        "crm_url": "https://app.hubspot.com/contacts/PORTAL/record/0-2/123"
      },
      "signals": []
    }
  ],
  "blocked_accounts": []
}
```

## Account Source Adapters

Use the available connector or input file as the data layer:

- `HubSpot list`: use HubSpot connector/MCP to list members and hydrate company/account records.
- `Salesforce report/list view`: use Salesforce connector/MCP to read rows and hydrate account records.
- `CSV`: parse the file and require at least company name plus either domain or CRM record URL/id.
- `connected CRM search`: use the CRM connector's search capabilities and document the filter criteria.

For every account, require:

- company/account name
- domain or explicit `needs_domain_enrichment`
- owner or explicit `needs_owner_review`
- CRM record URL when the account came from CRM

Do not research accounts missing domains externally unless the user explicitly approves domain enrichment.

## Discover Signal Providers (Hard Gate — Required Before Extraction)

Before extracting any signals, you MUST self-discover available providers and surface the result to the user. Do not call any signal-source tool until the user has confirmed the provider list, unless the user has already named the provider(s) to use in the current request.

Step 1 — Self-discover existing connectors/MCPs (do not ask the user yet):

- Enumerate the connectors/MCPs available in the current environment by inspecting the tool registry. Do not ask the user which connectors exist; check yourself.
- Common GTM data sources to scan for: Apollo, ZoomInfo, Vector, RB2B, Clay, Cognism, BuiltWith, G2, product-usage feed (Mixpanel, Amplitude, internal warehouse), Marketing Hub form-fill / identified-visit feed.
- Common signal-provider connectors to scan for: Exa, PredictLeads, Sumble, Trigify, Jungler, TAMRadar, CrustData, AI Ark, SignalBase.
- Common companion connectors: Slack (for delivery), Gmail/Outlook (for reply signals), Notion / Google Sheets / CSV (for housing).
- For each found provider, classify whether it produces *buyer-authored* signals (form fills, product events, replies, identified web visits, public triggers) or only *rep-authored* CRM activity, and which `signal_group` values it can populate.

Step 2 — Present findings:

Render a short table to the user showing every detected provider, its classification, and a one-line "what it would contribute" note. Be explicit about anything you expected to find but did not (e.g. "Sumble: not connected").

Step 3 — Ask about RevOps data sources the agent cannot detect:

After the CRM connector check and provider discovery, explicitly ask the user what *additional* GTM data sources they have available that are not exposed as MCPs in this environment. Keep the first prompt focused on common RevOps-owned sources:

- "ZoomInfo or Apollo intent feeds you can export to CSV?"
- "Vector or RB2B visitor/account-identification exports?"
- "Glean or internal-search exports for account context?"
- "Slack channels or threads where reps log inbound interest?"
- "Gong/Chorus call mentions or competitor-mention exports?"
- "A G2 or Capterra buyer-intent CSV?"
- "An internal product-usage CSV from the data team?"
- "Any other CSV/sheet RevOps maintains for hand-raisers, champions, or accounts under review?"

Then ask which additional signal-provider connectors they can expose or want checked next, using concrete examples:

- "Exa for news, launches, funding, and public web triggers?"
- "PredictLeads for hiring, funding, tech, and company events?"
- "Sumble for hiring and organization enrichment?"
- "Trigify, Jungler, TAMRadar, CrustData, AI Ark, or SignalBase for account trigger feeds?"

Wait for the user's response before proceeding unless they already gave explicit provider instructions in the current request. Capture additional sources alongside the detected MCPs in the run's provider list.

Step 4 — Confirm and proceed:

Confirm in writing the final provider list for the run, then begin extraction. If the only available provider yields rep-authored CRM activity, explicitly warn the user the digest will likely be empty under the actionability gate.

A digest sourced from a single CRM connector is the most common low-utility failure mode. Skipping this gate is a defect, not a shortcut.

## Signal Housing (Local CSV)

All signals from all providers must land in a single per-run CSV before the digest is rendered. This is the canonical staging layer — the renderer reads only the CSV, never directly from connectors. This makes runs auditable, replayable, and enrichable across multiple providers.

Path convention (portable):

```text
outputs/account-digests/<list-slug>/run-<YYYY-MM-DD>/signals.csv
```

Columns (header row required):

```text
account_name,domain,crm,crm_record_id,crm_url,owner,signal_group,summary,source_title,source_url,date,confidence,suggested_action,buyer_authored,provider
```

Field rules:

- `signal_group`: one of `product_signups`, `product_usage`, `web_intent`, `form_fill`, `email_reply`, `crm_activity`, `lifecycle`, `hiring`, `funding`, `launch`, `news`, `tech_change`, `other`.
- `date`: ISO `YYYY-MM-DD` of when the signal occurred (not when it was fetched).
- `confidence`: `low` | `medium` | `high`.
- `buyer_authored`: `true` if the row reflects prospect/customer action or an external market trigger attributable to the account, such as public hiring, funding, launches, replies, product events, form fills, or identified visits; `false` if rep-authored CRM activity or static lifecycle.
- `provider`: connector name (`hubspot`, `salesforce`, `exa`, `sumble`, `crustdata`, `clay`, `csv`, `product`, etc.).
- One row per signal. Re-run dedup on `(account, signal_group, summary, source_url, date)`.

The renderer applies the actionability gate to this CSV: rows with `buyer_authored=false` and no companion buyer-authored or external-trigger row for the same account inside the signal window are suppressed.

A run with zero `buyer_authored=true` rows must produce a digest that says so explicitly, not pad with rep-authored rows. Public trigger providers such as Sumble may set `buyer_authored=true` when the event is dated, account-specific, and not authored by the rep.

## Signal Source Adapters

Collect signals only from installed, approved connectors:

- `CRM/internal`: product usage, signups, website visits, lifecycle stage, open opportunities, recent activity, owner context.
- `Exa`: public research, recent news, hiring pages, funding announcements, website context.
- `Sumble`: hiring signals and organization enrichment when configured and approved.
- `PredictLeads`: hiring, funding, tech, and company event signals when configured and approved.
- `Trigify`, `Jungler`, `TAMRadar`, `CrustData`, `AI Ark`, `SignalBase`, or other trigger providers: only if connected and explicitly approved for the run.
- `Apollo`, `ZoomInfo`, `Vector`, `RB2B`, `Clay`, `G2`, or other GTM data sources: use exported CSV/sheet data or connectors only after the user confirms the data source and intended signal groups.
- `CSV`: user-supplied signal columns or supplemental research files.

Connector readiness is not the same as usage. A connector is used only when the run artifact includes signals sourced from that connector.

## Normalization Rules

- Deduplicate signals by account, group, summary, source URL, and date.
- Prefer dated signals inside the user-approved signal window, defaulting to 30 days.
- Keep older evergreen signals only if the user explicitly asks for static account context.
- Preserve source URLs and titles.
- Include confidence when the connector or source quality supports it.
- Include `suggested_action` when a signal is rendered to Slack.
- Keep `why_it_matters` optional in structured payloads and note previews, not in the default Slack digest.

## Actionability Gate

Do not include a signal merely because a connector or CRM field returned data. A signal is digest-worthy only when it:

- gives the rep a timely reason to review or contact the account,
- can be explained in one concise sentence,
- has a clear source title and URL where possible,
- supports a concrete suggested action, and
- is not just generic firmographic/static context.

Dated signals inside the approved signal window pass the timing bar. Undated signals should pass only when they represent durable product/commercial context, such as active usage or plan data.

Weak examples to suppress by default:

- old funding stage with no recent trigger,
- one anonymous website session with no recent date or contact context,
- raw employee count, industry, or location,
- generic technology lists without a sales reason,
- rep-authored CRM activity timestamps (e.g. `notes_last_updated`, `hs_lastmodifieddate`) on their own — these reflect the rep's own work, not buyer intent. Surface them only when paired with a buyer-authored event (form fill, product usage, reply, web visit by a known contact) or an external trigger.

Distinguish buyer-authored signals (form fills, product events, replies, identified web visits, public triggers from Exa/Sumble) from rep-authored CRM activity (logged notes, calls, tasks). A digest of rep-authored activity tells the rep what they already did; the goal is to tell them what the buyer or market just did.

## Slack Digest Format

The digest must be Slack-native, visually scannable, and pleasant to read. Use concise emoji section markers, Slack `mrkdwn`, and stable per-account blocks. The digest is **always rendered per account** when there are surfaced signals. One block per account that has at least one surfaced signal. Do not collapse, summarize across accounts, or substitute a top-level prose summary for the per-account blocks. If multiple signals exist for one account, list each as its own signal/source/action set inside that account's block.

Formatting/copy model guidance:

- Use a cheaper capable model for Slack formatting, ordering, and wording polish when one is available.
- Do not use the formatting model to decide facts, add signals, infer dates, or change source attribution.
- The formatting pass may only rewrite already-normalized rows from `signals.csv` plus run metadata.
- If a signal is not in `signals.csv`, it must not appear in Slack.

Header shape:

```text
📣 *Account Digest — Rep or Segment Name*
🗂️ *Source:* HubSpot list 1024 · *Window:* last 14 days
🔌 *Connectors used:* HubSpot, Sumble
✅ *Surfaced accounts:* 4 · 🧹 *Suppressed:* 43



`-----`


```

For each surfaced account, use exactly this shape:

```text
🏢 *Account:* Company name
🔗 <CRM_RECORD_URL|CRM record>

🚨 *Signal:* concise summary
📍 *Source:* <SOURCE_URL|Source title> · *Date:* Apr 24
➡️ *Suggested action:* concrete next step



`-----`


```

For multiple signals on the same account, repeat the `Signal`, `Source`, and `Suggested action` lines for each signal under the same company header. Each surfaced signal must include all three fields. Use a blank line between signals for readability.

If the run has zero surfaced accounts, post a single short status block plus the "what to connect next" section. Do not invent placeholder per-account blocks. Use this shape:

```text
📣 *Account Digest — Rep or Segment Name*
🗂️ *Source:* HubSpot list 1024 · *Window:* last 14 days
🔌 *Connectors used:* HubSpot only



`-----`


🚫 *No buyer-authored signals to surface*
Under the strict actionability gate, this run produced 0 surfaced accounts.

🔎 *What we checked:* concise source/provider summary.
🧭 *What to connect next:* concise list of missing signal sources.
```

Blocked records, missing domains, and `RevOps review needed` items do **not** belong in the Slack digest by default. Share those back in the agent/user channel after rendering or posting the digest, unless the user explicitly asks to include them in the Slack message.

Rules:

- Strip query strings and UTM parameters from CRM record links.
- Link sources with Slack syntax: `<url|source title>`.
- Use Slack's native `mrkdwn` convention: single asterisks for bold (`*text*`). Bold labels exactly as `*Account:*`, `*Signal:*`, `*Source:*`, `*Date:*`, and `*Suggested action:*`. Do not use standard Markdown double-asterisk bold (`**text**`) in Slack output.
- Render divider lines as inline code (`` `-----` ``) because bare hyphen-only lines can be interpreted as unsupported horizontal-rule blocks.
- Put two blank lines before and after each divider line so Slack has visual breathing room between account blocks.
- Use short dates such as `Apr 24`.
- Include `suggested_action` below each signal.
- Omit `why_it_matters` from Slack by default.
- Omit blocked accounts and `RevOps review needed` sections from Slack by default; report them separately in the agent/user channel.
- Emojis are part of the Slack digest format. Use them sparingly as section markers, not decoration on every word.

## Delivery Gates

Before Slack delivery:

1. Confirm the exact Slack destination: channel, DM, or group DM.
2. Confirm the connector exposes a Slack send/draft tool in the current environment.
3. If no Slack connector is available, ask for a webhook or stop at a post-ready Markdown artifact.
4. Do not post to each rep's Slack until a test destination has passed QA.

Before CRM note writeback:

1. Slack QA must pass.
2. User must approve note writeback.
3. User must approve the exact accounts receiving notes.
4. Use the CRM connector/MCP for writeback; do not assume a local API client.

## Repo-Specific Adapters

This skill should be portable. Local CLIs are optional adapters, not the default workflow.

If a repo provides its own implementation, you may use it only when:

- the user is intentionally running inside that repo,
- the CLI command is documented,
- the CLI's data sources are disclosed, and
- the outputs still conform to the portable schema above.

For the Freckle GTM repo specifically, the historical adapter is:

```bash
python3 -m outbound_ops.cli account-digest-check-connectors
python3 -m outbound_ops.cli account-digest-extract-hubspot-list
python3 -m outbound_ops.cli account-digest-render
python3 -m outbound_ops.cli account-digest-preview-notes
python3 -m outbound_ops.cli account-digest-write-notes
```

That adapter uses Freckle's repo-local Python code and HubSpot API client. It is not the portable default and should be described as a Freckle-only implementation path when used.
