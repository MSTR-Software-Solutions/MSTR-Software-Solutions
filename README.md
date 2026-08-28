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
├── images/
│   ├── mstr.png        # favicon + apple-touch-icon
│   ├── mstr-logo.png   # header and footer logo
│   └── mstr-logo.jpeg  # currently unreferenced
└── video/
    └── mstr-bg-video.mp4   # hero background loop
```

### Third-party dependencies

All loaded from CDNs at runtime; nothing is installed or vendored.

| Dependency | Version | Source | Used for |
|---|---|---|---|
| Google Fonts | — | fonts.googleapis.com | Space Grotesk (display), Inter (body), JetBrains Mono (labels) |
| Boxicons | 2.1.4 | unpkg | all icons |
| GSAP + ScrollTrigger | 3.12.2 | cdnjs | scroll and entrance animations |

> GSAP is currently load-bearing for rendering, not just decoration — see
> [#2](../../issues/2) before assuming the page degrades gracefully without it.

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
only exists on a deployed site. Submitting locally hits the fallback path.

---

## Page structure

Sections in document order, with their anchor IDs where they have one:

| Section | ID |
|---|---|
| Hero (background video) | — |
| Who We Are | `#about` |
| Values / pillars | — |
| What We Do | `#services` |
| Our Solutions | `#projects` |
| Who We Serve | — |
| Development Approach | `#approach` |
| FAQ | — |
| Contact | `#contact` |
| Footer | — |

---

## Design system

All design decisions are CSS custom properties at the top of the `<style>` block in
`index.html`. **Change the token, not the rule** — the tokens are consumed throughout.

```css
--void:          #03050E   /* page background */
--deep:          #070B1F   /* raised surfaces */
--electric:      #0091FF   /* primary accent */
--electric-soft: #4FB8FF   /* hover, icons */
--cyan:          #22D3EE   /* secondary accent */
```

Notes on the palette:

- It is a dark theme with a single blue accent. The blue is fully saturated in HSL, so
  "more vibrant" means purifying the hue and deepening the darks around it, not raising
  saturation — there is no headroom there.
- Surfaces and hairline borders are **blue-tinted**, not neutral white. This is
  deliberate; it is most of what makes the palette feel lit rather than grey.
- Every radius token is `0`. The sharp-cornered look is intentional and consistent.
- Type scales are `clamp()` based, so headings are fluid without breakpoints.

Body text meets WCAG AA against both backgrounds, most of it AAA. The one known
exception is the primary button — [#7](../../issues/7).

### Animation

Elements tagged `.gsap-fade-up` or `.gsap-stagger-grid` are hidden by default and
revealed by GSAP on scroll. The animation targets an element's **children**, not the
element itself — so a container that carries its own border or background should wrap
the animated element rather than being tagged directly, or its border will not paint.

---

## Contact form

Handled by **Netlify Forms**. There is no backend and no third-party form service.

- The form posts urlencoded to the site root with `form-name=contact`.
- On success the visitor stays on the page and gets an inline confirmation.
- With JS unavailable it falls back to a native POST and Netlify redirects to
  `/?sent=1#contact`, which renders the same confirmation.
- Spam is filtered by a honeypot field (`bot-field`), declared via `netlify-honeypot`.

Netlify detects the form by parsing the deployed HTML, so **the form must exist in
`index.html` at deploy time** — it cannot be injected by script.

### Where submissions go

Every submission is stored under **Netlify → Forms**, whether or not email works.

To get email notifications, set the recipient under
**Netlify → Forms → Form notifications**. This is dashboard configuration, not code:
the address does not appear in the repo except as display text next to the form.

---

## Deploying

Netlify builds from this repo. There is no build command — the site is served as-is.

- `main` → production at https://mstrsoftwaresolutions.co.za
- Any other branch → deploy preview, if previews are enabled

Work on a branch and merge to `main` to release.

```bash
git checkout -b my-change
# edit index.html
git commit -am "..."
git push -u origin my-change
```

---

## Known issues

Tracked in [GitHub Issues](../../issues). The two worth reading before making changes:

- [#2](../../issues/2) — most of the page is invisible if the GSAP CDN fails; the no-JS
  fallback in the CSS is dead code
- [#3](../../issues/3) — there is no navigation at all below 1024px

Others cover SEO metadata, image and video weight, accessibility, and repo setup.
