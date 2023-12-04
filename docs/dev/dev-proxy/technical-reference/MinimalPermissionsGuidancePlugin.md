---
title: MinimalPermissionsGuidancePlugin
description: MinimalPermissionsGuidancePlugin reference
author: garrytrinder
ms.author: garrytrinder
ms.date: 11/03/2023
ms.topic: conceptual
ms.service: microsoft-cloud-for-developers

categories:
  - developer-tools
products:
  - microsoft-365
  - microsoft-graph
  - sharepoint-online
  - m365
ms.custom:
  - fcp
  - team=cloud_advocates
---

# MinimalPermissionsGuidancePlugin

Compares the permissions used in the JWT token sent to Microsoft Graph against the minimum required scopes needed for requests that proxy recorded and shows the difference.

## Plugin instance definition

```json
{
  "name": "MinimalPermissionsGuidancePlugin",
  "enabled": false,
  "pluginPath": "plugins\\dev-proxy-plugins.dll",
  "configSection": "minimalPermissionsGuidancePlugin"
}
```

## Configuration example

```json
{
  "minimalPermissionsGuidancePlugin": {
    "filePath": "permissions-summary.json"
  }
}
```

## Configuration properties

| Property | Description | Default |
|----------|-------------|:-------:|
| `filePath` | Path to the file where proxy saves the generated permissions summary. If not specified, proxy only displays the summary in the terminal. | None |

## Command line options

| Name | Description | Default |
|----------|-------------|:-------:|
| `--minimal-permissions-summary-file-path` | Path to the file where proxy saves the generated permissions summary. If not specified, proxy only displays the summary in the terminal. | None |