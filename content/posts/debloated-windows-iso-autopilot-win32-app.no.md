---
title: "Lag rengjorte Windows 11 ISO-er med WiFi-drivere for Autopilot"
date: 2026-02-19
draft: false
tags: ["intune", "autopilot", "powershell", "windows-11"]
categories: ["How-To"]
summary: "Bygg rene Windows 11 ISO-er med bloatware fjernet, HP WiFi-drivere injisert, og OOBE hoppet over - pluss en Win32-app for Intune-tilbakestillinger."
---

## Problemet

Vi opplevde dette gang på gang ved skoletilbakestillinger: en enhet blir slettet, Windows reinstalleres, men det er ingen WiFi. Skolekonsulenten kan ikke fullføre Autopilot-registrering uten nettverk. De må levere laptopen til IT.

Men det var ikke det eneste problemet. Selv når WiFi fungerte, kom den nyinstallerte Windows lastet med søppel - Candy Crush, TikTok, Clipchamp, personlig Teams, Xbox-apper. Alt reinstalleres under OOBE. Elevene og ansatte ser en rotete startmeny før Intune i det hele tatt får ryddet opp.

Jeg trengte en løsning som håndterer begge problemene:
1. WiFi fungerer umiddelbart etter tilbakestilling
2. Bloatware installeres aldri i utgangspunktet

## Miljø

- Windows 11 24H2
- HP ProBook bærbare (Education og Enterprise-utgaver)
- Intune standalone
- Autopilot for enhetsregistrering

## Hva jeg bygde

To verktøy som fungerer sammen:

### 1. Build-ISO.ps1 - Egendefinert ISO-bygger

Lager rengjorte Windows 11 ISO-er per utgave (Education og Enterprise) med:
- 119+ bloatware-apper fjernet på image-nivå
- HP WiFi-drivere injisert (Realtek, Intel AX211)
- Registry-justeringer (deaktiver Copilot, Widgets, telemetri)
- Autopilot-kompatibel autounattend.xml (hopper over ikke-essensielle skjermer, bevarer registreringsflyt)

### 2. Win32-app - For Intune-administrerte tilbakestillinger

Når brukere tilbakestiller via Innstillinger eller Intune sender en wipe, bruker de ikke vår egendefinerte USB. Så jeg laget en Win32-app som distribueres under ESP:
- Deteksjons-script sjekker for bloatware
- Reparasjons-script fjerner apper og bruker registry-justeringer

## Bygge den egendefinerte ISO-en

Scriptet gjør alt automatisk. Pek det mot en Windows 11 ISO og la det jobbe:

```powershell
.\Build-ISO.ps1 -SourceISO "C:\Win11_24H2.iso" -OutputFolder "C:\Output" -Edition Both

# Fullt script: https://github.com/Thugney/eriteach-scripts/blob/main/deployment/Build-ISO.ps1
```

Hva som skjer:
1. Monterer kilde-ISO
2. Ekstraherer WIM for hver utgave
3. Fjerner all bloatware fra imaget
4. Injiserer HP WiFi-drivere
5. Bruker registry-justeringer
6. Oppretter autounattend.xml
7. Bygger den endelige ISO-en

Tar ca. 30-45 minutter per utgave.

### Apper som fjernes

Scriptet fjerner 119+ apper inkludert:

**Microsoft-søppel:**
- Bing-apper (Nyheter, Vær, Finans, Sport)
- Clipchamp, Solitaire, Tips, Journal
- Copilot og CopilotRuntime
- Xbox-økosystem (GamingApp, GameOverlay, TCUI)
- Personlig Teams (MicrosoftTeams - ikke jobb MSTeams)

**Tredjeparts-søppel:**
- Netflix, Spotify, Disney+, TikTok
- Candy Crush (alle varianter)
- Facebook, Instagram, LinkedIn

**OEM-bloatware:**
- HP Support Assistant, myHP, Quick Drop
- Dell/Lenovo-ekvivalenter

**Beholdes:**
- Edge (Autopilot trenger det)
- OneDrive (vi bruker det)
- Jobb-Teams (MSTeams)
- Kalkulator, Kamera, Bilder, Utklippsverktøy

### Registry-justeringer som brukes

```powershell
# Deaktiver Copilot
TurnOffWindowsCopilot = 1

# Deaktiver Widgets
AllowNewsAndInterests = 0

# Deaktiver telemetri
AllowTelemetry = 0
DisableWindowsConsumerFeatures = 1

# Deaktiver Bing-søk
DisableWebSearch = 1
BingSearchEnabled = 0
```

Alle 40+ registry-verdier bakes inn i offline-imaget før Windows i det hele tatt starter.

## Win32-appen

For enheter som tilbakestilles via Intune (ikke USB), pakket jeg fjernings-scriptet som en Win32-app.

**Deteksjons-script** - sjekker om bloatware finnes:
```powershell
$bloatware = @("Microsoft.BingNews", "Microsoft.Copilot", "Clipchamp.Clipchamp")
foreach ($app in $bloatware) {
    if (Get-AppxPackage -Name "*$app*" -AllUsers) {
        exit 1  # Trenger reparasjon
    }
}
exit 0  # Rent

# Fullt script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Detect-Bloatware.ps1
```

