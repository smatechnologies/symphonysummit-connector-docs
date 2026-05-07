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

### 24.2.0

### What's new

:eight_spoked_asterisk: **CON-621**: Added the ability to assign a job to a specific person using the `Assigned_Engineer_Email` attribute.

### 24.1.0

### What's new

:eight_spoked_asterisk: **CONNUTIL-655**: Added an Incident view URL capability that allows a full definition of the incident viewing URL. To implement the change, add the new address to the template. This allows the incident address and the view address to differ when required; otherwise, define the `view-address` the same as the main address.

```json
"viewAddress": {
    "name": "view-address",
    "value": "address.com"
}
```

### 24.0.0

Initial release of the Symphony Summit Connector.
