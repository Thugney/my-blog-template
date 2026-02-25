---
title: "Example: Intune Proactive Remediation for Registry Cleanup"
date: 2026-01-15
draft: false
tags: ["intune", "powershell", "how-to"]
categories: ["How-To"]
summary: "Learn how to create a proactive remediation script to clean up registry keys across your device fleet."
description: "Step-by-step guide on creating Intune proactive remediation scripts for registry cleanup with detection and remediation PowerShell scripts."
keywords: ["intune", "proactive remediation", "powershell", "registry"]
---

This is an example post demonstrating the blog structure. Replace this content with your own.

## Introduction

Explain the problem you're solving and why it matters.

## Prerequisites

- Microsoft Intune license
- Appropriate admin permissions
- PowerShell knowledge

## Detection Script

```powershell
# Example detection script
$registryPath = "HKLM:\SOFTWARE\Example"
if (Test-Path $registryPath) {
    Write-Output "Registry key found - remediation needed"
    exit 1
} else {
    Write-Output "Registry key not found - compliant"
    exit 0
}
```

## Remediation Script

```powershell
# Example remediation script
$registryPath = "HKLM:\SOFTWARE\Example"
try {
    Remove-Item -Path $registryPath -Recurse -Force
    Write-Output "Successfully removed registry key"
    exit 0
} catch {
    Write-Error "Failed to remove registry key: $_"
    exit 1
}
```

## Deployment Steps

1. Navigate to **Intune Admin Center** > **Devices** > **Remediations**
2. Click **Create script package**
3. Upload your detection and remediation scripts
4. Configure the schedule and scope

## Summary

Wrap up what was accomplished and any next steps.
