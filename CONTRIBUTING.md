# Contributing

Thanks for wanting to work on the MSTR Software Solutions site.

This repository is public so the work is visible, **not** so it is open to unsolicited
changes. It is the live marketing site for a real business — every merge to `main`
deploys straight to https://mstrsoftwaresolutions.co.za. Please read this before you
write any code.

---

## 1. Get approval before you start

**Ask first. Every time. Including for small changes.**

Open an issue describing what you want to change and why, and wait for a maintainer to
approve it before you write anything.

A maintainer is an owner of the **MSTR-Software-Solutions** GitHub organisation.

You have approval when a maintainer has replied on the issue saying so, or assigned the
issue to you. A reaction, a "sounds good" from a non-maintainer, or silence is not
approval.

**Pull requests opened without an approved issue will be closed unread**, however good
the change is. This is not about the quality of your work — it is that an unreviewed
change to this repo is a change to a business's public front door, and we would rather
discuss it before you spend your time on it.

### What to put in the issue

- what is wrong, or what is missing
- what you propose to do about it
- which part of the page it touches
- anything visual: a screenshot or a rough mock

If you are reporting a problem rather than proposing a fix, just open the issue and stop
there. You do not need approval to report something.

### What usually gets approved

- bugs, accessibility barriers, performance problems, broken links
- anything already filed and labelled `help wanted` or `good first issue`

### What usually does not

- redesigns, palette changes, new sections, or copy rewrites — these are brand decisions,
  not code decisions
- adding a framework, a build step, or a package manager (see [The constraints](#3-the-constraints))
- new third-party dependencies
- dependency version bumps done for their own sake

If you think one of these is genuinely needed, open an issue and make the case. The
answer may well be yes. Just do not arrive with it already built.

---

## 2. Before you write code

Read [README.md](README.md) first. It is not a formality — it documents several things
that are easy to break without noticing, because breaking them produces **no error**:

- the [CSP](README.md#third-party-dependencies) silently blocks any third party not named
  in `netlify.toml`
- the [GSAP-optional contract](README.md#animation) is what keeps most of the page from
  being invisible when a CDN fails
- the [portrait hero gate](README.md#breakpoints) exists in two places that must agree

---

## 3. The constraints

These are deliberate. A PR that breaks one will be sent back.

**No build step.** One HTML file, no framework, no package manager, no bundler, no
`node_modules`. Markup, the full design system and all behaviour live in `index.html`.
Open it, edit it, refresh. Keep it that way.

**No new third-party dependencies without approval.** The site loads exactly three:
Google Fonts, Boxicons (unpkg), GSAP (cdnjs). Each is named explicitly in the
`Content-Security-Policy` in `netlify.toml`. **A fourth will be blocked by the browser
with no visible error** unless you also edit that policy — so if a dependency is approved,
updating the CSP is part of the same PR.

**Change the token, not the rule.** Colours, spacing and type live as CSS custom
properties at the top of the `<style>` block and are consumed throughout. Do not hardcode
a hex value because it was quicker.

**Animation is optional by design.** Anything you animate must stay reachable by
`.gsap-fade-up` or `.gsap-stagger-grid`, or it will be invisible whenever GSAP fails to
load. Test with the CDN blocked — see [Testing](#6-testing).

**Contrast is not negotiable.** Body text meets WCAG AA. The primary button fill is a
deeper blue than the brand accent for exactly this reason. If you lighten it, you break
it — the README explains the maths.

**Do not commit generated assets by hand.** Everything in `images/` except
`images/source/mstr-mark-master.png` is derived from that master. If the mark changes,
replace the master and regenerate.

**Keep the enquiry form in the markup.** Netlify detects it by parsing the deployed HTML.
A form injected by script does not exist as far as Netlify is concerned, and enquiries
are lost silently.

---

## 4. Branches and commits

Never commit to `main` directly. Branch from an up-to-date `main`:

```bash
git checkout main && git pull
git checkout -b short-description-of-change
```

Commit messages: a short imperative subject line, then a body explaining **why**, not
what — the diff already says what. If the change is not obvious, say what you ruled out
and why.

Reference the approved issue in the PR, not usually in every commit.

---

## 5. Pull requests

Open the PR against `main` and include:

- a link to the approved issue
- what changed and why
- **before/after screenshots for anything visual**, at both desktop and phone width
- how you tested it
- anything you were unsure about

Keep PRs to one concern. A PR that fixes a bug and also reformats a file is two PRs.

Every PR needs a maintainer's review and approval to merge. Expect review comments —
they are about the code, not about you. If a review asks a question, answering it in the
thread is fine; you do not have to change the code to close a comment.

---

## 6. Testing

There is no test suite. That means testing is manual and you are responsible for it.

Serve the site over HTTP rather than opening the file directly:

```bash
python -m http.server 8000
```

Before you open a PR, check:

- [ ] **Phone width.** Around 390x844 and 360x800. Most of the traffic is here.
- [ ] **Portrait and landscape.** The hero deliberately behaves differently in each.
- [ ] **No horizontal overflow.** `document.documentElement.scrollWidth` must not exceed
      `window.innerWidth`. Note that `body { overflow-x: hidden }` will hide overflow from
      you — check the number, not just the scrollbar.
- [ ] **With GSAP blocked.** Block `cdnjs.cloudflare.com` in devtools and reload. Every
      section must still be visible. This is the single easiest way to break the site
      badly without noticing.
- [ ] **Keyboard only.** Tab through it. The skip link should come first, focus should be
      visible throughout, and the mobile drawer should close on `Escape`.
- [ ] **Console is clean.**

The contact form cannot be tested locally — it is handled by Netlify, which only exists
on a deployed site. Submitting locally exercises the failure path, which is worth seeing
once.

---

## 7. Deploying

You do not deploy. Merging to `main` does.

Netlify builds from this repo with no build command. `main` goes to production; other
branches get a deploy preview if previews are enabled. Check the preview on your PR
before asking for review.

---

## 8. Reporting something sensitive

If you find a security problem, **do not open a public issue.** Email
admin@mstrsoftwaresolutions.co.za with what you found and how to reproduce it, and give
us a chance to fix it before it is discussed publicly.

---

## Questions

If something here is unclear or seems wrong, open an issue and ask. Being wrong in an
issue costs nothing; being wrong in a PR costs your afternoon.
