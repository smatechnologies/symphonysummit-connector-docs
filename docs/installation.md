---
sidebar_label: 'Installation'
title: Symphony Summit Connector installation
description: "How to install and configure the Symphony Summit Connector, including supported software levels, EncryptValue utility usage, Connector.config settings, templates, and Notification Manager wiring."
tags:
  - Procedural
  - System Administrator
  - Connectors
---

# Installation

## What is it?

This page describes how to install and configure the Symphony Summit Connector so it can submit incident creation requests to a Symphony Summit instance when an OpCon job fails.

- Use this when you set up the connector for the first time on an OpCon Windows Server.
- Use this when you upgrade or reconfigure the connector to point at a different Symphony Summit instance.
- Use this when you add a new Notification Manager job trigger that should create an incident on job failure.

The connector relies on three OpCon components:

- **Notification Manager** runs the connector when a job fails.
- The **OpCon Windows Agent** hosts the connector executable.
- The **OpCon REST API** identifies the failed job and retrieves its job log.

## Before you begin

The following software levels are required to implement the Symphony Summit Connector.

| Component | Required level |
| --- | --- |
| OpCon | Release 21.0 or higher |
| OpCon REST API | Configured to use TLS |
| OpCon Windows Agent | Installed on the OpCon Windows Server |
| OpCon Notification Manager | Installed and configured |
| Symphony Summit | An instance that supports the required REST API |
| Java | OpenJDK 11 (shipped with the connector — no separate install needed) |

:::caution Where the connector runs
The Symphony Summit Connector must be installed on the **OpCon Windows Server** because Notification Manager invokes it locally using the **Run Command** option.
:::

## Installation overview

To install the Symphony Summit Connector, complete the following steps:

