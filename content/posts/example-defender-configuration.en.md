---
title: "Example: Configuring Microsoft Defender Attack Surface Reduction Rules"
date: 2026-01-10
draft: false
tags: ["defender", "security", "how-to"]
categories: ["How-To"]
summary: "A guide to implementing Attack Surface Reduction (ASR) rules via Intune for enhanced endpoint security."
description: "Learn how to configure and deploy Microsoft Defender Attack Surface Reduction rules through Intune to protect your organization."
keywords: ["defender", "ASR", "attack surface reduction", "intune", "security"]
---

This is an example post showing the Defender tag cover color. Replace with your content.

## Introduction

Attack Surface Reduction rules help prevent actions that malware often abuses to compromise devices.

## Prerequisites

- Microsoft Defender for Endpoint license
- Intune admin access
- Windows 10/11 devices

## Configuration Steps

1. Navigate to **Intune** > **Endpoint Security** > **Attack Surface Reduction**
2. Create a new policy
3. Configure the desired rules

## Common ASR Rules

| Rule | Description |
|------|-------------|
| Block Office apps from creating child processes | Prevents Office from spawning processes |
| Block credential stealing from LSASS | Protects credentials |
| Block executable content from email | Prevents email-based attacks |

## Testing

Always test in audit mode before blocking:

```powershell
# Check ASR rule status
Get-MpPreference | Select-Object AttackSurfaceReductionRules_Ids, AttackSurfaceReductionRules_Actions
```

## Summary

Implement ASR rules gradually and monitor for false positives.
