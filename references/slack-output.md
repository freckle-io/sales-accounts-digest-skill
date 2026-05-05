# Slack Output

Use the templates in `../templates/`. Do not hand-roll a different format unless the user asks.

## Formatting Rules

- Use Slack `mrkdwn` single asterisks for bold labels.
- Use `🟢 *Why now:*` for the primary surfaced trigger and any one supporting trigger.
- Use `🎯 *Sales hypothesis:*` to translate the trigger into the likely account pain or buying motion.
- Use `👤 *Who to try:*` to name the most likely persona lane or concrete title pattern.
- Use `➡️ *Suggested opener:*` for a rep-ready sentence or two that can be pasted and lightly edited.
- Render divider lines as inline code: `` `-----` ``.
- Put CRM record links on the account line.
- Link sources with Slack syntax: `<url|source title>`.
- Include provider in source label, e.g. `(found with Sumble)`.
- Include short dates like `Apr 27`.
- Keep each account block concise enough for Slack scanning. Default target: 90-140 words per account.
- Omit `why_it_matters` by default.
- Do not mash multiple unrelated provider facts into one messy signal. Pick one primary trigger, then mention at most one supporting trigger if it changes the rep move.
- Avoid vague actions like "ask about data quality." The opener must name the trigger, the likely operational pain, and a concrete question.

## Template Inputs

- `rep_or_segment_name`
- `source_label`
- `window_label`
- `connectors_used`
- `surfaced_count`
- `suppressed_count`
- rendered `account_blocks`

Each rendered account block should include:

- `account_name`
- `fit_urgency_label`: short triage label such as `High fit, medium urgency`
- `crm_record_url`
- `crm_record_link_label`
- `why_now`: one primary trigger, with one supporting trigger only if useful
- `sales_hypothesis`: the sales-relevant interpretation of the trigger
- `who_to_try`: persona lane or title pattern, not a generic department
- `suggested_opener`: paste-ready outbound opener, preferably one or two sentences
- `source_links_and_dates`: compact Slack links with provider attribution and short dates

## AE Readiness Rules

- Start from the strongest account-specific trigger, not the provider name.
- Turn every trigger into a testable sales hypothesis grounded in the ICP and persona canon.
- Prefer persona lanes from the local canon when available: RevOps, GTM Ops, Sales Ops, Revenue Systems, Marketing Ops, Lifecycle, Demand Gen, funnel analytics, or CRM operations.
- Suggested openers should sound like a rep wrote them after reading the signal, not like an analyst wrote an instruction.
- If the trigger does not support a specific opener, suppress the account instead of padding the digest.
- Use sources as proof, not as the main story.

## Full Example Output

```text
📣 *Account Digest — Aaron Trent*

🗂️ *Source:* CRM

📊 *Accounts tracked:* 50

✅ *Accounts with signals:* 10

🔌 *Providers used:* HubSpot CRM, PredictLeads, Sumble, Apollo


`-----`


🏢 *Northstar Kinetic Labs* — High fit, high urgency (<https://fake.com/hubspot/deals/demo-1001|HubSpot Deal>)

🟢 *Why now:* Sumble found a hiring spike in RevOps and outbound ops, with 4 new roles in 9 days.

🎯 *Sales hypothesis:* They are likely formalizing outbound operations, which usually puts pressure on routing, account enrichment, and CRM data quality.

👤 *Who to try:* RevOps, GTM Ops, Sales Ops, or outbound operations leadership.

➡️ *Suggested opener:* Saw Northstar is hiring several RevOps and outbound ops roles. Teams at that point often find the bottleneck is not more sequences, but clean account context for routing and prioritization. Is that something your team is tightening this quarter?

📍 *Sources:* <https://fake.com/sumble/jobs/demo-1001|RevOps hiring feed (found with Sumble)> · *Date:* Apr 27


`-----`


🏢 *Bramble Forge Systems* — Medium fit, medium urgency (<https://fake.com/hubspot/deals/demo-1002|HubSpot Deal>)

🟢 *Why now:* PredictLeads detected a new customer-logo strip on the pricing page, and Apollo found new operations stakeholders this month.

🎯 *Sales hypothesis:* They may be moving from founder-led growth toward a more repeatable GTM motion, so account research and lifecycle segmentation may be getting more important.

👤 *Who to try:* GTM Ops, Marketing Ops, lifecycle marketing, or the new operations stakeholder.

➡️ *Suggested opener:* Noticed Bramble added customer proof and has new ops leadership in place. When teams shift into a more repeatable GTM motion, messy account context often starts hurting routing and lifecycle segmentation. Worth comparing notes?

📍 *Sources:* <https://fake.com/predictleads/site/demo-1002|Site diff (found with PredictLeads)> · <https://fake.com/apollo/accounts/demo-1002|Stakeholder enrichment (found with Apollo)> · *Date:* Apr 27
```
