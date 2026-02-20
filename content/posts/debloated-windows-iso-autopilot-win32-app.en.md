---
title: "Create Debloated Windows 11 ISOs with WiFi Drivers for Autopilot"
date: 2026-02-19
draft: false
tags: ["intune", "autopilot", "powershell", "windows-11", "deployment", "bloatware-removal"]
categories: ["How-To"]
summary: "Build clean Windows 11 ISOs with bloatware removed, HP WiFi drivers injected, and OOBE skipped - plus a Win32 app for Intune resets."
description: "Step-by-step guide to creating debloated Windows 11 ISOs with 119+ apps removed, HP WiFi drivers injected, and registry tweaks applied. Includes Win32 app for Intune-managed device resets."
keywords: ["debloated Windows 11", "Windows ISO customization", "remove Windows bloatware", "HP WiFi drivers", "Autopilot deployment", "Intune Win32 app", "DISM image optimization", "Windows 11 Education", "enterprise deployment"]
---

## The Problem

We kept running into this at school resets: a device gets wiped, Windows reinstalls, but there's no WiFi. The school consultant can't complete Autopilot enrollment without network. They have to bring the laptop to IT.

But that wasn't the only issue. Even when WiFi worked, the freshly installed Windows came loaded with junk - Candy Crush, TikTok, Clipchamp, personal Teams, Xbox apps. All of it reinstalls during OOBE. Our students and staff see a cluttered Start menu before Intune even gets a chance to clean it up.

I needed a solution that handles both problems:
1. WiFi works immediately after reset
2. Bloatware never installs in the first place

## Environment

- Windows 11 24H2
- HP ProBook laptops (Education and Enterprise editions)
- Intune standalone
- Autopilot for device enrollment

## What I Built

Two tools that work together:

### 1. Build-ISO.ps1 - Custom ISO Creator

Creates debloated Windows 11 ISOs per edition (Education and Enterprise) with:
- 119+ bloatware apps removed at image level
- HP WiFi drivers injected (Realtek, Intel AX211)
- Registry tweaks applied (disable Copilot, Widgets, telemetry)
- Autopilot-compatible autounattend.xml (skips non-essential screens, preserves enrollment flow)

### 2. Win32 App - For Intune-Managed Resets

When users reset via Settings or Intune sends a wipe, they don't use our custom USB. So I created a Win32 app that deploys during ESP:
- Detection script checks for bloatware
- Remediation script removes apps and applies registry tweaks

## Building the Custom ISO

The script does everything automatically. Point it at a Windows 11 ISO and let it work:

```powershell
.\Build-ISO.ps1 -SourceISO "C:\Win11_24H2.iso" -OutputFolder "C:\Output" -Edition Both

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/deployment/Build-ISO.ps1
```

What happens:
1. Mounts the source ISO
2. Extracts the WIM for each edition
3. Removes all bloatware from the image
4. Injects HP WiFi drivers
5. Applies registry tweaks
6. Creates autounattend.xml
7. Builds the final ISO

Takes about 30-45 minutes per edition.

### Apps Removed

The script removes 119+ apps including:

**Microsoft bloat:**
- Bing apps (News, Weather, Finance, Sports)
- Clipchamp, Solitaire, Tips, Journal
- Copilot and CopilotRuntime
- Xbox ecosystem (GamingApp, GameOverlay, TCUI)
- Personal Teams (MicrosoftTeams - not work MSTeams)

**Third-party junk:**
- Netflix, Spotify, Disney+, TikTok
- Candy Crush (all variants)
- Facebook, Instagram, LinkedIn

**OEM bloat:**
- HP Support Assistant, myHP, Quick Drop
- Dell/Lenovo equivalents

**Kept:**
- Edge (Autopilot needs it)
- OneDrive (we use it)
- Work Teams (MSTeams)
- Calculator, Camera, Photos, Snipping Tool

### Registry Tweaks Applied

```powershell
# Disable Copilot
TurnOffWindowsCopilot = 1

# Disable Widgets
AllowNewsAndInterests = 0

# Disable telemetry
AllowTelemetry = 0
DisableWindowsConsumerFeatures = 1

# Disable Bing search
DisableWebSearch = 1
BingSearchEnabled = 0
```

All 40+ registry values are baked into the offline image before Windows even boots.

## The Win32 App

For devices reset through Intune (not USB), I packaged the removal script as a Win32 app.

**Detection script** - checks if bloatware exists:
```powershell
$bloatware = @("Microsoft.BingNews", "Microsoft.Copilot", "Clipchamp.Clipchamp")
foreach ($app in $bloatware) {
    if (Get-AppxPackage -Name "*$app*" -AllUsers) {
        exit 1  # Needs remediation
    }
}
exit 0  # Clean

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Detect-Bloatware.ps1
```