1. [Install the OpCon Windows Agent](#step-1-install-the-opcon-windows-agent).
2. [Install the Symphony Summit Connector](#step-2-install-the-symphony-summit-connector).
3. [Configure `Connector.config`](#step-3-configure-the-connector).
4. [Create a template](#step-4-create-a-template).
5. [Configure Notification Manager](#step-5-configure-notification-manager).

---

## Step 1: Install the OpCon Windows Agent

Copy the supplied install file `SMASymphonySummitConnector-win.zip` and extract it into the installation directory.

After the installation is complete, the root installation directory contains the following items:

| Item | Purpose |
| --- | --- |
| `SMASymphonySummit.exe` | Connector executable |
| `EncryptValue.exe` | Encryption utility for credentials |
| `Connector.config` | Connector configuration file |
| `java\` | Embedded OpenJDK 11 runtime |
| `joblogs\` | Temporary storage for job logs retrieved from OpCon |
| `templates\` | Symphony Summit template files |
| `log\` | Connector log files |

## Step 2: Install the Symphony Summit Connector

To install the connector, complete the following steps:

1. Copy the downloaded install file `SMASymphonySummitConnector-win.zip` to a temporary directory (for example, `c:\temp`).
2. Extract the contents, including subdirectories, into the required installation directory.
3. Create the **$SCHEDULE DATE-SSUM** global property:
   - Set the value to `yyyy-MM-dd`.
   - This property returns the schedule date in the `yyyy-MM-dd` format from the standard `$SCHEDULE DATE` property and is required by the connector.

## Step 3: Configure the connector

Configuration of the connector requires three things:

- Encrypted credentials for the OpCon and Symphony Summit connections.
- A populated `Connector.config` file.
- At least one template (covered in [Step 4](#step-4-create-a-template)).

### 3.1 Encrypt sensitive values

All user and password values placed in the configuration and template files must be encrypted using the `EncryptValue.exe` utility provided with the connector. The utility uses standard 64-bit encryption and supports a `-v` argument that prints the encrypted value to the screen.

To encrypt a value, run the utility with the `-v` argument:

```bat
EncryptValue.exe -v "abcdefg"
```

:::tip
Encrypt the Symphony Summit `apiKey` and the OpCon application `TOKEN` before pasting them into the template or `Connector.config`.
:::

### 3.2 Configure Connector.config

`Connector.config` is divided into four sections. Set values in each section as described below.

#### `[GENERAL]`

| Property | Description | Default |
| --- | --- | --- |
| `JOBLOGDIR` | Subdirectory for retrieved job logs. After successful attachment to the Symphony Summit incident, the log file is deleted. | `joblogs` |
| `TEMPLATESDIR` | Subdirectory containing the template definitions. | `templates` |
| `DAILY_START_HOUR` | The hour the daily batch processing starts (for example, `07` for 07:00). | — |
| `DEBUG` | Debug logging mode. Run with `OFF` and switch to `ON` to capture an error condition. | `OFF` |

#### `[DEFAULTS]` — default ticket attribute values

| Property | Sets the default value for | Default |
| --- | --- | --- |
| `PRIORITY_NAME_VALUE` | `Priority_Name` | `P3` |
| `IMPACT_NAME_VALUE` | `Impact_Name` | `Medium` |
| `URGENCY_NAME_VALUE` | `Urgency_Name` | `Medium` |
| `SUP_FUNCTION_VALUE` | `Sup_Function` | `IT` |
| `MEDIUM_VALUE` | `Medium` | `Web` |
| `CLASSIFICATION_NAME_VALUE` | `Classification_Name` | `Application Support` |
| `SOURCE_VALUE` | `Source` | `Event Trigger` |
| `CATEGORY_NAME_VALUE` | `Category_Name` | `ElasticSearch` |
| `ASSIGNED_WORK_GROUP_NAME_VALUE` | `Assigned_WorkGroup_Name` | `DevOps` |

#### `[PROXY SERVER]` — optional proxy

| Property | Description | Default |
| --- | --- | --- |
| `USES_PROXY` | Whether the connector uses a proxy server. Values: `True` or `False`. | `False` |
| `ADDRESS` | The address of the proxy server. | — |
| `PORT` | The port of the proxy server. | — |

#### `[OPCON API]` — connection to OpCon

| Property | Description |
| --- | --- |
| `ADDRESS` | The server address of the OpCon API. |
| `PORT` | The port number used by the OpCon API server. |
| `USES_TLS` | Must be set to `True`. |
| `TOKEN` | An application token used for authentication. Generate this in OpCon. |

#### Example `Connector.config`

```ini
[GENERAL]
JOBLOGDIR=joblogs
TEMPLATESDIR=templates
DAILY_START_HOUR=07
DEBUG=OFF

[DEFAULTS]
PRIORITY_NAME_VALUE=P3
IMPACT_NAME_VALUE=Medium
URGENCY_NAME_VALUE=Medium
SUP_FUNCTION_VALUE=IT
MEDIUM_VALUE=Web
CLASSIFICATION_NAME_VALUE=Application Support
SOURCE_VALUE=Event Trigger
CATEGORY_NAME_VALUE=ElasticSearch
ASSIGNED_WORK_GROUP_NAME_VALUE=DevOps

[PROXY SERVER]
USES_PROXY=False
SERVER=
PORT=

[OPCON API]
SERVER=BVHTEST02
PORT=9010
USES_TLS=True
TOKEN=fc0520dc-fc93-4d3a-bf2f-7d0584c69df2
```

:::note
The `TOKEN` value is the application token generated through the OpCon REST API. See the OpCon REST API documentation for instructions on creating an application token.
:::

---

## Step 4: Create a template

Templates provide information about the Symphony Summit connection and the JSON payload submitted with each request. A default template is in the `templates` directory of the connector installation.

A single connector can submit incidents to multiple Symphony Summit instances by creating multiple templates and selecting them at run time using the `-t` argument on the connector command line.

A template defines:

- The Symphony Summit instance address and view address.
- Encrypted credentials.
- Rules that control which features are active.
- URLs used by the connector.
- Working hours.
- Default attributes, working-hours overrides, and custom attributes.
- Tag-routing definitions.

### 4.1 Address, view address, and credentials

| Field | Description |
| --- | --- |
| `address.name` | A name to identify the Symphony Summit instance. |
| `address.value` | The address of the Symphony Summit instance. |
| `viewAddress.name` | A name for the view-address entry. |
| `viewAddress.value` | The address used to view incidents. May differ from `address.value`. |
| `credentials.apiKey` | The encrypted API key with the privileges required to submit requests to Symphony Summit. |

:::caution
The `apiKey` value must be encrypted using `EncryptValue.exe` before being placed in the template.
:::

### 4.2 Rules

Rules turn features on or off for a given template.

| Rule | Description | Default |
| --- | --- | --- |
| `includeJobLogAttachment` | Attach the OpCon job log to the incident. | `true` |
| `includeTagRouting` | Use OpCon tags to set the `Assigned_WorkGroup_Name` attribute. See [Tag routing](#tag-routing). | `false` |
| `includeWorkGroupNameTag` | Use a `WRKGRP_<name>` tag to set the `Assigned_WorkGroup_Name` attribute. See [Workgroup names from tags](#workgroup-names-from-tags). | `false` |
| `includeCategoryNameTag` | Use a `CATNAME_<name>` tag to set the `Category_Name` attribute. See [Category names from tags](#category-names-from-tags). | `false` |
| `includeAssignToTag` | Use an `ASSIGNTO_<name>` tag to set the `Assigned_Engineer_Email` attribute. See [AssignTo from tags](#assignto-from-tags). | `false` |
| `submitSingleIncidentPerDay` | Suppress duplicate incidents for the same job within a daily window. The `DAILY_START_HOUR` value in `Connector.config` defines the start of the daily window. | — |

:::caution Mutually exclusive rules
Either **includeTagRouting** or **includeWorkGroupNameTag** can be enabled, not both. When both are set to `true`, the connector applies **includeWorkGroupNameTag** first.
:::

### 4.3 URLs

`urls` is a list of URL definitions used by the connector. Each entry has a `name` and a `value`. The address portion is omitted from `value` because the connector prefixes it with the value from `address.value`.

| Name | Value |
| --- | --- |
| `incident` | URL path used to create an incident. |
| `attachment` | URL path used to upload an attachment. |
| `viewIncident` | Full URL pattern used to construct a view link for the incident. |

### 4.4 Working hours

`workingHours` defines a start and stop time for each day of the week, allowing different attribute values to be applied during working and non-working hours.

Each day is an object with `start` and `stop`, each formatted as four digits (`HHMM`). Set both to `0000` to skip a day.

### 4.5 Attributes

The connector applies attribute values in the following order:

1. Defaults from `Connector.config`.
2. `attributes` from the template.
3. `workingHoursAttributes` if the current time is within working hours.
4. `nonWorkingHoursAttributes` if the current time is outside working hours.

| Block | Purpose |
| --- | --- |
| `attributes` | Attribute values that override the defaults from `Connector.config`. |
| `workingHoursAttributes` | Attribute values applied only during working hours. |
| `nonWorkingHoursAttributes` | Attribute values applied only outside working hours. |
| `customAttributes` | Custom attributes added to the `CustomFields` section of the ticket information. Each entry has `groupName`, `name`, and `value`. |

Common attribute names: `Priority_Name`, `Impact_Name`, `Urgency_Name`, `Classification_Name`, `Sup_Function`, `Medium`, `Source`.

### 4.6 Tag-routing definitions

`tags` is a list of routing rules used when at least one of **includeTagRouting**, **includeWorkGroupNameTag**, **includeCategoryNameTag**, or **includeAssignToTag** is enabled.

| Field | Description |
| --- | --- |
| `indicator` | The match mode: `TAG_END`, `TAG_START`, `DEFAULT`, `EXIT`, `CATNAME`, `WRKGRP`, or `ASSIGNTO`. |
| `indicatorValue` | The value matched against the OpCon tag. |
| `attribute` | The ticket attribute name set when the rule matches. |
| `value` | The value assigned to the attribute when the rule matches. |

See [Tag routing](#tag-routing) and the related sections below for detailed examples.

### Example template

```json
{
  "ticketDescription": "OpCon job failure ( date @EV_Date schedule @EV_Schedule job @EV_Job server @EV_Agent error code @EV_errorcode )",
  "ticketInformation": "Test ticket created from API. Please ignore!!",
  "address": {
    "name": "production",
    "value": "Symphony Summit Instance address"
  },
  "viewAddress": {
    "name": "view-address",
    "value": "Symphony Summit Instance address"
  },
  "rules": {
    "includeJobLogAttachment": true,
    "includeTagRouting": true,
    "includeCategoryNameTag": true,
    "includeWorkGroupNameTag": false,
    "submitSingleIncidentPerDay": true,
    "includeAssignToTag": true
  },
  "credentials": {
    "apiKey": "encrypted key"
  },
  "urls": [
    {
      "name": "incident",
      "value": "api_integration/REST/Summit_RESTWCF.svc/RESTService/CommonWS_JsonObjCall_JSON"
    },
    {
      "name": "attachment",
      "value": "api_integration/REST/Summit_RESTWCF.svc/RESTService/Summit_UploadAttachmentBase64Encoded"
    },
    {
      "name": "viewIncident",
      "value": "https://{0}/MDLIncidentMgmt/IM_TicketDetail.aspx?ID={1}"
    }
  ],
  "workingHours": {
    "monday":    { "start": "0800", "stop": "1900" },
    "tuesday":   { "start": "0800", "stop": "1900" },
    "wednesday": { "start": "0800", "stop": "1900" },
    "thursday":  { "start": "0800", "stop": "1900" },
    "friday":    { "start": "0800", "stop": "1900" },
    "saturday":  { "start": "0800", "stop": "1100" },
    "sunday":    { "start": "0000", "stop": "0000" }
  },
  "attributes": [
    { "name": "Impact_Name",  "value": "Low" },
    { "name": "Urgency_Name", "value": "High" }
  ],
  "workingHoursAttributes": [],
  "nonWorkingHoursAttributes": [],
  "customAttributes": [
    { "groupName": "Other Details", "name": "Job Name", "value": "@EV_Job" },
    { "groupName": "Other Details", "name": "Abend",    "value": "Yes" }
  ],
  "tags": [
    { "indicator": "TAG_END",   "indicatorValue": "ROUTE1",   "attribute": "Assigned_WorkGroup_Name", "value": "DevOps" },
    { "indicator": "TAG_START", "indicatorValue": "ROUTE2",   "attribute": "Assigned_WorkGroup_Name", "value": "Operating SystemOrg2" },
    { "indicator": "EXIT",      "indicatorValue": "NOTICKET", "attribute": "",                        "value": "" },
    { "indicator": "CATNAME",   "indicatorValue": "CATNAME",  "attribute": "Category_Name",           "value": "testcatvalue" },
    { "indicator": "ASSIGNTO",  "indicatorValue": "ASSIGNTO", "attribute": "Assign_To",               "value": "service-now.com" },
    { "indicator": "DEFAULT",   "indicatorValue": "DEFAULT",  "attribute": "Assigned_WorkGroup_Name", "value": "DevOps" }
  ]
}
```

---

## Step 5: Configure Notification Manager

Notification Manager runs the Symphony Summit Connector when a job completes with a failure condition. Adding the connector to a Notification Manager rule lets you assign it to many jobs at once instead of defining a failure event on every job.

To configure Notification Manager, complete the following steps:

1. In Notification Manager, on the **Jobs** tab, create a new group named **SymphonySummit**.
2. Open the context menu for the **SymphonySummit** group and select **Add Job Trigger**.
3. In the Add Job Trigger dialog, select **Job Failed**.
4. On the **Run Command** tab, enter the values listed below.

#### Run command

```text
C:\Connectors\SymphonySummit\SMASymphonySummit.exe -a [[$MACHINE NAME]] -s [[$SCHEDULE NAME]] -jn [[$JOB NAME]] -e [[$JOB TERMINATION]] -sd [[$SCHEDULE DATE-SSUM]] -si [[$SCHEDULE ID]] -sn [[$SCHEDULE INST]] -t basic.json
```

| Argument | Resolves to |
| --- | --- |
| `C:\Connectors\SymphonySummit\SMASymphonySummit.exe` | Path to the connector executable. |
| `-a [[$MACHINE NAME]]` | Agent name. |
| `-s [[$SCHEDULE NAME]]` | Schedule name. |
| `-jn [[$JOB NAME]]` | Job name. |
| `-e [[$JOB TERMINATION]]` | Job termination code. |
| `-sd [[$SCHEDULE DATE-SSUM]]` | Date in `YYYY-MM-DD` format. |
| `-si [[$SCHEDULE ID]]` | Schedule ID. |
| `-sn [[$SCHEDULE INST]]` | Schedule instance. |
| `-t basic.json` | Template in the `templates` folder to use. |

#### Other Run Command settings

| Setting | Value |
| --- | --- |
| **Working Directory** | `C:\Connectors\SymphonySummit` |
| **Batch User** | Use Service Account — the batch user under which the job runs. |

---

## Customization

### Description and information placeholders

The `ticketDescription` and `ticketInformation` template values can include the following placeholders. The connector substitutes the placeholders with arguments passed by Notification Manager.

| Placeholder | Substituted with |
| --- | --- |
| `@EV_Agent` | Name of the agent that ran the failing job. |
| `@EV_Date` | Date when the failure occurred (`yyyy-MM-dd`). |
| `@EV_errorcode` | Job termination code. |
| `@EV_Job` | Name of the job that failed. |
| `@EV_Schedule` | Name of the schedule that contained the failed job. |

The same placeholders are supported in `attributes` and `customAttributes` values.

### Tag routing

Tag routing uses an OpCon tag prefix or suffix to set the `Assigned_WorkGroup_Name` attribute on the ticket.

**Requires:** **includeTagRouting** = `true`.

| Indicator | What it matches |
| --- | --- |
| `TAG_END` | An OpCon tag that **ends** with the `indicatorValue`. |
| `TAG_START` | An OpCon tag that **starts** with the `indicatorValue`. |
| `DEFAULT` | Used when no other rule matches. |
| `EXIT` | Suppresses ticket creation when the OpCon tag matches the `indicatorValue`. |

The connector evaluates `EXIT` rules before any other tag routing rules.

#### Example 1 — match on tag suffix

```text
OpCon tag: APP1_ROUTE1
```

```json
"tags": [
  {
    "indicator": "TAG_END",
    "indicatorValue": "ROUTE1",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "Application One"
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

The ticket's `Assigned_WorkGroup_Name` is set to **Application One**.

#### Example 2 — fallback to DEFAULT

```text
OpCon tag: APP_ONE
```

```json
"tags": [
  {
    "indicator": "TAG_END",
    "indicatorValue": "ROUTE1",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "Application One"
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

`APP_ONE` does not end with `ROUTE1`, so the `Assigned_WorkGroup_Name` falls through to the `DEFAULT` value, **DevOps**.

#### Example 3 — suppress ticket with EXIT

```text
OpCon tags: APP1_ROUTE1, NOTICKET
```

```json
"tags": [
  {
    "indicator": "TAG_END",
    "indicatorValue": "ROUTE1",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "Application One"
  },
  {
    "indicator": "EXIT",
    "indicatorValue": "NOTICKET",
    "attribute": "",
    "value": ""
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

`NOTICKET` matches the `EXIT` rule, so no ticket is created.

### Workgroup names from tags

Use a `WRKGRP_<name>` tag on the OpCon job to set the `Assigned_WorkGroup_Name` attribute on the incident. The connector strips the `WRKGRP_` prefix and uses the remainder as the workgroup name.

**Requires:** **includeWorkGroupNameTag** = `true`.

| Indicator | Description |
| --- | --- |
| `WRKGRP` | Prefix used for the workgroup name check. |

```text
OpCon tags: APP1, WRKGRP_DevOps, TESTING
```

`DevOps` is extracted from `WRKGRP_DevOps` and assigned to the `Assigned_WorkGroup_Name` attribute.

### Category names from tags

Use a `CATNAME_<name>` tag on the OpCon job to set the `Category_Name` attribute on the incident. The connector strips the `CATNAME_` prefix and uses the remainder as the category name.

**Requires:** **includeCategoryNameTag** = `true`.

| Indicator | Description |
| --- | --- |
| `CATNAME` | Prefix used for the category name check. |

```text
OpCon tags: APP1_ROUTE1, CATNAME_Elasticsearch
```

```json
"tags": [
  {
    "indicator": "TAG_END",
    "indicatorValue": "ROUTE1",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "Application One"
  },
  {
    "indicator": "CATNAME",
    "indicatorValue": "CATNAME",
    "attribute": "Category_Name",
    "value": ""
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

`Elasticsearch` is extracted from `CATNAME_Elasticsearch` and assigned to the `Category_Name` attribute.

### AssignTo from tags

Use an `ASSIGNTO_<name>` tag on the OpCon job to set the `Assigned_Engineer_Email` attribute on the incident. The connector strips the `ASSIGNTO_` prefix, uses the remainder as the user portion of the address, and appends the `value` from the `ASSIGNTO` rule (typically a domain).

**Requires:** **includeAssignToTag** = `true`.

| Indicator | Description |
| --- | --- |
| `ASSIGNTO` | Prefix used for the AssignTo check. |

```text
OpCon tags: APP1_ROUTE1, ASSIGNTO_test
```

```json
"tags": [
  {
    "indicator": "TAG_END",
    "indicatorValue": "ROUTE1",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "Application One"
  },
  {
    "indicator": "ASSIGNTO",
    "indicatorValue": "ASSIGNTO",
    "attribute": "Assigned_Engineer_Email",
    "value": "@service-now.com"
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

`test` is extracted from `ASSIGNTO_test` and combined with the `value` `@service-now.com`. The `Assigned_Engineer_Email` attribute is set to `test@service-now.com`.

---

## FAQs

**Why must the connector be installed on the OpCon Windows Server?**
Notification Manager invokes the connector using the **Run Command** option, which runs the command on the same server as Notification Manager. Installing the connector on the same server allows Notification Manager to start it directly when a job fails.

**Why does the OpCon REST API have to use TLS?**
The connector communicates with the OpCon system to retrieve job information and to update the incident ticket ID on the job. The `Connector.config` requires `USES_TLS=True` for this connection.

**Where do I get an OpCon application token?**
Generate an application token using the OpCon REST API, then record the token in the `TOKEN` value of the `[OPCON API]` section of `Connector.config`.

**Why are credentials encrypted?**
The connector requires that the Symphony Summit `apiKey` and any user or password values placed in the configuration and template files are encrypted with `EncryptValue.exe`. Storing only encrypted values keeps secrets out of plain text on disk.

**Can I send incidents to more than one Symphony Summit instance from the same OpCon system?**
Yes. Create a separate template for each Symphony Summit instance and pass the appropriate template name to the connector using the `-t` argument.

**What is the $SCHEDULE DATE-SSUM property used for?**
It is a special version of the schedule date in the `yyyy-MM-dd` format that the connector requires. Notification Manager passes this value as the `-sd` argument to the connector.

**Can I enable both includeTagRouting and includeWorkGroupNameTag?**
The two rules are mutually exclusive in effect. If both are set to `true`, the connector applies **includeWorkGroupNameTag** first and ignores **includeTagRouting**.

## Glossary

> **EncryptValue** — Utility (`EncryptValue.exe`) shipped with the connector that produces encrypted values for use in `Connector.config` and templates.

> **Connector.config** — The configuration file that defines the OpCon API connection, default ticket attribute values, the proxy server, and global behavior such as debug mode.

> **Template** — A JSON file in the `templates` directory that defines the connection to a Symphony Summit instance, the rules, attribute values, and tag-routing definitions.

> **apiKey** — Encrypted Symphony Summit API key recorded in the `credentials` section of a template; used to authenticate with a Symphony Summit instance.

> **Application token** — A token generated through the OpCon REST API that the connector uses to authenticate when communicating with the OpCon system.

> **$SCHEDULE DATE-SSUM** — A global OpCon property created during installation that returns the schedule date in `yyyy-MM-dd` format.

## Related topics

- [Overview](./overview.md)
- [Operation](./operation.md)
- [Release notes](./release-notes.md)
