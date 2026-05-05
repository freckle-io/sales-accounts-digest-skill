# Account Digest For Reps

Build Slack-ready account digests for reps or managers.

The skill helps answer: which accounts deserve attention now, why, and what should the rep do next?

## How It Works

1. Ask who the digest is for.
2. Ask which accounts to track.
3. Ask where the digest should go in Slack.
4. Discover available CRM, enrichment, product/activity, collaboration, and file/sheet sources.
5. Ask which signal mode fits the rep workflow.
6. Ask the user to approve or add enrichment providers before signal analysis.
7. Stage approved signals in `signals.csv`.
8. Render a Slack preview.
9. Post or write back to CRM only after explicit approval.

## Signal Modes

- Pipeline follow-up
- Product-led signups/trials
- Named target accounts
- Expansion/customer health
- Outbound trigger hunting

The skill should recommend useful sources for the selected mode instead of defaulting to CRM-only data.

## Enrichment Gate

Before building a digest, the skill must ask about enrichment providers and other prospect-update surfaces, including:

- Sumble, Exa, Clay, Apollo, ZoomInfo, PredictLeads, RB2B/Vector, G2/Capterra
- Slack channels or threads
- Notion notes or docs
- Clerk/session activity
- Product analytics or warehouse exports
- HubSpot custom events/objects
- RevOps CSVs or sheets

If the user declines or those sources are unavailable, the digest can run with CRM/first-party data, but it should say the insights may be thinner.

## Output Contract

Every surfaced signal must be staged in `signals.csv` before it appears in the Slack preview. A provider is only listed as used if it produced rows.

Slack posting and CRM writeback require separate approval.
