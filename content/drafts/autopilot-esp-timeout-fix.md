---
title: "Fixing Autopilot ESP Timeout During Device Enrollment"
date: 2025-01-31
draft: true
tags: ["autopilot", "intune", "troubleshooting"]
categories: ["Troubleshooting"]
summary: "How to diagnose and fix Enrollment Status Page timeouts in Windows Autopilot deployments."
---

## The Problem

Devices stuck at Enrollment Status Page (ESP) with "Identifying" or timing out after 60 minutes.

## Environment

- Windows 11 23H2
- Intune standalone
- User-driven Autopilot deployment

## Root Cause

In this case, a Win32 app with incorrect detection rules was causing ESP to wait indefinitely.

## Solution

1. Check ESP diagnostics: `Shift + F10` → `mdmdiagnosticstool.exe -area Autopilot -cab c:\temp\diag.cab`
2. Review the app install logs in the CAB
3. Found app "CompanyPortal" stuck in "Installing" state
4. Fixed detection rule: changed from file path to registry key check

## Prevention

- Always test Autopilot profiles with a subset of apps first
- Use proper detection methods for Win32 apps
- Set reasonable ESP timeout (recommend 90 mins max)

## Related Links

- [Microsoft Docs: Troubleshoot ESP](https://learn.microsoft.com/en-us/mem/intune/enrollment/troubleshoot-windows-enrollment-errors)
