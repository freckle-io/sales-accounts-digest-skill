# Account Digest For Reps

Portable skill package for building Slack-ready account intelligence digests from CRM lists, CRM searches, Salesforce reports, or CSV account lists.

The skill is intentionally connector-first and repo-agnostic. It does not depend on Freckle-specific code, local CLIs, environment variables, or a particular CRM setup.

## Contents

- `SKILL.md` - the full skill definition and operating contract.
- `agents/openai.yaml` - optional UI metadata for skill lists and default invocation.

## What It Does

The skill guides an agent through:

1. Resolving an account list from HubSpot, Salesforce, another CRM connector, or CSV.
2. Discovering available signal providers before extraction.
3. Normalizing all signals into a per-run `signals.csv` staging layer.
4. Applying an actionability gate that suppresses rep-authored CRM noise.
5. Rendering a polished Slack `mrkdwn` digest with per-account signal, source, and suggested action blocks.
6. Holding Slack delivery and CRM note writeback behind explicit user approval.

## Requirements

At least one account source is required:

- HubSpot list/search connector
- Salesforce report/list-view connector
- another CRM connector with account identity, owner, domain, and record URL fields
- CSV with account name plus domain or CRM record ID/URL

Useful signal providers include:

- Sumble, Exa, PredictLeads, Trigify, Jungler, TAMRadar, CrustData, AI Ark, or SignalBase for public triggers
- HubSpot Marketing Hub, product analytics, or internal warehouse data for buyer-authored product/form/visit signals
- Apollo, ZoomInfo, Vector, RB2B, G2, or RevOps-owned CSV/sheet exports for supplemental account signals

The skill still works with only CRM data, but it will usually produce an empty digest if the CRM only contains rep-authored activity.

## Install

Copy this folder into a supported skills directory.

Project-level install:

```text
.claude/skills/account-digest-for-reps/
```

User-level install:

```text
~/.claude/skills/account-digest-for-reps/
```

Codex/Codex-style environments can install the same folder under their configured skills directory. Keep `SKILL.md` at the package root and preserve `agents/openai.yaml` if your environment supports skill UI metadata.

## Usage

Invoke the skill explicitly:

```text
Use $account-digest-for-reps to build a Slack-ready account digest from this HubSpot list.
```

Or provide a CSV/account source and ask for an account digest. The skill will guide the agent through connector discovery, signal staging, rendering, and delivery approvals.

## Output Contract

Signals are staged in a CSV with this header:

```text
account_name,domain,crm,crm_record_id,crm_url,owner,signal_group,summary,source_title,source_url,date,confidence,suggested_action,buyer_authored,provider
```

The Slack digest must be Slack-native `mrkdwn`, emoji-led, and rendered per account. Each surfaced account must include:

- the account name
- a CRM record link when available
- the signal
- the source and date
- a concrete suggested action

## Safety And Approval Gates

The skill requires explicit approval before:

- extracting from optional signal providers not already approved by the user
- sending a Slack message
- writing CRM notes
- enriching missing account domains externally

It also prevents misleading attribution: a connector is only listed as used if that connector produced rows in the run artifact.

## Packaging Notes

- This repository should contain only the standalone skill package.
- Do not include repo-local symlinks such as `.claude/skills/account-digest-for-reps`.
- Do not include Freckle-specific adapters or local CLI wrappers unless they are moved into optional references and clearly labeled as examples.
