# A minimal SolarAssistant customer portal

Your customers sign in, see their sites, and manage their account — under your name, your colours
and your logo. SolarAssistant serves this repository directly: a request path maps onto a file
here, and your organization's own details are substituted into each page as it is served.

Nothing in it is specific to any company, and there is no branding to strip out — which is what
makes it safe to start from. It is deliberately the smallest complete portal there is, meant to be
read in one sitting and changed.

## Getting started

You do not need to fork anything to *use* a portal. Choosing this template in SolarAssistant gives
you a working portal in your own branding, and most companies never need more than that.

Fork it when you want to change something — the wording, the layout, an extra page:

1. **Fork this repository** into your own GitHub account or organization.
2. **Connect the fork** on your organization's **Cloud portal** page:

   ```
   https://solar-assistant.io/organizations/<your-organization-id>/portal
   ```

   If you do not know your organization id, start at
   [solar-assistant.io/user](https://solar-assistant.io/user), click your organization, then
   **Cloud portal**. You choose the repository once; after that it is yours.
3. **Set your branding in SolarAssistant**, not in the code. Your name, colours and logo live on
   your organization record and are substituted into the pages — see [Tokens](#tokens). There is
   nothing company-specific to edit here, and a logo is required before a portal can be enabled.
4. **Edit, commit, push.** A push changes nothing on its own: you stage a revision to preview it and
   publish when you are happy. See [Publishing](#publishing).

The same **Cloud portal** page is where you choose a ready-made template, stage a revision to preview
it, and publish. It is the one page to bookmark.

Fuller help pages are coming. For now this README is the documentation.

## Working on it locally

**The faithful preview is staging**, not localhost. Commit, push, then refresh and preview from your
**Cloud portal** page. That is the only place your colours, logo and organization id are actually
substituted, so it is the only place sign-in works and the only place the pages look like what your
customers get.

For quick work on layout and styling, serve `dist/` with any static server **that serves `/sign_in`
as `sign_in.html`**:

```bash
npx serve dist
```

That detail matters more than it sounds. `python3 -m http.server` does *not* do extensionless URLs,
and since `index.html` immediately redirects to `/sign_in`, the first thing you see is a 404 — which
looks like a broken template and is not one.

Three things will differ from the real thing, all expected:

- **No branding.** Tokens are only substituted when SolarAssistant serves the page, so the tab reads
  `{{ org_name }}` and the colours fall back to the neutral palette in `dist/assets/style.css` —
  exactly what an organization with no colours set would see.
- **No logo.** `/assets/logo.svg` 404s locally, because it comes from your organization rather than
  from this repository.
- **No sign-in**, because `organization-id` is still `{{ org_id }}` rather than a number. If you want
  to exercise it, put your real organization id into `dist/sign_in.html` while you work — the API
  accepts requests from any origin, so sign-in genuinely works from localhost. Don't commit that
  change: your branding should keep coming from your organization record, not from the files.

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
  assets/style.css
```

So `/sign_in` is `dist/sign_in.html`, and this README is not reachable on your portal. See
[Serving from a subdirectory](#serving-from-a-subdirectory) if you want to move or rename it.

## Tokens

Four tokens are replaced, **in `.html` files only**:

| Token | Becomes | Used for |
|---|---|---|
| `{{ org_id }}` | the organization's numeric id | `organization-id` on the components |
| `{{ org_name }}` | the organization's name, HTML-escaped | page `<title>`, logo `alt` text |
| `{{ org_primary }}` | its brand colour | `--sa-primary` |
| `{{ org_accent }}` | its soft tint | `--sa-accent` |

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

Referenced but **deliberately absent** from this repository. A logo differs for every company, and a shared
template must not carry one. SolarAssistant serves that path from the organization's stored
logo.

Two consequences worth knowing:

- **It is always SVG.** The stored logo is served as `image/svg+xml` whatever its filename, so
  the path stays `logo.svg`. Do not point it at a `.png`.
- **A portal cannot be enabled without one.** There is no fallback and no placeholder — the
  header expects an image at that path, so uploading a logo is part of setting a portal up
  rather than something to do afterwards.

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

Request paths are unchanged by the root: `/sign_in`, not `/dist/sign_in`.

## Publishing

A push does not change any live portal. SolarAssistant pins each organization to a revision;
a new commit becomes visible only when someone refreshes staging and promotes it.
