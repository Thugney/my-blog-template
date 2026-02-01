---
title: "Setting Up a Phishing Simulation with Defender Attack Simulation Training"
date: 2026-01-31
draft: false
tags: ["defender", "security", "how-to", "attack-simulation"]
categories: ["How-To"]
summary: "How to create and deploy a realistic SharePoint phishing simulation using Microsoft Defender for Office 365 Attack Simulation Training."
---

## The Scenario

Security awareness training works best when users experience realistic phishing attempts in a safe environment. Microsoft Defender for Office 365 includes Attack Simulation Training that lets you send simulated phishing emails and track who clicks.

For this simulation, I created a SharePoint "mass file deletion" notification - a template that mimics legitimate Microsoft emails and creates urgency without being overly alarming.

## Environment

- Microsoft 365 E5 (or Defender for Office 365 Plan 2)
- Exchange Online
- Attack Simulation Training enabled

## Creating the Email Template

The most effective phishing simulations closely match legitimate emails. The SharePoint deletion notification works well because:

- **Urgency** - "permanently removed", "93 days" countdown
- **Familiarity** - Users receive real SharePoint notifications regularly
- **Multiple click targets** - Links in body text and CTA button

### Template Design Elements

The template included:
- Microsoft branding (logo, Fluent Design colors)
- Dark theme matching current SharePoint notifications
- Animated floating file icons for visual appeal
- Three phishing link placements:
  - "recycle bin" text link
  - "deleting and restoring files" help link
  - Blue "Recycle bin" CTA button

### The HTML Template

Below is the full HTML template used for this simulation. The template includes detailed comments explaining configuration options and indicators of compromise for training purposes.

