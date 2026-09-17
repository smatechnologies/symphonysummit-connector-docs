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

The connector runs under an OpCon Windows Agent on the OpCon Windows Server. Either use an existing Windows Agent on that server or install one, following the OpCon Windows Agent documentation.

No connector-specific agent configuration is required. Notification Manager uses the agent's **Run Command** option to start the connector, which is set up in [Step 5](#step-5-configure-notification-manager).

## Step 2: Install the Symphony Summit Connector

To install the connector, complete the following steps:

1. Copy the downloaded install file `SMASymphonySummitConnector-win.zip` to a temporary directory (for example, `c:\temp`).
2. Extract the contents, including subdirectories, into the required installation directory.
3. Create the **$SCHEDULE DATE-SSUM** global property:
   - Set the value to `yyyy-MM-dd`.
   - This property returns the schedule date in the `yyyy-MM-dd` format from the standard `$SCHEDULE DATE` property and is required by the connector.

After the extraction, the root installation directory contains the following items:

| Item | Purpose |
| --- | --- |
| `SMASymphonySummit.exe` | Connector executable |
| `EncryptValue.exe` | Credential encoding utility |
| `Connector.config` | Connector configuration file |
| `java\` | Embedded OpenJDK 11 runtime |
| `templates\` | Symphony Summit template files |

The connector also uses two directories that the package does not contain. Create them if they are not present:

| Directory | Purpose |
| --- | --- |
| `logFiles\` | Temporary storage for job logs retrieved from OpCon before they are attached to an incident. The name must match `LOG_FILES_DIRECTORY` in `Connector.config`. |
| `log\` | The connector's own log files. See [Logging](#logging). |

## Step 3: Configure the connector

Configuration of the connector requires three things:

- Encrypted credentials for the OpCon and Symphony Summit connections.
- A populated `Connector.config` file.
- At least one template (covered in [Step 4](#step-4-create-a-template)).

### 3.1 Encrypt sensitive values

The Symphony Summit `apiKey` must be encoded with the `EncryptValue.exe` utility provided with the connector before it is placed in a template. The connector decodes it on startup, so a plain-text key does not work. The utility supports a `-v` argument that prints the encoded value to the screen.

To encode a value, run the utility with the `-v` argument:

```bat
EncryptValue.exe -v "abcdefg"
```

:::caution

Despite its name, `EncryptValue.exe` **encodes** values rather than encrypting them. It applies no cipher and uses no key, so anyone who can read a template can recover the original value. Encoding stops a credential being read at a glance, and that is all it does.

Restrict access to `Connector.config` and the templates with operating system permissions, and treat every credential in them as recoverable.

:::

### 3.2 Configure Connector.config

`Connector.config` is divided into four sections. Set values in each section as described below.

#### `[GENERAL]`

| Property | Description | Default |
| --- | --- | --- |
| `LOG_FILES_DIRECTORY` | Subdirectory for retrieved job logs. After successful attachment to the Symphony Summit incident, the log file is deleted. | `logFiles` |
| `TEMPLATES_DIRECTORY` | Subdirectory containing the template definitions. | `templates` |
| `DAILY_START_HOUR` | The hour the daily batch processing starts, as two digits (for example, `07` for 07:00). **Required when the `submitSingleIncidentPerDay` rule is enabled** in any template. | none |
| `DEBUG` | Debug logging mode. Run with `OFF` and switch to `ON` to capture an error condition. | `OFF` |

:::caution
`DAILY_START_HOUR` has no default. If a template enables `submitSingleIncidentPerDay` and this property is missing or empty, the connector fails when it next handles a failure of a job that already has an incident. Set it whenever you use that rule.
:::

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

#### `[PROXY CONNECTION]` — optional proxy

| Property | Description | Default |
| --- | --- | --- |
| `USE_PROXY` | Whether the connector uses a proxy server. Values: `True` or `False`. | `False` |
| `PROXY_ADDRESS` | The address of the proxy server. | none |
| `PROXY_PORT` | The port of the proxy server. | none |

#### `[OPCON API CONNECTION]` — connection to OpCon

| Property | Description |
| --- | --- |
| `SERVER` | The server address of the OpCon API. |
| `PORT` | The port number used by the OpCon API server. |
| `USES_TLS` | Must be set to `True`. |
| `TOKEN` | An application token used for authentication. Generate this in OpCon. |

#### Example `Connector.config`

Replace every value in angle brackets with your own.

```ini
[GENERAL]
LOG_FILES_DIRECTORY=logFiles
TEMPLATES_DIRECTORY=templates
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

