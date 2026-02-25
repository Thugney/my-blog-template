# Hugo Blog Template - Microsoft Cloud Tech

A bilingual (English/Norwegian) Hugo blog template focused on Microsoft cloud technologies: Intune, Autopilot, Entra ID, Defender, and Purview.

## Features

- **Bilingual support** - English and Norwegian with language switcher
- **PaperMod theme** - Clean, fast, SEO-friendly
- **CSS-generated covers** - Gradient covers based on post tags (no image files needed)
- **Dark/Light mode** - Follows system preference
- **Security headers** - Pre-configured for A+ security rating
- **Netlify-ready** - Deploy config included
- **SEO optimized** - Sitemap, RSS, Open Graph, Twitter Cards

## Quick Start

```bash
# Clone this template
git clone -b template https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

# Initialize the theme submodule
git submodule update --init --recursive

# Run locally
hugo server -D

# View at http://localhost:1313
```

## Configuration

1. **Edit `hugo.toml`** - Update these fields:
   - `baseURL` - Your site URL
   - `title` - Your blog title
   - `author` - Your name
   - `copyright` - Your copyright
   - Profile settings under `[languages.en.params.profileMode]` and `[languages.no.params.profileMode]`
   - Social links under `[[params.socialIcons]]`

2. **Edit `netlify.toml`** - Update redirects if using a different domain

3. **Add your profile image** - Place at `static/images/profile.jpg` (150x150px recommended)

4. **Edit About pages**:
   - `content/about.en.md` - English version
   - `content/about.md` - Norwegian version

## Creating Posts

Posts use bilingual naming: `my-post.en.md` and `my-post.no.md`

```yaml
---
title: "Your Post Title"
date: 2026-01-15
draft: false
tags: ["intune", "powershell"]
categories: ["How-To"]
summary: "Brief description"
description: "SEO description (150-160 chars)"
keywords: ["keyword1", "keyword2"]
---

Your content here...
```

### Cover Colors

The first tag determines the CSS cover gradient:

| Tag | Color |
|-----|-------|
| intune | Blue-teal |
| defender | Red |
| entra-id | Purple-blue |
| autopilot | Green-teal |
| purview | Deep purple |
| windows-11 | Slate blue |
| powershell | Dark blue |
| (default) | Teal |

## Folder Structure

```
content/
├── posts/          # Published posts (.en.md and .no.md)
├── about.md        # Norwegian about page
├── about.en.md     # English about page
└── archives.md     # Archive page

static/
├── images/         # Your images (profile, post images)
├── favicon.png     # Site favicon
└── og-default.png  # Social sharing image

layouts/
├── partials/
│   ├── cover.html  # CSS cover gradient logic
│   └── footer.html # Custom footer
└── robots.txt      # SEO robots template

assets/css/extended/
└── custom.css      # Your custom styles (teal color scheme)
```

## Deployment

### Netlify (Recommended)

1. Push to GitHub
2. Connect repo to Netlify
3. Build command: `hugo --gc --minify`
4. Publish directory: `public`
5. Set custom domain in Netlify

The included `netlify.toml` handles build settings and security headers.

### Other Platforms

Works with any static hosting: Vercel, GitHub Pages, Cloudflare Pages, etc.

## Customization

### Colors

Edit `assets/css/extended/custom.css` to change the teal accent color.

### Cover Gradients

Edit `layouts/partials/cover.html` to add new tag-based gradients.

### Security Headers

Security headers are configured in `netlify.toml` and `static/_headers`.

## License

MIT - Use freely for your own blog.

## Credits

- [Hugo](https://gohugo.io/) - Static site generator
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) - Theme