<details>
<summary>Click to expand full HTML template</summary>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SharePoint - Files Deleted Notification</title>
    <!--
    ============================================================
    PHISHING SIMULATION TEMPLATE - FOR SECURITY AWARENESS TESTING
    ============================================================

    Template: SharePoint Mass File Deletion Notification
    Target: IT Department (Pilot) -> Organization-wide
    Platform: Microsoft Defender for Office 365 - Attack Simulation

    CONFIGURATION NOTES FOR DEFENDER:
    - Payload Type: Credential Harvest or Link in Attachment
    - Landing Page: Microsoft Sign-In (built-in or custom)
    - Tracking: Link click tracking via {{PhishingURL}}

    PERSONALIZATION TOKENS (Defender):
    - {{DisplayName}} - User's display name
    - {{FirstName}} - User's first name
    - {{LastName}} - User's last name
    - {{Email}} - User's email address
    - {{PhishingURL}} - Tracked phishing link

    INDICATORS OF COMPROMISE (for training):
    1. Sender address (will show simulation domain)
    2. Urgency language ("permanently removed", "93 days")
    3. Generic greeting vs personalized
    4. Hover over links to see actual URL
    5. Unexpected notification about file deletion
    ============================================================
    -->
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', 'Segoe UI Web (West European)', -apple-system, BlinkMacSystemFont, Roboto, 'Helvetica Neue', sans-serif;
            background-color: #1b1b1b;
            color: #d4d4d4;
            line-height: 1.5;
            -webkit-font-smoothing: antialiased;
        }

        .email-wrapper {
            max-width: 680px;
            margin: 0 auto;
            background-color: #1b1b1b;
        }

        .header {
            text-align: center;
            padding: 24px 0;
            background-color: #1b1b1b;
        }

        .microsoft-logo {
            display: inline-flex;
            align-items: center;
            gap: 8px;
        }

        .microsoft-logo svg {
            width: 20px;
            height: 20px;
        }

        .microsoft-logo span {
            font-size: 18px;
            font-weight: 600;
            color: #ffffff;
            letter-spacing: -0.3px;
        }

        .hero-section {
            background: linear-gradient(180deg, #f8e8dc 0%, #f5dfd0 100%);
            padding: 30px 40px;
            position: relative;
            overflow: hidden;
            min-height: 180px;
        }

        .hero-content {
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
            position: relative;
            z-index: 2;
        }

        .recycle-bin {
            position: relative;
            width: 200px;
        }

        .bin-container {
            width: 140px;
            height: 110px;
            background: linear-gradient(135deg, #0078d4 0%, #005a9e 100%);
            border-radius: 8px;
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-left: 20px;
        }

        .recycle-symbol {
            width: 50px;
            height: 50px;
            color: white;
        }

        .floating-items {
            position: absolute;
            top: -50px;
            left: 0;
            width: 180px;
            height: 80px;
        }

        .float-item {
            position: absolute;
            font-size: 24px;
            animation: float 3s ease-in-out infinite;
        }

        .float-item:nth-child(1) { left: 20px; top: 10px; animation-delay: 0s; }
        .float-item:nth-child(2) { left: 60px; top: 0; animation-delay: 0.3s; }
        .float-item:nth-child(3) { left: 100px; top: 15px; animation-delay: 0.6s; }
        .float-item:nth-child(4) { left: 140px; top: 5px; animation-delay: 0.9s; }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-8px); }
        }

        .cat-section {
            position: relative;
        }

        .cat-svg {
            width: 100px;
            height: auto;
        }

        .content-section {
            padding: 40px 48px;
            background-color: #1b1b1b;
        }

        .main-heading {
            font-size: 28px;
            font-weight: 600;
            color: #ffffff;
            line-height: 1.3;
            margin-bottom: 24px;
        }

        .greeting {
            font-size: 16px;
            color: #d4d4d4;
            margin-bottom: 20px;
        }

        .body-text {
            font-size: 15px;
            color: #b3b3b3;
            margin-bottom: 20px;
            line-height: 1.6;
        }

        .highlight-text {
            font-size: 15px;
            color: #ffffff;
            font-weight: 500;
            margin-bottom: 20px;
            line-height: 1.6;
        }

        .highlight-text a {
            color: #4da3ff;
            text-decoration: none;
        }

        .highlight-text a:hover {
            text-decoration: underline;
        }

        .muted-text {
            font-size: 14px;
            color: #8a8a8a;
            margin-bottom: 16px;
        }

        .learn-more {
            font-size: 14px;
            color: #8a8a8a;
            margin-bottom: 32px;
        }

        .learn-more a {
            color: #4da3ff;
            text-decoration: none;
        }

        .cta-button {
            display: inline-block;
            background-color: #0078d4;
            color: #ffffff;
            font-size: 15px;
            font-weight: 600;
            padding: 12px 28px;
            border-radius: 4px;
            text-decoration: none;
            border: none;
            cursor: pointer;
            transition: background-color 0.2s ease;
        }

        .cta-button:hover {
            background-color: #106ebe;
        }

        .footer-section {
            padding: 32px 48px;
            background-color: #262626;
            border-top: 1px solid #3d3d3d;
        }

        .footer-text {
            font-size: 12px;
            color: #8a8a8a;
            margin-bottom: 20px;
        }

        .footer-divider {
            height: 1px;
            background-color: #3d3d3d;
            margin: 20px 0;
        }

        .footer-logo {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            margin-bottom: 12px;
        }

        .footer-logo svg {
            width: 16px;
            height: 16px;
        }

        .footer-logo span {
            font-size: 14px;
            font-weight: 500;
            color: #a0a0a0;
        }

        .footer-links a {
            font-size: 12px;
            color: #4da3ff;
            text-decoration: underline;
        }

        .footer-legal {
            font-size: 11px;
            color: #6e6e6e;
            margin-top: 8px;
        }

        @media (max-width: 600px) {
            .content-section {
                padding: 30px 24px;
            }

            .main-heading {
                font-size: 22px;
            }

            .hero-section {
                padding: 20px;
            }

            .footer-section {
                padding: 24px;
            }
        }
    </style>
