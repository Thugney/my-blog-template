---
title: "M365 Apps oppdaterer ikke? Slik løser du konflikter mellom oppdateringspolicyer"
date: 2026-02-17
draft: false
tags: ["intune", "how-to", "troubleshooting", "device-management"]
categories: ["Troubleshooting"]
summary: "Hvordan motstridende Intune Administrative Templates, Cloud Update-profiler og distribusjons-XML holdt 465 enheter bak på sikkerhetsoppdateringer — og hvordan du fikser det med én styringsmekanisme."
---

## Problemet

Dashboardet for sikkerhetsoppdateringer i Microsoft 365 Apps viste et bekymringsfullt bilde: **465 enheter, ingen på siste build**. 362 enheter lå to eller flere bygg bak, noe som eksponerte organisasjonen for **1 571 kjente sikkerhetssårbarheter**. Oppdateringer nådde rett og slett ikke frem til enhetene, til tross for at oppdateringsstyring var konfigurert flere steder.

Første instinkt er å sjekke frister, nettverkstilkobling, og om enhetene sjekker inn. Men rotårsaken var mer arkitektonisk enn operasjonell — vi hadde **tre konkurrerende styringsmekanismer** som alle prøvde å kontrollere M365 Apps-oppdateringer samtidig, og resultatet var at ingenting fungerte.

## Miljø

- **Intune-administrerte Windows 11-enheter** (Entra-tilknyttet, Autopilot-klargjort)
- **Microsoft 365 E3 og A5-lisenser**
- **M365 Apps for Enterprise** (64-bit, norsk språkpakke)
- **config.office.com** Cloud Update med to aktive profiler (Monthly Enterprise Channel og Current Channel)
- **Intune Administrative Template**-policy rettet mot alle enheter med oppdateringsinnstillinger aktivert

## Rotårsak

Tre lag konkurrerte om hvem som styrer M365 Apps-oppdateringer:

### Lag 1: Distribusjons-XMLen

M365 Apps ble rullet ut via Intune med en ODT-konfigurasjons-XML som brukte `Channel="Current"`. Det betyr at alle enheter installerte Office på **Current Channel** fra dag én. Men organisasjonens tiltenkte kanal var **Monthly Enterprise Channel**.

```xml
<!-- Originalt — feil kanal -->
<Add OfficeClientEdition="64" Channel="Current" MigrateArch="TRUE">
```

### Lag 2: Intune Administrative Template

En konfigurasjonsprofil kalt "Win 10 - Office Updates" var tildelt **alle enheter** med disse innstillingene aktivert:

- Target Version: **Enabled**
- Enable Automatic Updates: **Enabled**
- Update Channel: **Enabled**
- Hide option to enable or disable updates: **Enabled**
- Update Deadline: **Enabled**

Disse innstillingene skriver til registret under `HKLM:\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate`. Det kritiske problemet: når disse GPO-baserte registernøklene finnes, **kan ikke Cloud Update (config.office.com) ta styringskontrollen over enheten**. Microsoft dokumenterer dette eksplisitt — Cloud Update viker for GPO/Intune-policy registernøkler når de er til stede.

Hvis Target Version var låst til en eldre build, ville enheten aldri oppdatere forbi den. Hvis verdien var tom men Enabled, kunne det fortsatt forårsake uforutsigbar oppførsel sammen med Cloud Update.

### Lag 3: Cloud Update (config.office.com)

To oppdateringsprofiler var aktive: én for Monthly Enterprise Channel og én for Current Channel, begge rettet bredt mot enheter. I teorien skulle Cloud Update ha pushet siste bygg. I praksis var den maktesløs — Intune Administrative Template registernøklene hadde forrang, og låste enhetene effektivt ute fra Cloud Update-styring.

### Resultatet

Enhetene satt fast: installert på Current Channel via XMLen, holdt på plass av GPO-baserte registernøkler fra Admin Template, og utilgjengelige for Cloud Update. Oppdateringer fløt ikke, og antall sikkerhetssårbarheter bare økte.

## Løsning

Løsningen er å konsolidere til **én styringsmekanisme** — Cloud Update via config.office.com — og fjerne alt som konkurrerer med den.

### Steg 1: Oppdater distribusjons-XMLen

Oppdater M365 Apps Intune-distribusjonen til å bruke Monthly Enterprise Channel for alle nye enheter:

```xml
<!-- Konfigurasjons-ID genereres av config.office.com — generer din egen på https://config.office.com -->
<Configuration ID="xxx-xxx-xxx">
  <Add OfficeClientEdition="64" Channel="MonthlyEnterprise" MigrateArch="TRUE">
    <Product ID="O365ProPlusRetail">
      <Language ID="nb-no" />
      <!-- Ekskluder apper som ikke trengs i organisasjonen -->
      <ExcludeApp ID="Access" />
      <ExcludeApp ID="Groove" />   <!-- OneDrive for Business gammel synkklient -->
      <ExcludeApp ID="Lync" />     <!-- Skype for Business — utfaset -->
      <ExcludeApp ID="MSTeams" />  <!-- Gammel Teams — Nye Teams installeres  -->
    </Product>
  </Add>

  <!-- Lisensiering — alt deaktivert for standard brukerbasert lisensiering -->
  <Property Name="SharedComputerLicensing" Value="0" />
  <Property Name="SCLCacheOverride" Value="0" />
  <Property Name="AUTOACTIVATE" Value="0" />
  <Property Name="DeviceBasedLicensing" Value="0" />

  <!-- Tving lukking av Office-apper under installasjon for å unngå at den henger -->
  <Property Name="FORCEAPPSHUTDOWN" Value="TRUE" />

  <!-- Ingen <Updates>-element — Cloud Update (config.office.com) styrer oppdateringer.
       Oppdateringsinnstillinger her ville konfliktet med Cloud Update. -->

  <AppSettings>
    <Setup Name="Company" Value="Org navn" />
    <!-- Standard lagringsformat: Excel=.xlsx (51), PowerPoint=.pptx (27), Word=.docx (standard) -->
    <User Key="software\microsoft\office\16.0\excel\options"
          Name="defaultformat" Value="51" Type="REG_DWORD"
          App="excel16" Id="L_SaveExcelfilesas" />
    <User Key="software\microsoft\office\16.0\powerpoint\options"
          Name="defaultformat" Value="27" Type="REG_DWORD"
          App="ppt16" Id="L_SavePowerPointfilesas" />
    <User Key="software\microsoft\office\16.0\word\options"
          Name="defaultformat" Value="" Type="REG_SZ"
          App="word16" Id="L_SaveWordfilesas" />
  </AppSettings>

  <!-- Full installasjonsvisning med automatisk godkjenning av lisensavtale -->
  <Display Level="Full" AcceptEULA="TRUE" />
</Configuration>
```

Dette påvirker kun nye installasjoner. Eksisterende enheter trenger proactive remediation nedenfor.

### Steg 2: Slett Intune Administrative Template

Fjern "Win 10 - Office Updates" Administrative Template-policyen fra Intune. Dette er det **kritiske steget** — så lenge denne policyen finnes, vil den fortsette å skrive blokkerende registernøkler til enheter ved hver policy-synkronisering, og overstyre eventuelle remediasjonsskript.

Naviger til: **Intune Admin Center → Devices → Configuration → "Win 10 - Office Updates" → Delete**

Hvis du trenger å beholde innstillingen "Hide option to enable or disable updates", opprett en ny minimal policy med kun den innstillingen og sett alt annet til **Not Configured**.

### Steg 3: Konfigurer Cloud Update som eneste styringsmekanisme

I **config.office.com → Cloud Update**:

1. Bekreft at **Monthly Enterprise**-profilen er aktiv
2. Sett en fornuftig **Update Deadline** (f.eks. 48 eller 72 timer for sikkerhetsoppdateringer)
3. Vurder å aktivere **Rollout waves** for å rulle ut oppdateringer gradvis via pilot- og produksjonsgrupper
4. Sjekk **Current Channel**-profilen — når alle enheter migrerer til MEC, vil denne profilen ha null enheter

### Steg 4: Rull ut proactive remediation for å bytte kanal på eksisterende enheter

Eksisterende enheter som fortsatt er på Current Channel med blokkerende registernøkler trenger aktiv remediering. Distribuer dette som et Proactive Remediation-skriptpar i Intune.

**Deteksjonsskript** — sjekker CDNBaseUrl, UpdateChannel, og blokkerende GPO-registernøkler:

