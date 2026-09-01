# A minimal SolarAssistant customer portal

Your customers sign in, see their sites, and manage their account — under your name, your colours
and your logo. SolarAssistant serves this repository directly: a request path maps onto a file
here, and your organization's own details are substituted into each page as it is served.

Nothing in it is specific to any company, and there is no branding to strip out — which is what
makes it safe to start from. It is deliberately the smallest complete portal there is, meant to be
read in one sitting and changed.

## Getting started

You do not need to copy anything to *use* a portal. Choosing this template in SolarAssistant gives
you a working portal in your own branding, and most companies never need more than that.

Take a copy when you want to change something — the wording, the layout, an extra page:

1. **Make your own repository from this one** with GitHub's **Use this template** button, which is
   where the button on your Cloud portal page sends you. Note that it *generates* a repository
   rather than forking one, deliberately: GitHub ties a fork's visibility to the repository it came
   from, so **a fork of a public repository can never be made private**. Generating gives you a
   repository you are free to keep private. The cost is that it starts with no history in common
   with ours — see [Staying up to date](#staying-up-to-date), which is worth reading before you
   have made many changes.
2. **Connect that repository** on your organization's **Cloud portal** page:

   ```
   https://solar-assistant.io/organizations/<your-organization-id>/portal
   ```

   If you do not know your organization id, start at
   [solar-assistant.io/user](https://solar-assistant.io/user), click your organization, then
   **Cloud portal**. You choose the repository once; after that it is yours.
3. **Set your branding in SolarAssistant**, not in the code. Your name, colours and logo live on
   your organization record and are substituted into the pages — see [Tokens](#tokens). There is
   nothing company-specific to edit here. Upload a logo too — without one you get a plain wordmark
   built from your name.
4. **Edit, commit, push.** A push changes nothing on its own: you stage a revision to preview it and
   publish when you are happy. See [Publishing](#publishing).

The same **Cloud portal** page is where you choose a ready-made template, stage a revision to preview
it, and publish. It is the one page to bookmark.

Fuller help pages are coming. For now this README is the documentation.

## Working on it locally

**Branded, without committing anything — `sacli portal`.** This is the loop worth having. It serves
your checkout the way SolarAssistant serves it: it reads `solar-assistant.json` and serves only the
directory `root` names, resolves `/sign_in` to `sign_in.html`, substitutes your organization's name
and colours into every page, serves `/assets/logo.svg` from your real stored logo, and reloads the
browser as you save.

```bash
sacli portal --org 42      # your organization id, needed once; later runs remember it
```

The pages call the live API from the browser exactly as they do in production, so **sign-in works**
— and so does everything else. Reads are your own organization's data, but a page that invites or
removes a user is acting on real people, not a sandbox.

`sacli` is [Solar-Assistant/sacli](https://github.com/Solar-Assistant/sacli); the portal command is
documented in [docs/portal.md](https://github.com/Solar-Assistant/sacli/blob/main/docs/portal.md),
including `--components` for pointing the pages at a checkout of the web components rather than the
published bundle.

**Preview what you have staged.** Commit, push, then refresh staging from your **Cloud portal**
page and open the **Staging URL** shown there — `staging-<your-name>.solar-power.live`. It serves
the revision you have staged, so you can look at a change before your customers do.

**Or serve `dist/` with any static server**, if you only care about layout. It must serve
`/sign_in` as `sign_in.html`: `python3 -m http.server` does not do extensionless URLs, so `/`
redirects to `/sign_in` and the first thing you see is a 404 that looks like a broken template.
Expect it to look unbranded — nothing substitutes tokens, so the tab reads `{{ org_name }}`, colours
fall back to the neutral palette in `dist/assets/style.css`, and sign-in cannot work because
`organization-id` is still a token rather than a number. It shows you structure, not branding.

**If a page looks slate grey on your real portal, that is not a fault.** It means no primary colour
is set on your organization yet — set one on your **Cloud portal** page and it appears. Grey is the
honest default rather than a failure.

## Staying up to date

We keep releasing functionality into this template, and we would rather you took it than drifted.

**Most of what we ship reaches you without you doing anything.** These pages are thin shells around
the components, and those are loaded at runtime from `cdn.solar-assistant.io` rather than copied
into your repository. When we improve what happens inside `<sa-sign-in>`, `<sa-sites>` or any of the
others, your customers get it on their next page load and there is nothing to merge. Only changes to
the HTML in this repository need pulling: a new page, a new component tag, a new token.

When there is one, add us as a second remote and merge:

```
git remote add upstream https://github.com/Solar-Assistant/portal-minimal.git
git fetch upstream
git merge upstream/main --allow-unrelated-histories
```

**The first merge is unpleasant, and that is expected rather than a fault.** Because your repository
was generated rather than forked, it begins at a single commit that shares no ancestry with ours.
Git refuses outright without the flag:

```
fatal: refusing to merge unrelated histories
```

With the flag it proceeds, but git has no common ancestor to compare against, so it treats every
file as added on both sides at once. If you have edited the pages, expect conflicts in most of them
and work through them once.

**It is bad exactly once.** That merge joins the two histories permanently. Every update after it is
an ordinary merge against a real common ancestor, touching only what actually changed, and the
`--allow-unrelated-histories` flag is never needed again. The whole cost is paid on the first pull,
which is a good reason to do it early rather than after a year of changes.

## What it is built on

The pages are plain HTML — no framework, no build step. The moving parts are the SolarAssistant web
components, loaded from one script tag:

```html
<script type="module" src="https://cdn.solar-assistant.io/js/solar-assistant.js"></script>
```

`<sa-sign-in>`, `<sa-register>`, `<sa-sites>` and `<sa-user>` are those components. They talk to the
API from the browser, so there is no server of yours involved anywhere.

They come from the **SolarAssistant Web Integration Kit** —
[github.com/Solar-Assistant/js_solar_assistant](https://github.com/Solar-Assistant/js_solar_assistant),
published as
[`@solar-assistant/components`](https://www.npmjs.com/package/@solar-assistant/components) and
[`@solar-assistant/api`](https://www.npmjs.com/package/@solar-assistant/api). If you want to go
further than editing these pages — build the portal with a framework, or put monitoring into a site
you already have — that kit is what to read next.

## Layout

**The site lives in `dist/`, not at the root**, because the required `solar-assistant.json` says so:

```
solar-assistant.json   { "root": "dist" }
README.md              this file — not served, because it is outside dist/
dist/                  everything below is served
  index.html
  sign_in.html
  register.html
  sites.html
  user.html
  terms.html
  privacy.html
  assets/style.css
  assets/favicon.svg
```

So `/sign_in` is `dist/sign_in.html`, and this README is not reachable on your portal. See
[Serving from a subdirectory](#serving-from-a-subdirectory) if you want to move or rename it.

## Tokens

Six tokens are replaced, **in `.html` files only**:

| Token | Becomes | Used for |
|---|---|---|
| `{{ org_id }}` | the organization's numeric id | `organization-id` on the components |
| `{{ org_name }}` | the organization's name, HTML-escaped | page `<title>`, logo `alt` text |
| `{{ org_primary }}` | its primary colour, or neutral slate if none is set | `--sa-primary` |
| `{{ org_accent }}` | the same colour at 10% — a soft tint, derived for you | `--sa-accent` |
| `{{ org_primary_dark }}` | the same, legible on a dark background | a dark-mode `--sa-primary` |
| `{{ org_accent_dark }}` | that colour at 10% | a dark-mode `--sa-accent` |

**The dark pair is always safe to use**, even for a company that has never set up a dark theme:
the value falls back to a neutral legible on a dark background rather than rendering empty. These
pages do not use them yet — there is no dark styling to attach them to — but if you add your own,
they are there.

An unknown token is left visible rather than blanked, so a typo shows up as `{{ org_nmae }}` on
the page instead of silently rendering nothing. It also writes a warning to the server log on
every single request for that page, so a stray token is worth fixing even where nobody sees it.

**Substitution does not know what an HTML comment is.** It is a plain text replacement over the
whole file, so a token inside a comment is replaced exactly like one in the markup:

```html
<!-- Do not change {{ org_id }} --> renders as <!-- Do not change 42 -->
```

Which destroys the instruction it was meant to leave behind. When you write a comment about a
token, name it in prose — `org_id` — and never in its `{{ … }}` form. The comments already in
these files follow that rule; keep it if you add your own.

**`{{ org_id }}` is the one to be careful with.** It appears in `dist/sign_in.html` and
`dist/register.html` — the two pages that mount components. A wrong or missing value does not look
wrong; it wires the sign-in and registration forms to the wrong organization while every page
still renders correctly.

## Why the colours are set in the page, not in `dist/assets/style.css`

Substitution runs on HTML only, so a `{{ … }}` inside a `.css` file is never replaced. Each page
therefore carries its own `:root` block in `<head>`, after the stylesheet link:

```html
<link rel="stylesheet" href="/assets/style.css" />
<style>:root { --sa-primary: {{ org_primary }}; --sa-accent: {{ org_accent }}; }</style>
```

Both target `:root`, so the later one wins. `dist/assets/style.css` keeps a neutral fallback for
organizations with no colours set and for viewing the template outside the portal.

**If you add a page, copy that `<style>` line into it.** A page without it renders in the
fallback palette while every other page is branded, which reads as a bug rather than a missing
setting.

## Paths

- `/` serves `dist/index.html`; an extensionless path serves the matching `.html`, so `/sign_in` is
  `dist/sign_in.html`. Keep links extensionless.
- Asset references (`src=` / `href=` pointing at a path with a file extension) are rewritten to
  `/_v/<revision>/…` when a portal is pinned to a revision, so assets can be cached
  indefinitely. Write them as ordinary absolute paths — `/assets/style.css` — and leave the
  rewriting alone.
- Absolute URLs are never rewritten, which is why the components bundle is referenced at its
  full CDN URL. There is no build or deploy step: the file is served as it stands here, so it
  has to contain the final URL.

## Serving from a subdirectory

**Every portal repository must have a `solar-assistant.json` at its root.** It is required, not
optional, and it has exactly one property:

```json
{ "root": "dist" }
```

`root` names the directory that gets served. Everything under it is your site; **anything outside it
is not served at all**. That is what keeps this README, `docs/`, `package.json` and your sources in
the same repository without them being reachable on your portal — it is why the file you are reading
is not on yours.

Rename `dist` to whatever suits you — `public`, `site`, `build` — and change the file to match. If
you want your pages at the repository root, say so explicitly:

```json
{ "root": "." }
```

Things worth knowing:

- **Nothing builds your site for you.** The files are served exactly as committed, so if a framework
  produces `dist/`, that directory has to be committed too. Most frameworks put it in `.gitignore`
  by default, so this is the step people miss. If the directory you name turns out to be empty, you
  get 404s rather than your old pages back: SolarAssistant will not quietly serve the repository
  root instead, because that would publish your sources at your customers' address.
- **A missing or invalid file is reported to you, not to your customers.** SolarAssistant checks it
  when you choose a template and when you save a staging revision, so a mistake here surfaces while
  you are looking at it, rather than as a broken portal somebody else discovers.
- **`/dist` and `dist` mean the same thing.** A leading slash is fine; the root is always relative to
  the repository. A root pointing outside it — `..`, an absolute path — is refused.
- **`/assets/logo.svg` is unaffected.** It never comes from the repository (see below), so it is not
  looked for under your root and does not need to exist in `dist/`.

Nothing in this template is built — `dist/` is committed source, not output. The name is the one a
framework is most likely to produce, so if you later add a build step that writes into `dist/`, it
already works.

## `assets/logo.svg`

Referenced but **deliberately absent** from this repository. A logo differs for every company, and a
shared template must not carry one. SolarAssistant serves that path from your organization.

Two consequences worth knowing:

- **It is always SVG.** The stored logo is served as `image/svg+xml` whatever its filename, so
  the path stays `logo.svg`. Do not point it at a `.png`.
- **You do not have to upload one.** If you have not, SolarAssistant generates a plain wordmark
  from your organization's name in your primary colour, and that generated logo already switches
  colour in a dark-mode browser. Uploading yours replaces it.

`dist/assets/style.css` sizes it by height, so headers line up whatever shape you upload, with a
`max-width` guard so that an unusually wide logo cannot crowd out the navigation.

## Pages

| File | Serves |
|---|---|
| `dist/index.html` | redirect to `/sign_in`; no styling, nothing to substitute, nothing ever seen |
| `dist/sign_in.html` | `<sa-sign-in>` |
| `dist/register.html` | `<sa-register>` |
| `dist/sites.html` | `<sa-sites>` — the site list and site detail |
| `dist/user.html` | `<sa-user>` — account details |
| `dist/terms.html` | your terms, framed from your own domain — see below |
| `dist/privacy.html` | the SolarAssistant privacy policy, framed the same way |

Request paths are unchanged by the root: `/sign_in`, not `/dist/sign_in`.

**The two legal pages are wrappers, not documents.** Each is an ordinary page of yours — your
header, your heading, your styling — around an `<iframe>` that SolarAssistant serves **on your own
domain**, so a customer reading your terms never leaves your site. The registration form links
`/terms`, so the page has to exist: delete it and your customers meet a 404 at the moment they are
creating an account.

If you would rather write your own terms, replace the frame with your text and keep the page. Your
terms are your document — we supply one so that you have something correct to start from, and the
obligations it has to satisfy live in your partner agreement, not in this repository.

## Publishing

A push does not change any live portal. SolarAssistant pins each organization to a revision;
a new commit becomes visible only when someone refreshes staging and promotes it.
