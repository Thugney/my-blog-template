---
name: eriteach-blog
description: Write technical blog posts for eriteach.com about Microsoft cloud technologies (Intune, Autopilot, Entra ID, Defender, Purview). Use when the user describes a scenario, problem, or solution they encountered and wants a blog post created. Triggers on requests like "write a blog post about...", "create a post for...", "document this scenario...", or when user shares a technical problem they solved.
---

# Eriteach Blog Post Skill

Write practical, scenario-based blog posts for blog.eriteach.com.

## Author Context

**Robel Mehari** - Microsoft 365 Security Specialist at Modum Kommune, Norway
- Manages 4,000+ users and 6,000+ endpoints
- Originally from Eritrea, started IT in 2009
- Worked as social worker with refugee youth before returning to IT
- IT apprentice 2021-2024, now full-time M365 security
- Runs Eriteach YouTube channel (20,000+ subscribers)
- Works in kommune (municipality) environment with multiple schools



## Voice & Style

- **First person** - Use "I" and "we" - these are the author's real experiences and solutions. "I ran into this problem", "We had devices that...", "Then it hit me..."
- **Simple language** - No jargon. If you must use acronyms, explain them in plain words
- **Short sentences** - Get to the point. No fluff
- **Real-world framing** - "We kept running into this issue..." not "When implementing device enrollment strategies..."
- **Not verbose** - Say it once, say it clearly, move on
- **No guru tone** - No "Let's dive in", "In this blog post we will", "As you may know"
- **Norwegian context** - Municipality environment, multiple schools, student devices, school consultants handle local IT
- **Use Deeplink** - Use deeplink when possible and alsways add microsoft related topics, confirm the link/blog from microsoft actually is related to what is about to be a blog post.
- **Security** - When writing post never expose critical info given by user for example purposes alsways clarify

## Blog Focus

Microsoft cloud tech:
- **Intune** - Device management, app deployment, compliance
- **Autopilot** - Device enrollment, ESP, deployment profiles
- **Entra ID** - Identity, conditional access, authentication
- **Defender** - Security, threat protection
- **Purview** - Data governance, sensitivity labels
- **Automation** - Proactive remediation, Poewrautomate

## Post Structure

```markdown
---
title: "Clear Title - What This Fixes or Does"
date: YYYY-MM-DD
draft: true
tags: ["primary-tech", "secondary-tech"]
categories: ["Troubleshooting" | "How-To" | "Quick-Tip"]
summary: "One sentence - what you'll learn"
---

## The Problem

What went wrong. Be specific. Use from users input. Once the user describe  the issue and the solutions, you write the blog as if the author is writing them.

Example: "A user gets a new laptop. Autopilot starts, then sits at 'Identifying' for 45 minutes before timing out."

## Environment

- Windows 11 24H2
- Intune standalone
- Hybrid Entra joined

Keep it short. Only what's relevant.

## What I Checked

What you looked at to find the cause. Be specific about where you clicked, what logs you pulled.

Example:
1. Opened Intune portal → Devices → Windows → Windows enrollment
2. Checked the device in Autopilot devices - status showed "Assigned"
3. Ran `mdmdiagnosticstool.exe -area Autopilot -cab c:\temp\diag.cab`
4. Found error in logs: "App XYZ stuck in installing state"

## The Fix

Step-by-step. Number each step. Include exact paths in Intune/Entra.

Example:
1. Go to **Intune** → **Apps** → **Windows apps**
2. Find the app "Company Portal"
3. Click **Properties** → **Detection rules**
4. Change from file path to registry key:
   - Path: `HKLM\SOFTWARE\Microsoft\CompanyPortal`
   - Key: `Installed`
   - Value: `1`
5. Save and sync

## Script (if applicable)

For scripts, show only a snippet that explains the logic. Full script lives in GitHub.

```powershell
# Check if device is Autopilot registered
$serial = (Get-WmiObject win32_bios).SerialNumber
# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/autopilot/check-registration.ps1
```

Always link to full script in GitHub repo: `github.com/Thugney/eriteach-scripts`

## What to Watch Out For

Optional section. Include if there are common mistakes or things that broke along the way.

## Related Links

- Link to Microsoft docs if helpful
- Link to related internal posts (cross-linking improves SEO and helps readers discover content)
- After publishing a new post, check existing related posts and add links to the new post
```

## Code Block Rules

ALWAYS use proper markdown code fences with language specified:

```powershell
Get-MgDevice -Filter "displayName eq 'PC001'"
<#
.SYNOPSIS
Avinstallerer Kodiak selvbetjeningsprogram.

.DESCRIPTION
Fjerner C:\Kodiak-katalogen, snarveier i Startmeny og skrivebordet.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System
#>
```

```json
{
  "setting": "value"
}
```

