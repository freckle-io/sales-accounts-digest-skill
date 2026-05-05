# HubSpot CRM, Custom Events, And Custom Objects

Use this reference when HubSpot is the account source or signal store.

## Connector Capability Check

Before asking the user for a HubSpot export or claiming HubSpot cannot support a source, validate the available HubSpot MCP commands and object capabilities:

- Run the connector's user/capability-details tool when available to inspect readable object types and tool availability.
- Check whether `OBJECT_LIST` is readable. If it is, inspect its metadata for list membership behavior before looking for a separate list-membership command.
- For HubSpot list membership, try filtering the relevant CRM object type with `ilsListIds` using `EQ` or `IN` against the HubSpot list/segment id.
- Search properties on `companies`, `contacts`, `deals`, and available activity objects for signup, form, visit, engagement, activity, lifecycle, opportunity, custom event, and custom object fields.
- Treat missing MCP commands differently from missing HubSpot capability: if the connector exposes object search and property metadata, use those surfaces before asking for a CSV fallback.

## Early Question

After detecting HubSpot, ask:

```text
Are relevant account signals stored in HubSpot properties, notes/tasks, custom events, custom objects, form submissions, web activity, or lists? If yes, which objects/events should I inspect?
```

## Account Source Fields

Common fields:

- company/account name
- domain
- company id or deal id
- company owner / deal owner / `hubspot_owner_id`
- lifecycle stage
- associated deal stage
- record URL

Do not assume company and deal are the same object. If the source is a deal list, preserve the deal link while also resolving the associated company when possible.

## Custom Events

Use HubSpot custom events for buyer-authored or product/account activity such as:

- signup
- workspace created
- invited user
- logged in
- activated feature
- trial started
- pricing viewed
- form submitted
- account identified

Normalize event rows with `signal_group=custom_event`, `product_signups`, `product_usage`, `session_activity`, `form_fill`, or `web_intent` as appropriate.

## Custom Objects

Use custom objects when teams store signal entities such as:

- account signal records
- intent records
- product workspaces
- onboarding records
- trial records
- account research records

Each custom object row must map to a CRM account/company/deal before it can surface in Slack.

## Rep-Authored Activity

Notes, calls, tasks, owner changes, and last-modified timestamps are rep-authored by default. Use them for context or tie-breaking only unless paired with buyer-authored or external-trigger rows.

## Optional Logging

If the user approves CRM writeback, log a note or custom signal object only after Slack QA. Include:

- run date
- account rank
- signal summary
- source/provider
- suggested action
- Slack message link if posted

Never write CRM notes before the user approves exact accounts.