**Reparasjons-script** - fjerner alt:
```powershell
# Fjern installerte pakker
Get-AppxPackage -Name "*Microsoft.BingNews*" -AllUsers | Remove-AppxPackage -AllUsers

# Fjern provisjonerte pakker (forhindrer reinstallering)
Get-AppxProvisionedPackage -Online | Where-Object { $_.DisplayName -like "*BingNews*" } |
    Remove-AppxProvisionedPackage -Online

# Bruk registry-justeringer
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsCopilot" -Name "TurnOffWindowsCopilot" -Value 1

# Fullt script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Remove-Bloatware.ps1
```

### Distribuere Win32-appen

1. Gå til **Intune-administrasjonssenter** > [**Apper**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/AppsMenu/~/windowsApps) > **Windows-apper**
2. Legg til Win32-app
3. Pakk med IntuneWinAppUtil:
   ```cmd
   IntuneWinAppUtil.exe -c "C:\Scripts" -s "Remove-Bloatware.ps1" -o "C:\Output"
   ```
4. Installasjonskommando: `powershell.exe -ExecutionPolicy Bypass -File Remove-Bloatware.ps1`
5. Deteksjon: Egendefinert script med Detect-Bloatware.ps1
6. Tildel til Alle enheter med ESP-krav

## Problemer jeg løste underveis

### WIM Dismount timeout

PowerShells `Dismount-WindowsImage` henger noen ganger på store WIM-er. Jeg la til en fallback:

```powershell
try {
    Dismount-WindowsImage -Path $mountPath -Save
}
catch {
    # Fallback til DISM
    dism.exe /Unmount-Wim /MountDir:"$mountPath" /Commit
}
```

### oscdimg.exe ikke funnet

Scriptet trenger oscdimg.exe for å lage ISO-er. Det installerer nå automatisk Windows ADK Deployment Tools hvis det mangler:

```powershell
# Laster ned ADK og installerer bare Deployment Tools-funksjonen
adksetup.exe /features OptionId.DeploymentTools /quiet
```

### Beskyttede apper feiler

Noen apper som `XboxGameCallableUI` og `PeopleExperienceHost` kan ikke fjernes (feil 0x80070032). De er systemkomponenter. Jeg kommenterte dem ut i stedet for å la hele bygget feile.

### Deteksjons-script returnerer Exit 1

Opprinnelig sjekket jeg ALLE 119 apper i deteksjon. Dette var tregt og upålitelig. Nå sjekker jeg bare 10 nøkkel-apper - hvis noen finnes, trenger enheten rengjøring:

```powershell
$KeyApps = @(
    "Microsoft.BingNews",
    "Microsoft.Copilot",
    "MicrosoftTeams",  # Personlig Teams
    "king.com.CandyCrushSaga"
)
```

### Driverinjeksjon per utgave

Noen driverpakker er utgavespesifikke. HP ProBook G10-pakken går til Education (elevenheter), WinPE-pakken går til Enterprise (ansattenheter):

```powershell
@{
    Name = "HP ProBook G10 Pack"
    Editions = @("Education")  # Kun elevenheter
}
```

### Autounattend.xml blokkerte Autopilot

Min første autounattend.xml hadde disse innstillingene som fullstendig omgikk Autopilot:

```xml
<!-- Disse innstillingene ØDELEGGER Autopilot - ikke bruk dem! -->
<HideOnlineAccountScreens>true</HideOnlineAccountScreens>
<SkipMachineOOBE>true</SkipMachineOOBE>
<SkipUserOOBE>true</SkipUserOOBE>
```

Løsningen: `HideOnlineAccountScreens` må være `false` fordi Autopilot brukerdrevet registrering trenger Entra ID-påloggingsskjermen. `SkipMachineOOBE` og `SkipUserOOBE` må fjernes helt fordi Autopilot-registrering skjer under OOBE - hopp over OOBE, hopp over Autopilot.

Det du KAN trygt hoppe over:
- `HideEULAPage` - Lisensavtale
- `HideOEMRegistrationScreen` - OEM-registrering
- `HideLocalAccountScreen` - Lokal kontoopprettelse (tvinger Entra ID)
- `ProtectYourPC` - Personverninnstillinger (Intune håndterer dette)

## Resultatet

**For USB-tilbakestillinger:**
1. Skolekonsulent starter fra vår egendefinerte USB
2. Windows installeres med fungerende WiFi
3. Ingen bloatware - ren startmeny
4. OOBE går direkte til Autopilot-registrering
5. Enhet klar samme dag

**For Intune-tilbakestillinger:**
1. Bruker tilbakestiller via Innstillinger
2. Autopilot starter ved OOBE
3. ESP distribuerer Win32-app
4. Bloatware fjernes under registrering
5. Bruker får ren enhet

Ingen mer transport av bærbare på tvers av byen. Ingen mer Candy Crush på skoleenheter.

## Scripts

- [Build-ISO.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/deployment/Build-ISO.ps1) - Lager rengjorte ISO-er
- [Detect-Bloatware.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Detect-Bloatware.ps1) - Win32-deteksjon
- [Remove-Bloatware.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/win32/Remove-Bloatware.ps1) - Win32-reparasjon

## Relaterte innlegg

- [Injiser WiFi-drivere i Windows ISO](/no/posts/wifi-driver-injection-windows-iso/) - Det frittstående driverinjeksjons-scriptet dette bygger på
