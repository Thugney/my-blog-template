---
title: "Eksempel: Konfigurere Microsoft Defender Attack Surface Reduction-regler"
date: 2026-01-10
draft: false
tags: ["defender", "security", "how-to"]
categories: ["How-To"]
summary: "En guide til implementering av Attack Surface Reduction (ASR)-regler via Intune for forbedret endepunktsikkerhet."
description: "Lær hvordan du konfigurerer og distribuerer Microsoft Defender Attack Surface Reduction-regler gjennom Intune for å beskytte organisasjonen din."
keywords: ["defender", "ASR", "attack surface reduction", "intune", "sikkerhet"]
---

Dette er et eksempelinnlegg som viser Defender-tag coverfargen. Erstatt med ditt innhold.

## Introduksjon

Attack Surface Reduction-regler bidrar til å forhindre handlinger som skadelig programvare ofte misbruker.

## Forutsetninger

- Microsoft Defender for Endpoint-lisens
- Intune-administratortilgang
- Windows 10/11-enheter

## Konfigurasjonstrinn

1. Naviger til **Intune** > **Endepunktsikkerhet** > **Attack Surface Reduction**
2. Opprett en ny policy
3. Konfigurer de ønskede reglene

## Vanlige ASR-regler

| Regel | Beskrivelse |
|-------|-------------|
| Blokker Office-apper fra å opprette barneprosesser | Forhindrer Office fra å spawne prosesser |
| Blokker legitimasjonstyveri fra LSASS | Beskytter legitimasjon |
| Blokker kjørbart innhold fra e-post | Forhindrer e-postbaserte angrep |

## Testing

Test alltid i revisjonsmodus før blokkering:

```powershell
# Sjekk ASR-regelstatus
Get-MpPreference | Select-Object AttackSurfaceReductionRules_Ids, AttackSurfaceReductionRules_Actions
```

## Oppsummering

Implementer ASR-regler gradvis og overvåk for falske positiver.
