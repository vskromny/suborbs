# SubOrbs landing page — audit

Reviewed: `index.html`, `privacy-page.html`, `terms-page.html`
Reference material: four App screenshots (home light, home dark, subscription list, subscription detail)
Date: 19 Aug 2026 · branch `claude/landing-page-audit-j0o47v`

The screenshots were the first ground truth available for what the app actually does, so most of
this audit is a comparison of three sources that disagreed with each other: the landing page, the
screenshots, and the privacy policy that is already published next to the landing page.

---

## A. Claims on the landing page that contradicted the app's own privacy policy

These are the serious ones. The old landing copy promised more privacy than the app delivers, and
the privacy policy sitting one click away said so. That is the kind of mismatch that gets flagged in
App Store review and, in the EU, reads as a misleading commercial practice.

| # | Old landing copy | What `privacy-page.html` says | Status |
|---|---|---|---|
| A1 | "Private by design. **Nothing leaves your phone.**" | §2: a scanned bill is "sent over an encrypted (HTTPS) connection to our processing backend (hosted on Vercel), which forwards it to **OpenAI's API**" | **Fixed** — the page now states the Snap & Read exception in the feature section *and* in the privacy band |
| A2 | "**Apple Intelligence** reads it right on your iPhone … **Nothing is uploaded. Ever.**" | Same as A1 — the reader is OpenAI, not Apple Intelligence, and the image *is* uploaded | **Fixed** — section rewritten around what actually happens (downscale, EXIF strip, HTTPS, discard) |
| A3 | "optionally sync across your own devices through your **private iCloud** — encrypted, invisible to us" | §3: "The current version of SubOrbs **does not sync your data to iCloud** or any other cloud service" | **Fixed** — claim removed |
| A4 | "**No server.**" (three places) | §2/§4: there is a Vercel backend for Snap & Read, and StoreKit receipt validation over the network | **Fixed** — narrowed to "no SubOrbs server holding your data", which is true and still strong |
| A5 | "**Free during the beta.**" | §4 and the Terms describe **SubOrbs Pro**, an auto-renewable App Store subscription | **Partly fixed** — now "Free to use, with an optional SubOrbs Pro subscription". See D4: the page still never says what Pro costs or unlocks |
| A6 | "**Inspectable** — see everything SubOrbs knows about you, on one screen, any time." | No such screen appears in the screenshots or the policy; §6 describes "Delete all data" in Settings | **Fixed** — replaced with "No account", which is verifiable |

---

## B. Content that did not match the screenshots

| # | Was | Now |
|---|---|---|
| B1 | Invented totals: € 75.75 / month, € 909.00 / year | Real: **€ 180.31 monthly, € 2'163.76 / year** |
| B2 | Invented subscriptions: Figma, Google, "U", a gym | Real: **Netflix € 50.00, Anthropic US$ 100.00, Spotify € 10.99, Apple Music € 10.99, Apple Developer € 100.00** |
| B3 | "Gym membership renewed — € 24.90 for August. **You've used it 9 times.**" | Removed. Usage tracking does not exist and would contradict the no-analytics promise |
| B4 | Orbs are "sized by cost, **coloured by category, positioned by how soon it renews**" | The screenshots show brand marks and price tags, not category colours or renewal-ordered positions. Copy rewritten to what is visible |
| B5 | Mock list rows: "Renews Thu", "Renews in 6 days" | The app's actual pattern: weekday + date, plus a colour-coded countdown pill — **IN 3 DAYS** (red), **IN 4 DAYS** (orange), **IN 338 DAYS** (grey) |
| B6 | — | **Multi-currency** was missing entirely. The home screen converts a mixed-currency portfolio and labels the total "ⓘ mixed currencies". It is a real differentiator; it now has a paragraph |
| B7 | — | Also missing and now on the page: **Insights**, the profile / "All subscriptions" switcher, **per-subscription notification lead time**, the computed "Cancel by Friday" deadline, **Mark as cancelled** vs **Delete**, first-charge history, category and currency fields |
| B8 | Reminder mock said "Netflix renews in 3 days · Cancel by Thursday to avoid the € 17.99 charge" | Matches the app's real string and real data: "Apple Music renews in 3 days · Cancel by Friday to avoid the € 10.99 charge" |

---

## C. Technical and rendering problems (all fixed)

1. **The screenshot gallery rendered as nothing.** All four slots were
   `<x-import component-from-global-scope="image-slot" …>` — Claude Design canvas elements with no
   browser meaning. On the live GitHub Pages site, the section headed *"Real screenshots from the
   app"* was four empty phone bezels. Replaced with real `<img>` elements.
2. **The document had no real `<head>`.** `<title>`, the stylesheet and the viewport tag lived
   inside `<x-dc><helmet>` in the body. Moved into a proper `<head>`; the `<x-dc>` wrapper and the
   inert `<script type="text/x-dc">` (which carried an unused `ctaLabel` prop) were dropped.
   *Note: this means `index.html` is no longer re-importable into the design canvas. Keep the canvas
   as the source if you still edit there, and export into this file.*
3. **`<html>` had no `lang`.** Screen readers had to guess the language.
4. **No metadata at all**: no description, canonical, favicon, apple-touch-icon, `theme-color`, or
   Open Graph / Twitter cards. A link to the site pasted into any chat app showed a bare URL.
   Added, plus a generated `assets/og-image.png` (1200×630).
5. **The page called Google on every load.** Brand icons came from
   `https://www.google.com/s2/favicons?domain=netflix.com` — a third-party request, with referrer,
   on a page whose entire pitch is "no tracking". Removed; the orbs are pure CSS now.
