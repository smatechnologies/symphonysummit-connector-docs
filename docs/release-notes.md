---
sidebar_label: 'Release notes'
title: Symphony Summit Connector release notes
description: "Version history and change details for the Symphony Summit Connector, including new features, improvements, and bug fixes."
tags:
  - Reference
  - System Administrator
  - Connectors
---

# Symphony Summit Connector release notes

## General

This release of the Symphony Summit Connector is for OpCon system 21.0 or greater. The connector only supports connections to the OpCon API to retrieve job information and insert or update incident information.

## 24

### 24.3.0

**Released:** 2026 May

#### What's new

- **CON-1333**: Removed vulnerability CVE-2022-41404 by replacing the ini4j library with the Apache Commons Configuration library.

### 24.2.0

**Released:** 2025 October

#### What's new

- **CON-621**: Added the ability to assign a job to a specific person using the `Assigned_Engineer_Email` attribute.

### 24.1.0

**Released:** 2025 January

#### What's new

- **CONNUTIL-655**: Added an Incident view URL capability that allows a full definition of the incident viewing URL. To implement the change, add the new address to the template. This allows the incident address and the view address to differ when required; otherwise, define the `view-address` the same as the main address.

```json
"viewAddress": {
    "name": "view-address",
    "value": "address.com"
}
```

### 24.0.0

**Released:** 2024 October

Initial release of the Symphony Summit Connector.