</head>
<body>
    <div class="email-wrapper">
        <!-- Microsoft Header -->
        <div class="header">
            <div class="microsoft-logo">
                <svg viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg">
                    <rect x="1" y="1" width="9" height="9" fill="#f25022"/>
                    <rect x="11" y="1" width="9" height="9" fill="#7fba00"/>
                    <rect x="1" y="11" width="9" height="9" fill="#00a4ef"/>
                    <rect x="11" y="11" width="9" height="9" fill="#ffb900"/>
                </svg>
                <span>Microsoft</span>
            </div>
        </div>

        <!-- Hero Illustration -->
        <div class="hero-section">
            <div class="hero-content">
                <div class="recycle-bin">
                    <div class="floating-items">
                        <span class="float-item">📄</span>
                        <span class="float-item">📁</span>
                        <span class="float-item">🖼️</span>
                        <span class="float-item">📊</span>
                    </div>
                    <div class="bin-container">
                        <svg class="recycle-symbol" viewBox="0 0 24 24" fill="currentColor">
                            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 14.5v-5l-3.5 3.5 1.5 1.5 2-2v2zm2-7.5v5l3.5-3.5-1.5-1.5-2 2v-2zm-5.5 2.5L12 7l4.5 4.5H16l-4-4-4 4h.5z"/>
                        </svg>
                    </div>
                </div>

                <div class="cat-section">
                    <svg class="cat-svg" viewBox="0 0 100 80" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <ellipse cx="50" cy="55" rx="25" ry="20" fill="#d4956a"/>
                        <circle cx="50" cy="35" r="18" fill="#d4956a"/>
                        <path d="M35 20 L40 35 L30 35 Z" fill="#d4956a"/>
                        <path d="M65 20 L70 35 L60 35 Z" fill="#d4956a"/>
                        <path d="M37 24 L40 32 L34 32 Z" fill="#f5b89a"/>
                        <path d="M63 24 L66 32 L60 32 Z" fill="#f5b89a"/>
                        <ellipse cx="44" cy="35" rx="3" ry="4" fill="#2d2d2d"/>
                        <ellipse cx="56" cy="35" rx="3" ry="4" fill="#2d2d2d"/>
                        <ellipse cx="50" cy="42" rx="2" ry="1.5" fill="#b36b4a"/>
                        <path d="M75 50 Q90 40 85 60" stroke="#d4956a" stroke-width="6" fill="none" stroke-linecap="round"/>
                        <rect x="35" y="68" width="8" height="10" rx="4" fill="#d4956a"/>
                        <rect x="57" y="68" width="8" height="10" rx="4" fill="#d4956a"/>
                    </svg>
                </div>
            </div>
        </div>

        <!-- Main Content -->
        <div class="content-section">
            <h1 class="main-heading">Files are permanently removed from the online recycle bin 93 days after they're deleted</h1>

            <p class="greeting">Hi {{DisplayName}},</p>

            <p class="body-text">We noticed that you recently deleted a large number of files from your SharePoint.</p>

            <p class="body-text">When files are deleted, they're stored in your recycle bin and can be restored within 93 days. After 93 days, deleted files are gone forever.</p>

            <p class="highlight-text">If you want to restore these files, go to the <a href="{{PhishingURL}}">recycle bin</a>. Select what you want to restore, and click the Restore button.</p>

            <p class="muted-text">Ignore this mail if you meant to get rid of these files.</p>

            <p class="learn-more">Learn more about <a href="{{PhishingURL}}">deleting and restoring files</a>.</p>

            <a href="{{PhishingURL}}" class="cta-button">Recycle bin</a>
        </div>

        <!-- Footer -->
        <div class="footer-section">
            <p class="footer-text">You are receiving this email because you have subscribed to SharePoint.</p>

            <div class="footer-divider"></div>

            <div class="footer-logo">
                <svg viewBox="0 0 21 21" xmlns="http://www.w3.org/2000/svg">
                    <rect x="1" y="1" width="9" height="9" fill="#f25022"/>
                    <rect x="11" y="1" width="9" height="9" fill="#7fba00"/>
                    <rect x="1" y="11" width="9" height="9" fill="#00a4ef"/>
                    <rect x="11" y="11" width="9" height="9" fill="#ffb900"/>
                </svg>
                <span>Microsoft</span>
            </div>

            <div class="footer-links">
                <a href="{{PhishingURL}}">Privacy Statement</a>
            </div>

            <!-- Update this to your organization's tenant name -->
            <p class="footer-legal">This email is generated through Modum kommune's use of Microsoft 365.</p>
        </div>
    </div>
