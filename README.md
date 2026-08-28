# MSTR Software Solutions

Marketing site for MSTR Software Solutions — a South African software studio building
mobile apps, web apps, SaaS platforms and custom business systems.

**Live:** https://mstrsoftwaresolutions.co.za

---

## What this is

A single-page static site. One HTML file, no build step, no framework, no package manager.

Everything — markup, the full design system, and all behaviour — lives in `index.html`.
Open it, edit it, refresh. That is the whole workflow.

```
.
├── index.html          # the entire site: markup, <style>, <script>
├── 404.html            # self-contained error page, no shared dependencies
├── netlify.toml        # security headers, caching, publish dir
├── robots.txt
├── sitemap.xml
├── images/
│   ├── favicon-32.png        # browser tab
│   ├── favicon-64.png        # browser tab, and the 404 page's mark
│   ├── apple-touch-icon.png  # 180x180 on the brand ground (iOS)
│   ├── mstr-logo.png         # header and footer lockup, 73x80
│   ├── hero-poster.jpg       # hero still, and the hero itself on mobile
│   ├── og-card.png           # 1200x630 social share card
│   └── source/
│       └── mstr-mark-master.png   # 473x521 master. Not served. Regenerate from this.
└── video/
    ├── mstr-bg-video.mp4     # hero background loop, H.264
    └── mstr-bg-video.webm    # same clip, VP9, preferred where supported
```

Everything in `images/` except the master is generated from
`images/source/mstr-mark-master.png`. If the brand mark changes, replace the master and
regenerate rather than editing the derivatives by hand.

### Third-party dependencies

All loaded from CDNs at runtime; nothing is installed or vendored.

| Dependency | Version | Source | Used for |
|---|---|---|---|
| Google Fonts | — | fonts.googleapis.com | Space Grotesk (display), Inter (body), JetBrains Mono (labels) |
| Boxicons | 2.1.4 | unpkg | all icons |
| GSAP + ScrollTrigger | 3.12.2 | cdnjs | scroll and entrance animations |

All three are named explicitly in the CSP in `netlify.toml`. **Adding a fourth means
editing that policy**, or the browser will block it silently.