[PROXY CONNECTION]
USE_PROXY=False
PROXY_ADDRESS=
PROXY_PORT=

[OPCON API CONNECTION]
SERVER=<opcon-api-server>
PORT=9010
USES_TLS=True
TOKEN=<opcon-api-application-token>
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
| `credentials.apiKey` | The encoded API key with the privileges required to submit requests to Symphony Summit. Encode it with `EncryptValue.exe`. |

:::caution
The `apiKey` value must be encoded using `EncryptValue.exe` before being placed in the template. The connector decodes it unconditionally, so a plain-text key causes the connector to fail on startup.
:::

### 4.2 Rules

Rules turn features on or off for a given template. Every rule defaults to `false`, so set only the rules you want to enable.

:::note
Job log attachment is off by default. The example template further down this page switches it on, so a template based on the example attaches job logs and a minimal template built from this table does not.
:::

| Rule | Description | Default |
| --- | --- | --- |
| `includeJobLogAttachment` | Attach the OpCon job log to the incident. Requires an `attachment` entry in `urls`. | `false` |
| `includeTagRouting` | Use OpCon tags to set the `Assigned_WorkGroup_Name` attribute. See [Tag routing](#tag-routing). | `false` |
| `includeWorkGroupNameTag` | Use a `WRKGRP_<name>` tag to set the `Assigned_WorkGroup_Name` attribute. See [Workgroup names from tags](#workgroup-names-from-tags). | `false` |
| `includeCategoryNameTag` | Use a `CATNAME_<name>` tag to set the `Category_Name` attribute. See [Category names from tags](#category-names-from-tags). | `false` |
| `includeAssignToTag` | Use an `ASSIGNTO_<name>` tag to set the `Assigned_Engineer_Email` and `Assign_To` attributes. See [AssignTo from tags](#assignto-from-tags). | `false` |
| `submitSingleIncidentPerDay` | Suppress duplicate incidents for the same job within a daily window. Requires `DAILY_START_HOUR` in `Connector.config`, which defines the start of the daily window. | `false` |

:::caution Mutually exclusive rules
Either **includeTagRouting** or **includeWorkGroupNameTag** can be enabled, not both. When both are set to `true`, the connector applies **includeWorkGroupNameTag** first.
:::

### 4.3 URLs

`urls` is a list of URL definitions used by the connector. Each entry has a `name` and a `value`. The address portion is omitted from `value` because the connector prefixes it with the value from `address.value`.

| Name | Required? | Value |
| --- | --- | --- |
| `incident` | Required | URL path used to create an incident. |
| `attachment` | Required if `includeJobLogAttachment` is `true` | URL path used to upload an attachment. |
| `viewIncident` | Required | Full URL pattern used to construct a view link for the incident. `{0}` is replaced with `viewAddress.value` and `{1}` with the incident's identifier. |

:::caution
All three entries must be present. `viewIncident` is used on every ticket the connector creates, so a template without it fails on the first job failure even though nothing else references it.
:::

### 4.4 Working hours

`workingHours` defines a start and stop time for each day of the week, allowing different attribute values to be applied during working and non-working hours.

Each day is an object with `start` and `stop`, each formatted as four digits (`HHMM`). Set both to `0000` to treat the whole day as non-working.

:::caution Windows cannot span midnight
The connector compares the current hour against the start and stop hours of the same day, so `stop` must be later than `start`. A window such as `start` `2200` and `stop` `0200` never matches any hour, and every failure is treated as non-working hours. To cover an overnight operations period, use a window that ends at `2359`.
:::

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

`tags` is a list of routing rules. Not every tag feature reads it:

| Rule | Needs an entry in `tags`? |
| --- | --- |
| **includeTagRouting** | Yes — one entry per `TAG_START` / `TAG_END` match, plus `DEFAULT` |
| **includeAssignToTag** | Yes — an `ASSIGNTO` entry, whose `value` supplies the email domain |
| **includeWorkGroupNameTag** | No — the connector reads the `WRKGRP_` job tag directly |
| **includeCategoryNameTag** | No — the connector reads the `CATNAME_` job tag directly |

An `EXIT` entry is read whenever `tags` is present, independently of any rule.

| Field | Description |
| --- | --- |
| `indicator` | The match mode: `TAG_END`, `TAG_START`, `DEFAULT`, `EXIT`, `CATNAME`, `WRKGRP`, or `ASSIGNTO`. |
| `indicatorValue` | The value matched against the OpCon tag. |
| `attribute` | The ticket attribute name set when the rule matches. Used by `TAG_START`, `TAG_END` and `DEFAULT` entries only; the `ASSIGNTO`, `CATNAME`, `WRKGRP` and `EXIT` entries ignore it. |
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
    { "indicator": "ASSIGNTO",  "indicatorValue": "ASSIGNTO", "attribute": "Assigned_Engineer_Email", "value": "@example.com" },
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

#### Optional arguments

The connector accepts two further arguments that the run command above does not use.

| Argument | Purpose |
| --- | --- |
| `-i <number>` | Retrieves an existing incident by number. Not used when creating incidents from a job failure. |
| `--tlsType <list>` | Sets the TLS protocol versions the connector offers, as a comma-separated list. Accepted values are `TLSv1`, `TLSv1.1`, `TLSv1.2`, and `NONE`. `NONE` means the connector does not set the protocol list and the Java runtime's own default applies. |

:::note
The default for `--tlsType` is `TLSv1,TLSv1.1,TLSv1.2`, which offers two protocol versions that current Java runtimes disable. To restrict the connector to `TLSv1.2`, pass `--tlsType TLSv1.2`.
:::

---

## Logging

The connector writes its own log to the `log` directory beneath the installation directory.

| Item | Value |
| --- | --- |
| Active log file | `log\symphonysummit.log` |
| Rotation | A new file is started when the active log reaches 100 MB. |
| Retention | **Unlimited by default.** Rotated files are never deleted. |
| Level written to file | `DEBUG` |
| Level written to the console | `INFO` |

:::caution
Rotated log files are kept indefinitely. On a busy system the `log` directory grows without limit, so include it in whatever housekeeping you apply to the OpCon Windows Server.
:::

Setting `DEBUG=ON` in the `[GENERAL]` section of `Connector.config` raises the detail captured for troubleshooting. Return it to `OFF` once the condition has been captured.

Job logs retrieved from OpCon are written to the `logFiles` directory and deleted once they have been attached to an incident. A file left behind in that directory indicates an attachment that did not complete.

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

**Requires:** **includeWorkGroupNameTag** = `true`. No entry in the template's `tags` list is needed — the connector reads the job tag directly.

If no `WRKGRP_` tag is found, the `ASSIGNED_WORK_GROUP_NAME_VALUE` default from `Connector.config` is used.

```text
OpCon tags: APP1, WRKGRP_DevOps, TESTING
```

`DevOps` is extracted from `WRKGRP_DevOps` and assigned to the `Assigned_WorkGroup_Name` attribute.

### Category names from tags

Use a `CATNAME_<name>` tag on the OpCon job to set the `Category_Name` attribute on the incident. The connector strips the `CATNAME_` prefix and uses the remainder as the category name.

**Requires:** **includeCategoryNameTag** = `true`. No entry in the template's `tags` list is needed — the connector reads the job tag directly.

If no `CATNAME_` tag is found, the `CATEGORY_NAME_VALUE` default from `Connector.config` is used.

```text
OpCon tags: APP1_ROUTE1, CATNAME_Elasticsearch
```

`Elasticsearch` is extracted from `CATNAME_Elasticsearch` and assigned to the `Category_Name` attribute.

### AssignTo from tags

Use an `ASSIGNTO_<name>` tag on the OpCon job to set the `Assigned_Engineer_Email` and `Assign_To` attributes on the incident. The connector strips the `ASSIGNTO_` prefix, uses the remainder as the user portion of the address, and appends the `value` from the `ASSIGNTO` rule.

**Requires:** **includeAssignToTag** = `true`, and an `ASSIGNTO` entry in the template's `tags` list.

| Field | How the connector uses it |
| --- | --- |
| `indicator` | Must be `ASSIGNTO`. |
| `value` | Appended to the name taken from the job tag. Include the `@`, because the connector joins the two values without adding one. |
| `attribute` | Not used. Both `Assigned_Engineer_Email` and `Assign_To` are set regardless of what this field contains. |

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
    "value": "@example.com"
  },
  {
    "indicator": "DEFAULT",
    "indicatorValue": "DEFAULT",
    "attribute": "Assigned_WorkGroup_Name",
    "value": "DevOps"
  }
]
```

`test` is extracted from `ASSIGNTO_test` and combined with the `value` `@example.com`. The `Assigned_Engineer_Email` and `Assign_To` attributes are both set to `test@example.com`.

---

## FAQs

**Why must the connector be installed on the OpCon Windows Server?**
Notification Manager invokes the connector using the **Run Command** option, which runs the command on the same server as Notification Manager. Installing the connector on the same server allows Notification Manager to start it directly when a job fails.

**Why does the OpCon REST API have to use TLS?**
The connector communicates with the OpCon system to retrieve job information and to update the incident ticket ID on the job. The `Connector.config` requires `USES_TLS=True` for this connection.

**Where do I get an OpCon application token?**
Generate an application token using the OpCon REST API, then record the token in the `TOKEN` value of the `[OPCON API CONNECTION]` section of `Connector.config`.

**Why must the apiKey be encoded?**
Because the connector decodes it on startup and a plain-text key does not work. Note that `EncryptValue.exe` encodes rather than encrypts — no cipher, no key — so encoding is not a security control. Anyone who can read the template can recover the key. Protect the file with operating system permissions.

**Can I send incidents to more than one Symphony Summit instance from the same OpCon system?**
Yes. Create a separate template for each Symphony Summit instance and pass the appropriate template name to the connector using the `-t` argument.

**What is the $SCHEDULE DATE-SSUM property used for?**
It is a special version of the schedule date in the `yyyy-MM-dd` format that the connector requires. Notification Manager passes this value as the `-sd` argument to the connector.

**Can I enable both includeTagRouting and includeWorkGroupNameTag?**
The two rules are mutually exclusive in effect. If both are set to `true`, the connector applies **includeWorkGroupNameTag** first and ignores **includeTagRouting**.

## Glossary

> **EncryptValue** — Utility (`EncryptValue.exe`) shipped with the connector that produces encoded values for use in templates. It encodes rather than encrypts: the original value can be recovered from the encoded form.

> **Connector.config** — The configuration file that defines the OpCon API connection, default ticket attribute values, the proxy server, and global behavior such as debug mode.

> **Template** — A JSON file in the `templates` directory that defines the connection to a Symphony Summit instance, the rules, attribute values, and tag-routing definitions.

> **apiKey** — Encoded Symphony Summit API key recorded in the `credentials` section of a template; used to authenticate with a Symphony Summit instance.

> **Application token** — A token generated through the OpCon REST API that the connector uses to authenticate when communicating with the OpCon system.

> **$SCHEDULE DATE-SSUM** — A global OpCon property created during installation that returns the schedule date in `yyyy-MM-dd` format.

## Related topics

- [Overview](./overview.md)
- [Operation](./operation.md)
- [Release notes](./release-notes.md)
