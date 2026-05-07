---
sidebar_label: 'Operation'
title: Symphony Summit Connector operation
description: "How to operate the Symphony Summit Connector after installation, including tag-driven routing, duplicate-ticket prevention, and day-to-day monitoring."
tags:
  - Conceptual
  - Operations Staff
  - Connectors
---

# Operation

## What is it?

This page describes the day-to-day operational behavior of the Symphony Summit Connector after it has been installed and configured.

There is very little to manage at run time — Notification Manager invokes the connector when an OpCon job fails — but a few rules influence how tickets are routed and whether duplicate tickets are suppressed.

- Use this when you operate an OpCon environment that already has the Symphony Summit Connector installed.
- Use this when you need to understand how OpCon job tags drive workgroup, category, or assigned-engineer routing on incidents.
- Use this when you want to prevent multiple incidents being created for the same job on the same calendar day.

## How a ticket is created

When an OpCon job fails, the connector follows this flow:

1. Notification Manager invokes the connector with the failed job's identifiers.
2. The connector reads the OpCon Daily Job table to retrieve the job's current state and tags.
3. If the job already has an Incident Ticket ID, the connector attaches the new failure information to the existing ticket and re-opens it.
4. Otherwise, the connector creates a new incident and writes the returned incident number into the **Incident Ticket ID** field of the OpCon job.
5. If **includeJobLogAttachment** is enabled, the connector retrieves the OpCon job log and attaches it to the incident.

:::tip
Use the **Incident Ticket ID** field on the OpCon job in the Daily tables to find the corresponding Symphony Summit incident.
:::

## Tag-driven routing

If at least one of the routing rules below is enabled in the template, OpCon job tags drive specific attributes on the incident.

| Rule | OpCon tag pattern | Attribute set on the incident |
| --- | --- | --- |
| `includeTagRouting` | Tag matches `TAG_START` or `TAG_END` rule in the template | `Assigned_WorkGroup_Name` |
| `includeWorkGroupNameTag` | `WRKGRP_<name>` | `Assigned_WorkGroup_Name` |
| `includeCategoryNameTag` | `CATNAME_<name>` | `Category_Name` |
| `includeAssignToTag` | `ASSIGNTO_<name>` | `Assigned_Engineer_Email` |

:::caution Rule precedence
**includeTagRouting** and **includeWorkGroupNameTag** are mutually exclusive in effect — both target the `Assigned_WorkGroup_Name` attribute. If both are set to `true`, the connector applies **includeWorkGroupNameTag** first and ignores **includeTagRouting**.
:::

The Tag Manager can be used to apply consistent tags across many jobs.

For full tag syntax and examples, see the related sections in [Installation](./installation.md#customization):

- [Tag routing](./installation.md#tag-routing)
- [Workgroup names from tags](./installation.md#workgroup-names-from-tags)
- [Category names from tags](./installation.md#category-names-from-tags)
- [AssignTo from tags](./installation.md#assignto-from-tags)

## Preventing duplicate tickets per day

The **submitSingleIncidentPerDay** rule suppresses additional incidents for the same job within a daily window.

When the rule is enabled:

1. On a job failure, the connector writes the failed calendar date to the OpCon job's **Documentation** field, prefixed to any existing documentation. For example:

   ```text
   Last Incident Date : 2024-09-21
   ```

2. On the next failure, the connector reads that date and adds the `DAILY_START_HOUR` value from `Connector.config`.
3. The current date and time are compared against the result. If the daily window has not yet rolled over, no new incident is created.

:::note
The window starts at `DAILY_START_HOUR` on the day **after** the recorded last incident date. Set `DAILY_START_HOUR` in `Connector.config` to align the window with your operations day.
:::

## FAQs

**Why is no incident being created when a job fails?**
Confirm that Notification Manager is configured to invoke the Symphony Summit Connector for the failed job. Then confirm:

- **submitSingleIncidentPerDay** has not already suppressed a duplicate ticket for the same job on the same day.
- The job's tags do not match an `EXIT` rule in the template — an `EXIT` match suppresses ticket creation.

**Why is a ticket being routed to the wrong workgroup?**
If both **includeTagRouting** and **includeWorkGroupNameTag** are `true`, the connector applies **includeWorkGroupNameTag** first. Confirm which rule is enabled and that the OpCon job tags use the corresponding pattern (`WRKGRP_<name>` for **includeWorkGroupNameTag**, or a `TAG_START` / `TAG_END` match for **includeTagRouting**).

**How do I tell which incident corresponds to a failed OpCon job?**
The connector writes the returned incident number into the **Incident Ticket ID** field of the failed job in the OpCon Daily tables. Use that field to correlate the OpCon job with the Symphony Summit incident.

**Why was the same job's failure attached to an existing ticket instead of creating a new one?**
Before creating a new ticket, the connector checks whether an Incident Ticket ID already exists on the job. If it does, the connector attaches the new failure information to the existing ticket and re-opens it.

**How do I capture more detail when troubleshooting?**
Set `DEBUG=ON` in the `[GENERAL]` section of `Connector.config` to enable verbose logging. Restore `DEBUG=OFF` once the issue is captured.

## Glossary

> **submitSingleIncidentPerDay** — A connector rule that, when enabled, suppresses additional ticket creation for the same OpCon job within a daily window defined by `DAILY_START_HOUR`.

> **includeTagRouting** — A connector rule that, when enabled, uses OpCon job user-defined tags to set the `Assigned_WorkGroup_Name` attribute of the incident.

> **includeWorkGroupNameTag** — A connector rule that, when enabled, sets the `Assigned_WorkGroup_Name` attribute from an OpCon tag of the form `WRKGRP_<name>`.

> **includeCategoryNameTag** — A connector rule that, when enabled, sets the `Category_Name` attribute from an OpCon tag of the form `CATNAME_<name>`.

> **includeAssignToTag** — A connector rule that, when enabled, sets the `Assigned_Engineer_Email` attribute from an OpCon tag of the form `ASSIGNTO_<name>`.

> **Tag Manager** — An OpCon tool used to manage user-defined tags on jobs, simplifying the task of attaching consistent tags across jobs.

> **DAILY_START_HOUR** — A `Connector.config` value used together with the recorded last incident date to determine whether enough time has elapsed for a new incident to be created.

## Related topics

- [Overview](./overview.md)
- [Installation](./installation.md)
- [Release notes](./release-notes.md)