GSAP is deferred and treated as optional — see [Animation](#animation).

---

## Running it locally

No dependencies to install. Serve the directory over HTTP:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

Opening `index.html` directly via `file://` mostly works, but a real HTTP server is
closer to production.

**The contact form will not work locally.** It is handled by Netlify (see below), which
only exists on a deployed site. Submitting locally exercises the failure path, which is
worth seeing at least once.

---

## Page structure

Sections in document order, with their anchor IDs. All of them sit inside `<main>`.

| Section | ID |
|---|---|
| Hero (background video) | — |
| Who We Are | `#about` |
| Why work with us | `#values` |
| What We Do | `#services` |
| Our Solutions | `#projects` |
| Who We Serve | — |
| Development Approach | `#approach` |
| FAQ | — |
| Contact | `#contact` |
| Footer | — |

Every footer link points at one of these. The Services column also carries
`data-service-tab`, which opens the matching tab before the anchor scrolls.

---

## Design system

All design decisions are CSS custom properties at the top of the `<style>` block in
`index.html`. **Change the token, not the rule** — the tokens are consumed throughout.

```css
--void:           #03050E   /* page background */
--deep:           #070B1F   /* raised surfaces */
--electric:       #0091FF   /* primary accent: borders, glows, icons */
--electric-deep:  #0072E0   /* primary button fill */
--electric-soft:  #4FB8FF   /* hover, icons, eyebrows */
--cyan:           #22D3EE   /* secondary accent */
```

Notes on the palette:

- It is a dark theme with a single blue accent. The blue is fully saturated in HSL, so
  "more vibrant" means purifying the hue and deepening the darks around it, not raising
  saturation — there is no headroom there.
- Surfaces and hairline borders are **blue-tinted**, not neutral white. This is
  deliberate; it is most of what makes the palette feel lit rather than grey.
- `--electric` is too light to carry white text. The primary button fills with
  `--electric-deep`/`--electric-deeper` (4.68:1 and 5.45:1) and keeps `--electric` on the
  ring and glow. **Hover brightens the ring, not the fill** — brightening the fill puts
  the label back under 4.5:1.
- Every radius token is `0`. The sharp-cornered look is intentional and consistent.
- Type scales are `clamp()` based, so headings are fluid without breakpoints.

Body text meets WCAG AA against both backgrounds, most of it AAA.

### Animation

Elements tagged `.gsap-fade-up` or `.gsap-stagger-grid` start hidden and are revealed by
GSAP on scroll. The animation targets an element's **children**, not the element itself —
so a container that carries its own border or background should wrap the animated element
rather than being tagged directly, or its border will not paint.

GSAP arrives from a CDN, so it is **optional by design**. Three independent fallbacks
guarantee the content is never left hidden:

1. `.no-anim` on `<html>`, set at `DOMContentLoaded` if `gsap` or `ScrollTrigger` is
   missing — covers a CDN outage, an ad blocker, a corporate proxy, or being offline
2. a `prefers-reduced-motion: reduce` media query
3. a `<noscript>` style block, for JS disabled entirely

If you add a new animated element, it must be reachable by one of the two reveal
selectors, or it will be invisible whenever GSAP is.

### Breakpoints

| Width | What changes |
|---|---|
| ≤1024px | desktop nav is replaced by the hamburger and drawer |
| ≤900px | services, projects, contact and footer grids collapse to one column |
| ≤600px | hero switches to `100dvh`; the background video is not downloaded |

---

## Media strategy

The hero clip is the heaviest asset on the site, so it is **not** in the markup. The
`<video>` carries a `poster` and `data-webm` / `data-mp4` attributes, and sources are
attached by script only when all of these hold:

- viewport is wider than 600px
- `prefers-reduced-motion` is not set
- `navigator.connection` reports neither `saveData` nor a 2g `effectiveType`

Otherwise the poster *is* the hero, and the existing play button lets the visitor opt in.
That button is also the recovery path if a browser refuses programmatic playback.

Re-encoding, if the source clip ever changes:

```bash
# strip audio (the page never unmutes it) and compress hard - it sits behind
# a brightness(0.4) filter, so it tolerates far more than typical footage
ffmpeg -i source.mp4 -an -c:v libx264 -preset slow -crf 30 -movflags +faststart \
       video/mstr-bg-video.mp4
ffmpeg -i source.mp4 -an -c:v libvpx-vp9 -crf 46 -b:v 0 -row-mt 1 \
       video/mstr-bg-video.webm
# poster: the frame where the logo is fully revealed
ffmpeg -ss 6.0 -i source.mp4 -frames:v 1 -q:v 4 images/hero-poster.jpg
```

---

## Contact form

Handled by **Netlify Forms**. There is no backend and no third-party form service.

- The form posts urlencoded to the site root with `form-name=contact`.
- On success the visitor stays on the page and gets an inline confirmation.
- On failure they stay on the page and get a `mailto:` link **pre-filled with everything
  they typed**, so a broken handler never costs an enquiry.
- With JS unavailable it falls back to a native POST and Netlify redirects to
  `/?sent=1#contact`, which renders the same confirmation.
- Spam is filtered by a honeypot field (`bot-field`), declared via `netlify-honeypot`.

Netlify detects the form by parsing the deployed HTML, so **the form must exist in
`index.html` at deploy time** — it cannot be injected by script.

### If sending stops working

Symptom: every submission shows the failure message. Diagnose from outside:

```bash
curl -o /dev/null -w "%{http_code}\n" -X POST https://mstrsoftwaresolutions.co.za/ \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "form-name=contact" --data-urlencode "bot-field=probe"
```

Filling `bot-field` means Netlify discards it as spam, so this probes the endpoint
without creating a real submission.

- **200** — the handler is live; the problem is elsewhere.
- **404** — Netlify has no form registered for this site. Enable **Site configuration →
  Forms → form detection**, then **trigger a new deploy**. Detection only runs at deploy
  time, so enabling it alone changes nothing.

### Where submissions go

Every submission is stored under **Netlify → Forms**, whether or not email works.

To get email notifications, set the recipient under
**Netlify → Forms → Form notifications**. This is dashboard configuration, not code:
the address does not appear in the repo except as display text next to the form.

---

## Deploying

Netlify builds from this repo. There is no build command — the site is served as-is,
with headers and caching from `netlify.toml`.

- `main` → production at https://mstrsoftwaresolutions.co.za
- Any other branch → deploy preview, if previews are enabled

Work on a branch and merge to `main` to release.

```bash
git checkout -b my-change
# edit index.html
git commit -am "..."
git push -u origin my-change
```

### Caching

Images and video are cached for 30 days, not a year. **Asset filenames are not
content-hashed**, so a longer `max-age` would strand returning visitors on a stale logo
until it expired. If content hashing is ever introduced, `netlify.toml` can safely move
to a year with `immutable`.

`index.html` always revalidates, or a deploy would not reach anyone.

---

## Known issues

Tracked in [GitHub Issues](../../issues).