```powershell
# Monthly Enterprise Channel CDNBaseUrl
$targetCDNBaseUrl = "http://officecdn.microsoft.com/pr/55336b82-a18d-4dd6-b5f6-9e5095c314a6"
$c2rRegPath = "HKLM:\SOFTWARE\Microsoft\Office\ClickToRun\Configuration"
$policiesRegPath = "HKLM:\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate"

# Sjekk CDNBaseUrl (effektiv kanal)
$currentCDN = (Get-ItemProperty -Path $c2rRegPath -Name "CDNBaseUrl" -ErrorAction SilentlyContinue).CDNBaseUrl
if ($currentCDN -ne $targetCDNBaseUrl) { $nonCompliant = $true }

# Sjekk for blokkerende policy-nøkler som hindrer Cloud Update
foreach ($key in @("updatebranch", "updatepath", "updatetargetversion")) {
    $value = (Get-ItemProperty -Path $policiesRegPath -Name $key -ErrorAction SilentlyContinue).$key
    if ($null -ne $value -and $value -ne "") { $nonCompliant = $true }
}

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/m365apps-channel-switch-detection.ps1
```

**Remediasjonsskript** — fjerner blokkerende nøkler, setter riktig kanal, starter oppdateringssjekk og verifiserer:

```powershell
# Fjern blokkerende policy-nøkler
foreach ($key in @("updatebranch", "updatepath", "updatetargetversion",
                   "enableautomaticupdates", "updatedeadline")) {
    Remove-ItemProperty -Path $policiesRegPath -Name $key -Force -ErrorAction SilentlyContinue
}

# Sett CDNBaseUrl og UpdateChannel til Monthly Enterprise Channel
Set-ItemProperty -Path $c2rRegPath -Name "CDNBaseUrl" -Value $targetCDNBaseUrl -Force
Set-ItemProperty -Path $c2rRegPath -Name "UpdateChannel" -Value $targetCDNBaseUrl -Force

# Start Office-oppdateringssjekk
$c2rExe = Join-Path $env:CommonProgramFiles "Microsoft Shared\ClickToRun\OfficeC2RClient.exe"
Start-Process -FilePath $c2rExe -ArgumentList "/update user displaylevel=false forceappshutdown=false" -Wait -NoNewWindow

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/m365apps-channel-switch-remediation.ps1
```

Distribuer i Intune: **Devices → [Remediations](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents) → Create script package**, planlegg daglig, kjør som System (64-bit).

### Steg 5: Overvåk

Etter utrulling, overvåk fra to steder:

- **config.office.com → Cloud Update → Monthly Enterprise → Overview** — se at enheter dukker opp under MEC-styring
- **Intune Admin Center → Reports → Microsoft 365 Apps updates** — se at sårbarhetstallet synker

Gi det **48–72 timer** for at hele enhetsflåten skal bytte kanal og hente siste MEC-build.

## Forebygging / Beste praksis

Hovedlærdommen: **aldri ha mer enn én styringsmekanisme for M365 Apps-oppdateringer**.

Velg én av disse og bruk den eksklusivt:

| Styringsmekanisme | Passer best for | Hvordan den styrer oppdateringer |
|---|---|---|
| **Cloud Update (config.office.com)** | De fleste organisasjoner | Tjenestebasert styring, rollout waves, fristbasert håndhevelse |
| **Intune Admin Template** | Organisasjoner som trenger granulær GPO-kontroll | Registerbasert policy, fastlåste versjoner, kanalkontroll |
| **Oppdateringsprofil via Intune** | Intune-først-styring | M365 Apps-oppdateringspolicyer i Intune-portalen |

Hvis du bruker Cloud Update, **ikke** sett Update Channel, Target Version eller Update Path via Intune Administrative Templates eller GPO. Disse registernøklene blokkerer Cloud Update fra å styre enheten.

Distribusjons-XMLens kanal bør også samsvare med den tiltenkte oppdateringskanalen, slik at nye enheter ikke trenger kanalbytte etter installasjon. Og kritisk viktig — ikke inkluder et `<Updates>`-element i XMLen hvis Cloud Update er din styringsmekanisme.

## Skript

- [m365apps-channel-switch-detection.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/m365apps-channel-switch-detection.ps1)
- [m365apps-channel-switch-remediation.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/m365apps-channel-switch-remediation.ps1)

## Relaterte lenker

- [Oversikt over Cloud Update - Microsoft Learn](https://learn.microsoft.com/nb-no/microsoft-365-apps/admin-center/cloud-update)
- [Endre oppdateringskanalen for Microsoft 365 Apps](https://learn.microsoft.com/nb-no/microsoft-365-apps/updates/change-update-channels)
- [Oppdateringskanaler for Microsoft 365 Apps - Alternativer](https://learn.microsoft.com/nb-no/microsoft-365-apps/updates/overview-update-channels)
- [Proactive remediations i Intune](https://learn.microsoft.com/nb-no/mem/intune/fundamentals/remediations)