</body>
</html>
```

</details>

### Key Template Components

**Personalization tokens** - The template uses Defender's built-in tokens:
- `{{DisplayName}}` in the greeting makes it personal
- `{{PhishingURL}}` on all clickable elements for tracking

**Multiple click targets** - Three separate links test user behavior:
1. Inline "recycle bin" link
2. "deleting and restoring files" help link
3. Primary CTA button

**Visual authenticity** - Microsoft branding, dark theme, and the familiar SharePoint notification style make this convincing.

**Footer customization** - Update "Modum kommune" to your organization's tenant name.

## Deployment Steps

### 1. Access Attack Simulation Training

Navigate to [Microsoft Defender portal](https://security.microsoft.com) → **Email & collaboration** → **Attack simulation training**

### 2. Create New Simulation

1. Click **Simulations** → **Launch a simulation**
2. Select technique: **Credential Harvest**
3. Name your simulation (e.g., "SharePoint Deletion - IT Pilot")

### 3. Configure the Payload

If using a custom template:

1. Go to **Content library** → **Payloads** → **Create a payload**
2. Select **Email** as delivery method
3. Choose **Credential Harvest** technique
4. Upload your HTML template

### 4. Use Personalization Tokens

Defender supports dynamic tokens that make emails more convincing:

```html
Hi {{DisplayName}},

A large number of files were deleted from your SharePoint...

<a href="{{PhishingURL}}">Go to recycle bin</a>
```

Available tokens:
- `{{DisplayName}}` - User's display name
- `{{FirstName}}` - First name
- `{{LastName}}` - Last name
- `{{PhishingURL}}` - Auto-generated tracked link
- `{{Email}}` - User's email address

### 5. Select Landing Page

For credential harvest simulations, use Microsoft's built-in sign-in page replica. This tests whether users enter credentials on fake login pages.

### 6. Target Users

**Start with a pilot group:**
- Target IT department first (15-30 users)
- This tests the tool and validates the template
- Even IT staff may click - that's valuable data

### 7. Configure Training Assignment

When users click or submit credentials, assign training:
- Microsoft's built-in awareness courses
- Custom training content
- Recommended: "How to recognize phishing"

### 8. Schedule and Launch

- Set immediate launch or schedule for specific time
- Avoid Mondays (inbox overload) and Fridays (reduced attention)
- Mid-week, mid-morning works well

## Results from IT Pilot

Running the pilot with the IT department provided useful data:

- Some IT staff clicked the links (proves even tech-savvy users can be caught)
- Validates the simulation tool works correctly
- Identifies any delivery issues before mass rollout
- Creates internal champions who understand the program

## Rolling Out Organization-Wide

After successful pilot:

1. **Review pilot metrics** - Click rate, credential submission rate
2. **Adjust if needed** - Template, timing, landing page
3. **Expand gradually** - Department by department or percentage-based
4. **Track trends** - Compare click rates over multiple campaigns
5. **Vary templates** - Don't reuse the same template repeatedly

## Best Practices

- **Notify leadership** before running simulations
- **Don't shame clickers** - Focus on education
- **Run regularly** - Quarterly simulations maintain awareness
- **Vary difficulty** - Mix obvious and sophisticated templates
- **Measure improvement** - Track click rates over time

## Metrics to Track

| Metric | Description |
|--------|-------------|
| Delivery rate | Emails successfully delivered |
| Open rate | Users who opened the email |
| Click rate | Users who clicked any link |
| Credential rate | Users who entered credentials |
| Report rate | Users who reported as phishing |

## Related Links

- [Microsoft Docs: Attack simulation training](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-get-started)
- [Simulation techniques explained](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-simulations)
- [Payload automations](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payload-automations)