Never paste raw code without fences. Never use inline code for multi-line scripts.

## GitHub Scripts Repository

Full scripts live in: `https://github.com/Thugney/eriteach-scripts`

Local clone: `J:\Projects\eriteach-scripts-temp\`

### Folder Structure

```
eriteach-scripts/
├── intune/
│   ├── remediations/     # Proactive remediation scripts (detection + remediation pairs)
│   └── win32/            # Win32 app scripts (detection + install pairs)
├── deployment/           # OS deployment and imaging scripts
├── autopilot/            # Autopilot-related scripts
└── graph/                # Microsoft Graph API scripts
```

### Current Scripts in Repo

When adding new scripts, update the README tables to match this format (with clickable links):

**Intune Remediations** (`intune/remediations/`):
| Script | Purpose |
|--------|---------|
| [`script-name-detection.ps1`](intune/remediations/script-name-detection.ps1) | Description |
| [`script-name-remediation.ps1`](intune/remediations/script-name-remediation.ps1) | Description |

**Intune Win32 Apps** (`intune/win32/`):
| Script | Purpose |
|--------|---------|
| [`Detect-Feature.ps1`](intune/win32/Detect-Feature.ps1) | Win32 detection - description |
| [`Install-Feature.ps1`](intune/win32/Install-Feature.ps1) | Win32 install - description |

**Deployment** (`deployment/`):
| Script | Purpose |
|--------|---------|
| [`script-name.ps1`](deployment/script-name.ps1) | Description |

### Adding Scripts to GitHub

When a blog post includes scripts, push the full versions to GitHub.

**Steps:**
1. Create the script file in the appropriate folder with proper header
2. Update `README.md` — add the new script to the correct table with clickable link
3. Commit with descriptive message
4. Push directly — do NOT ask unnecessary questions

**If repo is NOT cloned locally:**
```bash
git clone https://github.com/Thugney/eriteach-scripts.git "J:\Projects\eriteach-scripts-temp"
```

### Script Header Format

**REQUIRED:** Every script MUST have this exact banner header followed by the help block:

```powershell
# ============================================================================
# Eriteach Scripts
# Author: Robel (https://github.com/Thugney)
# Repository: https://github.com/Thugney/eriteach-scripts
# License: MIT
# ============================================================================

<#
.SYNOPSIS
One line - what it does.

.DESCRIPTION
How it works. Key logic explained.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System | User
#>
```

### Linking in Blog Posts

In blog posts, show ONLY a short snippet with key logic, then link to the full script on GitHub. NEVER include full scripts inline in blog posts.

```powershell
# Key logic here (condensed)
$response = Invoke-RestMethod -Uri "https://api.example.com"

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/folder/script-name.ps1
```

### README Format

The repo README follows this structure:
- Badges at top (GitHub, Blog, YouTube, LinkedIn, License)
- About section with author context
- Structure tree
- Scripts tables with clickable links per category
- Usage section showing script header format
- Related links and License

## Intune Deep Links

When referencing Intune admin center locations, use deep links to help readers navigate directly:

| Location | Deep Link |
|----------|-----------|
| Scripts and remediations | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents` |
| Devices overview | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/overview` |
| Windows enrollment | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesWindowsMenu/~/enrollment` |
| Configuration profiles | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/configuration` |
| Compliance policies | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesComplianceMenu/~/policies` |
| Apps | `https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/AppsMenu/~/windowsApps` |

Example usage in markdown:
```markdown
1. Go to **Intune admin center** > **Devices** > [**Scripts and remediations**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents)
```

## Internal Linking Best Practices

**Cross-link related posts** - Internal linking helps readers discover more content and improves SEO.

When creating a new post:
1. Add links to related existing posts in the "Related Links" section
2. Check existing posts that cover related topics and add links back to the new post
3. Use descriptive link text that explains what the linked post covers

Example: If writing about "Remove Firefox", link to "Auto-Update Firefox" post and vice versa.

## Length Guide

- **Quick-Tip**: 200-400 words. One problem, one fix.
- **How-To**: 400-700 words. Step-by-step guide.
- **Troubleshooting**: 500-900 words. Problem → investigation → fix.

Shorter is better. If it can be said in 300 words, don't write 600.

## Tags

`intune`, `autopilot`, `entra-id`, `defender`, `purview`, `powershell`, `graph-api`, `conditional-access`, `compliance`, `windows-11`, `hybrid-join`

## Categories

- **Troubleshooting** - Something broke, here's why and how to fix
- **How-To** - Step-by-step to set something up
- **Quick-Tip** - Fast, focused, under 400 words

## File Output