6. **Phone frames were the wrong shape.** Frames were `aspect-ratio:1/2.05`; the screenshots are
   1320×2868 (`1/2.173`). Real screenshots dropped into those frames would have been cropped.
   Frames now match the source images exactly.
7. **Duplicate `style` attribute** on the nav CTA — the second one was silently discarded by the
   parser. Hover/active states were driven by non-standard `style-hover` / `style-active` attributes
   plus a JS shim, which meant no hover feedback for anyone with JS disabled and none at all on
   keyboard focus. Replaced with real CSS `:hover` / `:active` / `:focus-visible`; the shim is gone.
8. **`prefers-reduced-motion` broke the hero.** `*{animation:none !important}` killed the orbit
   animation — but that animation was also the only thing *positioning* the five satellites, so they
   all collapsed onto one point at 12 o'clock. Placement is now a static `transform` and the
   animation only spins, so the hero is laid out correctly whether motion is on or off.
9. **The legal pages were unreachable.** The footer's "Privacy Policy" linked to `#privacy`, the
   on-page band — so `privacy-page.html` and `terms-page.html` had no inbound link from anywhere on
   the site. Both are now linked from the footer, and the privacy band links to the full policy.
10. **Contrast below WCAG AA on the light theme.** Measured against the canvas gradient's darkest
    stop (`#E9E4F8`): the caption colour `--t3 #8A82A8` was **3.0:1** (needs 4.5:1 for body text),
    `--t2 #6B6490` was **4.58:1** — a hair over. Retuned to `--t3 #655D83` (**4.9:1**),
    `--t2 #5D5682` (**5.4:1**), `--link #5030D4` (**6.2:1**). Dark theme was already fine and was
    nudged up as well.
11. **No skip link, no focus styles.** Both added.
12. Images now carry `width`/`height` (no layout shift), `srcset`/`sizes`, `loading="lazy"` and real
    alt text. All four screenshots together are ~150 KB as WebP, versus 11 MB for the source PNGs.

---

## D. Still open — your call

1. **The CTA goes nowhere.** Both "Get SubOrbs" buttons are `href="#"`. This is the single biggest
   hole on the page: every conversion path is a dead end. It needs a TestFlight link, an App Store
   link, or at minimum an email-capture form.
2. **"iOS 17+" is unverified.** It is asserted in the hero badge and the CTA. I could not check it
   against the Xcode project from this repo — confirm the real deployment target.
3. **Your personal mobile number was in the footer** (`+41 78 308 77 07`). I removed it: a personal
   number on a public page is a spam magnet and is not required by anything. If you need a phone
   contact for App Store review, give it to Apple in App Store Connect, not to the open web. The
   email is still there; a dedicated `support@` alias would be better than a personal Gmail.
4. **Pricing is invisible.** SubOrbs Pro exists in the policy and the Terms, but the page never says
   what it costs or what it unlocks. Add a short pricing block before launch.
5. **The legal pages are dark-only.** `index.html` follows `prefers-color-scheme`; the two legal
   pages are hard-coded dark. They also read as second-class URLs — `privacy.html` / `terms.html`
   would be cleaner than `privacy-page.html` / `terms-page.html`. Rename now, before links get shared.
6. **Dates need a pass at launch.** Privacy policy "Effective date: 22 July 2026", footer "© 2026".
   Fine today; check them on the day you ship.
7. **The screenshots are not marketing-grade.** They show a real status bar — 19:44 / 19:45, a
   nearly-empty battery, one bar of signal, and the silent-mode bell. Apple's own convention is
   9:41 with a full battery, and the App Store listing will want that anyway. Re-capture from a
   simulator with a clean status bar and reuse them here.
8. **Two app-side issues visible in the screenshots** (not landing-page bugs, but worth logging):
   - The list row *"Apple Development Program · Thu, Jul 22 · IN 338 DAYS"* omits the year. The date
     is Thu **22 Jul 2027** and it is correct, but next to "Jul 22" with no year it reads as three
     weeks in the past. Show the year once a date is beyond ~11 months.
   - On the light home screen the orbit labels collide: the "€ 100.00" tag overlaps the Apple Music
     orb, and "A"/Apple sit almost on top of each other. Some collision avoidance, or hiding tags
     for the smallest orbs, would help.
9. **Nothing measures the page.** No analytics is a defensible choice and consistent with the pitch —
   just be aware you will have zero idea whether the page works. If you ever want numbers, use a
   cookieless, IP-truncating counter and disclose it in §5 of the policy.
10. **Missing site plumbing**: no `robots.txt`, no `sitemap.xml`, no App Store badge, no FAQ, no
    screenshots of Insights / Settings / Add-subscription (all real screens the page never shows).
11. **Screenshots ship as WebP only**, with no PNG/JPEG fallback. That covers roughly 97% of
    browsers; anything older shows the alt text. Easy to add a `<picture>` fallback if you care.
12. `assets/og-image.png` is 367 KB. Only crawlers fetch it, so it does not affect page speed, but a
    WebP or JPEG variant would be tidier.
13. `background-attachment:fixed` on `body` is known to be janky when scrolling long pages in iOS
    Safari — the exact device your audience is on. Consider a `position:fixed` background layer.

---

## Files added

```
assets/favicon.svg              purple-orb mark
assets/apple-touch-icon.png     180×180
assets/og-image.png             1200×630 social preview
assets/screenshots/*.webp       4 screenshots × 2 widths (360w / 720w)
```
