# Quorum public site (privacy, terms, support, beta)

Static pages that the App Store, Google Play and the apps link to. They are published with
**GitHub Pages** from the public repository **`lucasmqar/quorum-site`** (branch `main`, folder `/`):

| Page | Public URL |
|---|---|
| Home | https://dailyquorum.com/ |
| Privacy Policy | https://dailyquorum.com/privacy.html |
| Terms of Use | https://dailyquorum.com/terms.html |
| Support | https://dailyquorum.com/support.html |
| Become a tester (ads landing page) | https://dailyquorum.com/beta.html |

This folder (`site/` in the main Quorum repo) is the source of truth. The `quorum-site` repo is
just a copy of its contents.

## How the pages work

- Each page is one self-contained HTML file: inline CSS, inline JS, inline SVG logo. Every page
  except `beta.html` has no external scripts, fonts, analytics or cookies, and stores nothing in the
  browser. `beta.html` is the exception: after the visitor accepts its cookie banner it loads
  Microsoft Clarity, the Meta Pixel and Google tags (GA4 / Google Ads), which set cookies; it also keeps
  the visit's UTMs / click IDs in `sessionStorage` (`q_attr`) and the banner choice in `localStorage`
  (`q_consent`). See the `beta.html` item below.
- Every page carries all six languages: `en`, `pt` (pt-BR), `es`, `fr`, `de`, `ja`. Each one is a
  `<div data-lang="xx">` block; English is visible by default and the others are `hidden`.
- The language is chosen from `?lang=en|pt|es|fr|de|ja` (a tag like `pt-BR` also works), otherwise
  from the browser's languages (`navigator.languages`), otherwise English. The selector in the
  header switches language and updates `?lang=` in the address bar.
- Without JavaScript all languages are shown, one after the other.
- Light and dark follow the system setting (`prefers-color-scheme`). Layout is mobile-first.
- Sharing: every page has Open Graph / Twitter tags pointing to `og.png` (1200×630, absolute URL) and an
  `apple-touch-icon.png` (180×180, opaque). Both are generated images; keep them next to the HTML files.
- `beta.html` (ads landing page): the tester links and measurement IDs are JS constants at the top
  (`TESTFLIGHT_URL`, `GROUP_URL`, `OPTIN_URL`, `PLAY_URL`, `CLARITY_ID`, `META_PIXEL_ID`, `GA4_ID`,
  `GADS_ID`, `GADS_LABEL`). Official Google Play badges live in `badges/`; the iPhone button is a
  TestFlight badge (Apple's App Store badge may only link to the App Store, so switch to it at launch).
  On a phone the page shows that phone's section first; on desktop it shows a two-badge chooser.
  Measurement tags load only when an ID is set **and** the visitor accepts the cookie banner (Google
  Consent Mode v2 defaults to denied). Every store/tester link has `data-ev`; a click pushes
  `tester_click` {platform, step, utm_*} to `dataLayer`, GA4 and Clarity, Meta `Lead` (first step of each
  platform: `testflight`, `hero_testflight`, `group`) or `TesterStep`, and the Google Ads conversion if
  `GADS_LABEL` is set. UTMs / click IDs are kept for the visit (also through the language switch) and
  forwarded to Google Play as `referrer`. `?p=ios|android` forces a platform. The privacy policy
  section `#site-<lang>` describes this.
- Effective date of the Privacy Policy and Terms: **2026-11-11**. Those two pages and every
  footer name PEDE FLUX TECNOLOGIA LTDA (CNPJ 69.008.345/0001-98) as the developer of Quorum.

## Before publishing

1. Replace the support e-mail placeholder in every page (it appears in all six languages):

   ```bash
   grep -c SUPPORT_EMAIL_PLACEHOLDER site/*.html
   sed -i '' 's/SUPPORT_EMAIL_PLACEHOLDER/your-address@example.com/g' site/*.html
   grep -c SUPPORT_EMAIL_PLACEHOLDER site/*.html   # must print 0 for every file
   ```

2. Open the pages locally and check a couple of languages:

   ```bash
   python3 -m http.server 8788 --directory site
   # http://localhost:8788/privacy.html?lang=pt
   ```

## Publishing to GitHub Pages

First time:

```bash
gh repo create lucasmqar/quorum-site --public
git clone https://github.com/lucasmqar/quorum-site.git /tmp/quorum-site
cp site/*.html site/*.png site/.nojekyll /tmp/quorum-site/
cd /tmp/quorum-site && git add -A && git commit -m "Publish Quorum site" && git push
gh api -X POST repos/lucasmqar/quorum-site/pages -f "source[branch]=main" -f "source[path]=/"
```

Updates: copy the files again, commit and push. GitHub Pages redeploys in a minute or two.

`.nojekyll` tells GitHub Pages to serve the files as they are. This README is not copied.

## Editing

Edit the HTML directly. When you change a text, change it in all six language blocks of that
page so they stay equivalent, and keep the in-app labels (for example `Settings → Delete account`)
identical to `i18n/ui.json`. If the data the app collects changes, update the Privacy Policy here
and in the apps (`ios/Quorum/UI/LegalView.swift`, `android/.../ui/Legal.kt`), plus the
App Store privacy labels and the Google Play Data safety form. Change the effective date at the
same time (the `<p class="meta">` line under each title, in every language block).