Save to: `J:\Projects\eriteach-blog\content\drafts\`

Filename: `kebab-case-describing-topic.md`

Example: `autopilot-esp-timeout-win32-detection.md`

**IMPORTANT: Always create both English AND Norwegian versions:**
- `my-post.en.md` - English version
- `my-post.no.md` - Norwegian version

Both files go in the same folder. Hugo links them automatically by filename prefix.

**Tag ordering matters** — put the most relevant technology tag FIRST, as it determines the CSS cover gradient color (e.g., `tags: ["intune", "powershell"]` gives an Intune blue-teal cover).

## Workflow

1. User describes scenario
2. Ask clarifying questions if needed:
   - What error did you see?
   - What Windows/Intune version?
   - What did you try first?
3. Write post in user's voice (simple, direct, no fluff)
4. Save to drafts folder
5. If post includes scripts:
   - Clone eriteach-scripts repo to temp
   - Create full script files with proper headers
   - Update repo README with new scripts
   - Commit and push to GitHub
   - Ensure blog post links to the GitHub scripts

## Hosting & Deployment

The blog is hosted on **Netlify** (free tier), NOT GitHub Pages.

- **Repo**: `https://github.com/Thugney/eriteach-blog.git` (PRIVATE repo)
- **Netlify config**: `netlify.toml` in repo root
- **Auto-deploy**: Every push to `main` triggers a Netlify build
- **Custom domain**: `blog.eriteach.com`
- **Build command**: `hugo --gc --minify`
- **Hugo version**: 0.155.1 (extended)

### Publishing Workflow

1. Write post in `content/drafts/` with `draft: true`
2. When ready to publish, move to `content/posts/` and set `draft: false`
3. Commit and push to `main` — Netlify auto-deploys within ~1 minute
4. NEVER use GitHub Pages or GitHub Actions for deployment

## Design System

### Theme & Colors

- **Theme**: PaperMod (git submodule at `themes/papermod/`)
- **Color scheme**: Teal/modern
  - Light mode accent: `#0d9488`
  - Dark mode accent: `#2dd4bf`
  - Theme follows OS preference (`defaultTheme = "auto"`)

### Custom CSS Location

**IMPORTANT**: Custom CSS lives at `assets/css/extended/custom.css` (site root), NOT inside the theme directory. Hugo site-root assets take priority over theme assets.

NEVER put custom CSS in `themes/papermod/assets/css/extended/` — it will be lost on theme update.

### Homepage

Uses **Profile Mode** with bilingual config:
- Profile photo: `static/images/robel-mehari.jpg`
- Social icons: LinkedIn, X, GitHub, YouTube, RSS (configured via `[[params.socialIcons]]`)
- Buttons: language-specific (EN: Posts/About, NO: Innlegg/Om)
- ProfileMode is defined per-language in `hugo.toml` under `[languages.en.params.profileMode]` and `[languages.no.params.profileMode]`

### Cover Images

Posts use **CSS-generated gradient covers** based on the first tag — no image files needed.

The cover partial at `layouts/partials/cover.html` maps the first tag to a gradient class:

| First Tag | Gradient | Class |
|-----------|----------|-------|
| intune | Blue to teal | `cover-intune` |
| defender | Red | `cover-defender` |
| entra-id | Purple to blue | `cover-entra-id` |
| autopilot | Green to teal | `cover-autopilot` |
| purview | Deep purple | `cover-purview` |
| windows-11 | Slate blue | `cover-windows-11` |
| kiosk | Dark teal | `cover-kiosk` |
| powershell | Dark blue | `cover-powershell` |
| (anything else) | Teal brand | `cover-default` |

**Order your tags deliberately** — the first tag determines the cover color.

If a post has `cover.image` in front matter, the real image takes priority over CSS covers.

### Footer

The footer (`layouts/partials/footer.html`) shows copyright + PaperMod's SVG social icons from the `socialIcons` config.

### Table of Contents

Enabled globally (`ShowToc = true`, `TocOpen = false`). Can be disabled per-post with `ShowToc: false` in front matter.

### Layout Overrides

Only these layout files override PaperMod defaults:
- `layouts/partials/footer.html` — custom footer with social icons
- `layouts/partials/cover.html` — CSS-generated covers fallback

Do NOT create unnecessary layout overrides. Use hugo.toml config when possible.

### OG Image

Default social sharing image: `static/og-default.png` (1200x630, teal gradient with "Eriteach" branding).

## Example - What NOT to Write

❌ "In this comprehensive guide, we will explore the intricacies of Windows Autopilot Enrollment Status Page timeout issues and dive deep into the resolution methodology."

❌ "The school consultant has to physically deliver the laptop to the IT office." (third person, detached)

## Example - What TO Write

✅ "We kept running into this: a student downloads something sketchy, Defender flags the device. Now it needs a reset."

✅ "Then it hit me - what if I inject the drivers directly into the ISO?"

✅ "Our school consultants can now handle this themselves. No more delivering laptops to IT."