**Remediation script** - removes everything:
```powershell
# Remove installed packages
Get-AppxPackage -Name "*Microsoft.BingNews*" -AllUsers | Remove-AppxPackage -AllUsers

# Remove provisioned packages (prevents reinstall)
Get-AppxProvisionedPackage -Online | Where-Object { $_.DisplayName -like "*BingNews*" } |
    Remove-AppxProvisionedPackage -Online

# Apply registry tweaks
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsCopilot" -Name "TurnOffWindowsCopilot" -Value 1

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Remove-Bloatware.ps1
```

### Deploying the Win32 App

1. Go to **Intune admin center** > [**Apps**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/AppsMenu/~/windowsApps) > **Windows apps**
2. Add Win32 app
3. Package with IntuneWinAppUtil:
   ```cmd
   IntuneWinAppUtil.exe -c "C:\Scripts" -s "Remove-Bloatware.ps1" -o "C:\Output"
   ```
4. Install command: `powershell.exe -ExecutionPolicy Bypass -File Remove-Bloatware.ps1`
5. Detection: Custom script using Detect-Bloatware.ps1
6. Assign to All Devices with ESP requirement

## Issues I Fixed Along the Way

### WIM Dismount Timeout

PowerShell's `Dismount-WindowsImage` sometimes hangs on large WIMs. I added a fallback:

```powershell
try {
    Dismount-WindowsImage -Path $mountPath -Save
}
catch {
    # Fallback to DISM
    dism.exe /Unmount-Wim /MountDir:"$mountPath" /Commit
}
```

### oscdimg.exe Not Found

The script needs oscdimg.exe to create ISOs. It now auto-installs Windows ADK Deployment Tools if missing:

```powershell
# Downloads ADK and installs only Deployment Tools feature
adksetup.exe /features OptionId.DeploymentTools /quiet
```

### Protected Apps Failing

Some apps like `XboxGameCallableUI` and `PeopleExperienceHost` can't be removed (error 0x80070032). They're system components. I commented them out rather than fail the whole build.

### Detection Script Returning Exit 1

Initially I checked ALL 119 apps in detection. This was slow and unreliable. Now I check only 10 key apps - if any exist, the device needs cleaning:

```powershell
$KeyApps = @(
    "Microsoft.BingNews",
    "Microsoft.Copilot",
    "MicrosoftTeams",  # Personal Teams
    "king.com.CandyCrushSaga"
)
```

### Driver Injection Per Edition

Some driver packs are edition-specific. HP ProBook G10 pack goes to Education (student devices), WinPE pack goes to Enterprise (staff devices):

```powershell
@{
    Name = "HP ProBook G10 Pack"
    Editions = @("Education")  # Student devices only
}
```

### Autounattend.xml Blocking Autopilot

My initial autounattend.xml had these settings that completely bypassed Autopilot:

```xml
<!-- These settings BREAK Autopilot - don't use them! -->
<HideOnlineAccountScreens>true</HideOnlineAccountScreens>
<SkipMachineOOBE>true</SkipMachineOOBE>
<SkipUserOOBE>true</SkipUserOOBE>
```

The fix: `HideOnlineAccountScreens` must be `false` because Autopilot user-driven enrollment needs the Entra ID sign-in screen. `SkipMachineOOBE` and `SkipUserOOBE` must be removed entirely because Autopilot enrollment happens during OOBE - skip OOBE, skip Autopilot.

What you CAN safely skip:
- `HideEULAPage` - License agreement
- `HideOEMRegistrationScreen` - OEM registration
- `HideLocalAccountScreen` - Local account creation (forces Entra ID)
- `ProtectYourPC` - Privacy settings (Intune handles this)

## The Result

**For USB resets:**
1. School consultant boots from our custom USB
2. Windows installs with WiFi working
3. No bloatware - clean Start menu
4. OOBE flows directly into Autopilot enrollment
5. Device ready same day

**For Intune resets:**
1. User resets via Settings
2. Autopilot triggers at OOBE
3. ESP deploys Win32 app
4. Bloatware removed during enrollment
5. User gets clean device

No more hauling laptops across town. No more Candy Crush on school devices.

## Scripts

- [Build-ISO.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/deployment/Build-ISO.ps1) - Creates debloated ISOs
- [Detect-Bloatware.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Detect-Bloatware.ps1) - Win32 detection
- [Remove-Bloatware.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Remove-Bloatware.ps1) - Win32 remediation

## Related Posts

- [Inject WiFi Drivers into Windows ISO](/posts/wifi-driver-injection-windows-iso/) - The standalone driver injection script this builds on
