---
title: "Sett opp en multi-app kiosk med en tilpasset Win32-app i Intune"
date: 2026-02-02
draft: true
tags: ["intune", "kiosk", "windows-11", "win32-app"]
categories: ["How-To"]
summary: "Konfigurer en Windows multi-app kiosk med en tilpasset Win32-applikasjon - ikke bare en nettleser."
---

## Problemet

Vi trengte en kiosk for biblioteksreservasjoner. Ikke en nettleser som peker til en URL - en faktisk Win32-app kalt Kodiak som trenger skrivertilgang.

Alle kiosk-guider jeg fant handlet om nettleser-kiosker. Dette trengte noe annet.

## Løsningen

Intunes multi-app kioskmodus med en tilpasset Startmeny-layout XML for å feste Win32-appen.

## To tilnærminger

Når du legger til apper i en multi-app kiosk har du to valg:

| Metode | Fordeler | Ulemper |
|--------|----------|---------|
| **Add Win32 App** (direkte i profilen) | Enklere oppsett, ingen XML nødvendig | Begrenset kontroll over flis-layout |
| **Custom Start Layout XML** | Full kontroll over flisplassering og størrelse | Krever AUMID og XML-kunnskap |

**Viktig:** Du kan bruke én av disse metodene, ikke begge samtidig.

Jeg valgte XML-metoden for mer kontroll over hvordan appen vises på Startmenyen.

## Forutsetninger

Før du konfigurerer kioskprofilen trenger du:

1. **Win32-appen deployet til Intune** - Pakk Kodiak (eller din app) som en Win32-app og deploy den til målenhetene
2. **Appens AUMID** - Application User Model ID, nødvendig for Startmeny-layout XML

For å finne AUMID, kjør dette på en enhet hvor appen er installert:

```powershell
Get-StartApps | Where-Object { $_.Name -like "*Kodiak*" }
```

## Opprett kioskprofilen

1. Gå til **Intune admin center** > **Devices** > [**Configuration**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/configuration)
2. Klikk **Create** > **New policy**
3. Velg:
   - Platform: **Windows 10 and later**
   - Profile type: **Templates** > **Kiosk**
4. Gi den et beskrivende navn som "Kiosk - Bibliotek"

## Konfigurer kioskinnstillinger

### Grunnleggende innstillinger

| Innstilling | Verdi |
|-------------|-------|
| Select a kiosk mode | **Multi app kiosk** |
| Target devices running Windows 10/11 in S mode | **No** |
| User logon type | **Auto logon (Windows 10, version 1803 and later, or Windows 11)** |

### Alternativ 1: Legg til Win32-app direkte

Den enkleste metoden - legg til appen rett i kioskprofilen:

1. Under **Browsers and Applications**, klikk **Add Win32 app**
2. Fyll inn app-detaljene:
   - **Name**: Kodiak (eller ditt appnavn)
   - **App type**: Win32 App
   - **Path**: Sti til .exe-filen

Appen vises automatisk på Startmenyen.

### Alternativ 2: Custom Start Layout XML (anbefalt)

For mer kontroll over flis-layout, bruk XML-metoden. Dette er hva jeg brukte.

Sett **Use alternative Start layout** til **Yes** og lim inn denne XML-en:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LayoutModificationTemplate xmlns:defaultlayout="http://schemas.microsoft.com/Start/2014/FullDefaultLayout" xmlns="http://schemas.microsoft.com/Start/2014/LayoutModification">
  <LayoutOptions StartTileGroupCellWidth="6" />
  <DefaultLayoutOverride>
    <StartLayoutCollection>
      <defaultlayout:StartLayout GroupCellWidth="6">
        <start:Group Name="Bibliotek" xmlns:start="http://schemas.microsoft.com/Start/2014/StartLayout">
          <start:DesktopApplicationTile Size="4x4" Column="0" Row="0" DesktopApplicationID="DIN-APP-AUMID-HER" />
        </start:Group>
      </defaultlayout:StartLayout>
    </StartLayoutCollection>
  </DefaultLayoutOverride>
</LayoutModificationTemplate>
```

Erstatt `DIN-APP-AUMID-HER` med appens faktiske AUMID.

### XML-innstillinger forklart

| Element | Formål |
|---------|--------|
| `StartTileGroupCellWidth="6"` | Bredde på Startmeny-flisgruppen |
| `Group Name="Bibliotek"` | Gruppenavnet som vises på Start |
| `Size="4x4"` | Stor flisstørrelse for enkel berøring |
| `DesktopApplicationID` | AUMID-en til Win32-appen din |

## Tilordne og deploy

1. Klikk **Assignments**
2. Legg til enhetsgruppen som inneholder kioskenhetene dine
3. Gjennomgå og opprett

Enheten vil restarte og gå inn i kioskmodus med auto-pålogging. Kun Startmenyen med din festede app er tilgjengelig.

## Ting å passe på

- **Velg én metode** - Du kan ikke kombinere "Add Win32 App" og custom XML i samme profil
- **AUMID må være eksakt** - Én skrivefeil og appen vises ikke på Start. Dobbeltsjekk med `Get-StartApps`
- **Appen må installeres først** - Win32-app-deploymenten må fullføres før kioskprofilen aktiveres. Bruk tilordningsfiltre eller avhengigheter
- **Skrivertilgang** - Multi-app kioskmodus tillater skrivertilgang, i motsetning til single-app modus. Derfor bruker vi multi-app selv for én applikasjon
- **Testing** - Test på én enhet først. Feilkonfigurasjon av kiosk kan låse deg ute

## Relaterte lenker

- [Windows kiosk-konfigurasjon](https://learn.microsoft.com/en-us/mem/intune/configuration/kiosk-settings-windows)
- [Start layout XML-referanse](https://learn.microsoft.com/en-us/windows/configuration/start-layout-xml-desktop)
- [Finn app AUMID](https://learn.microsoft.com/en-us/windows/configuration/find-the-application-user-model-id-of-an-installed-app)
