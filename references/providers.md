# Provider Discovery, Signal Modes, And Enrichment Gate

Use this reference before any signal extraction. The goal is to prevent thin CRM-only digests unless the user deliberately chooses that path.

## Discovery Order

1. Inspect available tools/connectors.
2. Validate CRM object and list capabilities before declaring account sources unavailable.
3. Classify detected sources by category.
4. Ask what reps are managing and recommend a signal mode.
5. Ask the user to approve, add, connect, or export enrichment/provider sources for that mode.
6. Run signal extraction only after the user approves, declines, or rules out those sources.

## Source Categories

| Category | Examples | Use for |
| --- | --- | --- |
| CRM/account source | HubSpot, Salesforce, CRM search, reports/lists | account scope, owners, lifecycle, opportunity context |
| First-party activity | HubSpot custom events/objects, product analytics, Clerk/session, warehouse exports | signups, usage, sessions, activation, form fills, web intent |
| Collaboration/prospect updates | Slack channels/threads, Notion notes/docs, call notes, sheets | rep notes, customer/prospect updates, internal context to validate signals |
| Company event data | Sumble, Exa, PredictLeads, CrustData, SignalBase, AI Ark, TAMRadar | hiring, funding, launches, news, web changes |
| People/contact data | Apollo, ZoomInfo, Clay | stakeholder changes, new contacts, role/persona changes |
| Intent/review data | RB2B, Vector, G2, Capterra, review exports | identified visits, category intent, competitor intent |
| Files/provider workspaces | Clay, Google Sheets, CSV exports | any provider-backed signals with account/domain/date/provider columns |

## Signal Modes

Ask the user to pick or edit one mode. Use the mode to decide which providers matter.

| Mode | Best when reps manage | Default signals | Best sources |
| --- | --- | --- | --- |
| Pipeline follow-up | active prospects, open opps, demos, stalled deals | form fills, replies, meetings, deal movement, pricing/demo visits, stakeholder changes | CRM, HubSpot activity, Slack/Notion updates, web intent, people/contact providers |
| Product-led signups/trials | self-serve orgs, trials, activation, hand-raisers | org signups, workspace creation, activated features, repeat sessions, invited users, pricing visits | HubSpot events/objects, Clerk/session activity, product analytics, warehouse exports |
| Named target accounts | strategic accounts without active pipeline | relevant hiring, funding, launches/news, website changes, stakeholder changes, category intent | company event providers, Exa/search, people/contact providers, review/intent exports |
| Expansion/customer health | customers, renewals, upsell, risk | usage growth/dropoff, feature adoption, team growth, support signals, lifecycle/renewal changes | CRM, support data, product analytics, Slack/Notion updates, warehouse exports |
| Outbound trigger hunting | cold outbound lists or territories | GTM/RevOps hiring, funding, market expansion, launches, partner programs, category/competitor intent | Sumble, Exa, PredictLeads, G2/Capterra, web intent, provider CSVs |

## Required Gate

Before creating `signals.csv` or a Slack preview, ask a version of:

```text
Before I run signal analysis, should I include enrichment providers or prospect-update surfaces?

For this mode, useful sources are: ...
I found: ...
Missing or not connected: ...

Approve querying the available sources, or tell me which exports/connectors to add first.
```

Acceptable outcomes:

- User approves provider/surface queries. Query only approved sources.
- User declines. Continue as CRM-only/first-party-only and state the limitation.
- User says sources are unavailable. Continue only with available approved sources.
- User provides exports/sheets/files. Stage those as approved provider inputs.

Do not infer approval from connector availability. Connected is not approved. Used means the provider produced rows.

## Recommendation Rules

- Pipeline follow-up: prioritize CRM/HubSpot activity, replies/form fills, Slack/Notion updates, web visits, and stakeholder changes before broad news.
- Product-led: prioritize product/session/custom-event sources before public web enrichment.
- Named targets and outbound: prioritize Sumble/Exa/company-event providers, people/contact changes, and intent/review exports.
- Expansion/customer health: prioritize product usage, support/account-health data, Slack/Notion updates, lifecycle and renewal data.
- If detected sources are mostly CRM/static enrichment, say the digest will be thin unless the user connects or exports dated signal sources.
- Recommend the smallest provider set that covers the chosen mode.

## Provider To Signal Group Map

| Source | Useful signal groups | Notes |
| --- | --- | --- |
| HubSpot CRM | `crm_activity`, `lifecycle`, `form_fill`, `web_intent`, `product_signups`, `custom_event`, `custom_object` | Separate buyer-authored events from rep-authored notes/tasks. |
| Salesforce | `crm_activity`, `lifecycle`, `opportunity`, `custom_object` | Use reports/list views for account scope and owner mapping. |
| Slack / Notion | `crm_activity`, `custom_event`, `company_context`, `other` | Use only approved channels/docs; treat rep-authored context as supporting evidence unless paired with buyer-authored or external signals. |
| Clerk/session activity | `product_usage`, `product_signups`, `session_activity` | Useful for target-domain user activity, workspace creation, login frequency, and activation. |
| Product analytics / warehouse | `product_usage`, `product_signups`, `session_activity` | Prefer account-level aggregations with dates. |
| Sumble | `hiring`, `company_context` | Best for role/team hiring signals and org enrichment. Avoid broad parent-company hiring. |
| Exa | `news`, `launch`, `funding`, `hiring`, `web_change`, `company_context` | Use for current public web research and source citations. |
| PredictLeads | `hiring`, `funding`, `launch`, `news`, `tech_change`, `web_change` | Good for structured company events. |
| Apollo | `stakeholder_change`, `hiring`, `contact_change`, `enrichment` | Use when exported or connected. |
| ZoomInfo | `stakeholder_change`, `intent`, `enrichment`, `hiring` | Validate dates and source labels. |
| RB2B / Vector | `web_intent`, `identified_visit`, `intent` | Treat anonymous or weak visits carefully; prioritize known account/contact context. |
| G2 / Capterra | `review_intent`, `category_intent`, `competitor_intent` | Buyer-intent CSVs should include account, date, category/page, and source. |
| Clay / CSV / Sheets | any mapped group | Require provider/source attribution per row. |
| CrustData / AI Ark / SignalBase / Trigify / Jungler / TAMRadar | `funding`, `hiring`, `news`, `intent`, `company_context` | Use only if connected/exported and approved. |
