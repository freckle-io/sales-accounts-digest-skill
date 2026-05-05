---
name: account-digest-for-reps
description: Build connector-first account signal digests for reps or managers from CRM lists, Salesforce reports/list views, CRM searches, HubSpot custom events/objects, product/session activity, Slack/Notion/prospect updates, enrichment providers, or CSVs; rank the best accounts; render Slack-ready summaries; and gate enrichment, Slack delivery, and CRM writeback behind explicit approval.
---

# Account Digest For Reps

Build a Slack-ready account digest that helps reps decide which accounts deserve attention now. Keep the skill portable: do not assume Freckle-specific paths, local CLIs, local env vars, or a fixed CRM.

## Core Contract

- Start with three intake questions: who the digest is for, what accounts to track, and where it should go in Slack.
- Discover CRM, enrichment, product/activity, collaboration, file/sheet, and delivery connectors before signal analysis.
- Ask the user to add, connect, export, or approve enrichment/provider sources before running any signal analysis.
- Explicitly ask about prospect updates in Slack, Notion, Clerk/session activity, product analytics, warehouse exports, provider lists, and any other accessible surface the user wants queried.
- Do not stage `signals.csv`, render a preview, or run CRM-only analysis until the user has selected a signal mode and approved, declined, or ruled out enrichment/provider inputs.
- Stage every surfaced signal in a per-run `signals.csv` before rendering. Never render signals that are not in the CSV.
- Cap Slack output at the top 10 accounts per rep/segment unless the user asks for more.
- Use `🟢 *Why now:*` for the primary surfaced trigger. Do not use red alert/red-circle markers.
- Slack delivery, CRM note writeback, external domain enrichment, optional provider extraction, and querying collaboration surfaces require explicit approval.
- Never claim a provider was used unless it produced signal rows.

## Reference Router

Load only what the run needs:

- Provider discovery, signal modes, enrichment gate: `references/providers.md`
- Signal groups, scoring, suppression rules: `references/signal-groups.md`
- CSV staging schema: `references/extraction.md`
- HubSpot CRM/events/custom objects: `references/hubspot.md`
- Slack formatting and example output: `references/slack-output.md`

Templates:

- `templates/slack_digest.md`
- `templates/slack_account_block.md`
- `templates/slack_digest_zero_surface.md`

## Workflow

1. **Intake.** Ask who the digest is for, what accounts to track, and where it should go in Slack. If the user already provided any answer, ask only for what is missing.
2. **Discover sources.** Inspect available connectors/tools for CRM, enrichment providers, Slack, Notion/docs, Clerk/session activity, product analytics, sheets/CSVs, warehouse exports, web/search, and Slack delivery. Read `references/providers.md`.
3. **Resolve account source.** Accept HubSpot list/search, Salesforce report/list view, CRM search, CSV/sheet, territory/segment, or another source. Require account name plus domain or CRM record URL/id. Infer owner fields before asking.
4. **Gate enrichment before analysis.** Summarize detected and missing source categories. Ask the user what reps are managing, recommend a signal mode, and ask which enrichment providers, prospect-update surfaces, exports, or connectors to include. Do not create `signals.csv` or a preview yet.
5. **Get approval or decline.** Proceed only after the user explicitly approves provider/surface queries, declines them, or confirms they are unavailable. If they decline, label the run CRM-only or first-party-only and explain that insights may be thinner.
6. **Extract signals.** Query approved sources only. Normalize every usable signal into `signals.csv` using `references/extraction.md`.
7. **Score and render.** Rank by the selected mode, recency, ICP/account fit, confidence, and actionability. Render only from `signals.csv` using the Slack templates.
8. **Deliver after approval.** Confirm exact Slack destination and post only after approval. CRM writeback requires separate approval and account confirmation.

## Required User Prompts

Opening prompt:

```text
To build the account digest, I need three basics first:

1. Who is this for: you, a specific rep, selected reps, team members, or all owners?
2. What accounts should I track for them: named accounts, a CRM list/report/search, territory, segment, or CSV/sheet?
3. Where should the digest go in Slack: channel, DM, thread, or preview-only?
```

Pre-analysis enrichment prompt:

```text
Before I run signal analysis, I need to decide which signal sources to include.

What are these reps managing: pipeline follow-up, product-led signups/trials, named target accounts, expansion/customer health, or outbound trigger hunting?

I found these usable sources: ...

Do you want me to include enrichment providers or prospect-update surfaces before building the digest? Examples: Sumble, Exa, Clay, Apollo, ZoomInfo, PredictLeads, RB2B/Vector, G2/Capterra, Slack threads/channels, Notion notes/docs, Clerk/session activity, product analytics, HubSpot custom events/objects, warehouse exports, or RevOps CSVs/sheets.
```

If the selected mode needs a source that is missing, ask whether the user can connect or export it before proceeding.

## Output Rules

- Top 10 accounts by default; do not pad weak accounts.
- Every Slack account block needs account name, CRM link, why now, sales hypothesis, who to try, suggested opener, source/provider, and date.
- Source labels must include provider attribution, e.g. `(found with Sumble)`.
- Use short dates like `Apr 27` in Slack; keep ISO dates in CSV.
- Report blocked, missing-domain, and owner-review records outside the Slack digest unless the user asks to include them.

## Optional Repo Adapter

If the current repo has a documented implementation, use it only when the user intends to run that repo-specific path and outputs still conform to this skill. In the Freckle GTM repo, historical commands live behind `python3 -m outbound_ops.cli account-digest-*`; treat them as a local adapter, not the portable default.
