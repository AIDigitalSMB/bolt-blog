# AIDigitalSMB — Bolt Blog

A static blog for a single author. Astro + TypeScript, Markdown content collections,
Sveltia CMS at `/admin`, Pagefind search, Giscus comments. Designed to be hosted via
Bolt.new/bolt.host — but **this repo has not itself been imported into Bolt.new or
published**; see [Status](#status) below.

No server, no database, no tracking scripts, no cookie banner, no CSS framework. Three
runtime dependencies. The only client-side JavaScript is a theme toggle, a copy-link
button, the search page and the comment widget.

This is one of six sibling blogs for the AIDigitalSMB brand — same feature set, independent
content and git history, each with a deliberately different visual design and a different
host. This one is the bento-grid / glassmorphism design intended for Bolt.new. Cross-links
between the siblings are not wired up yet (`SIBLING_SITES` in `src/consts.ts` is empty)
and will be added once all of them are deployed.

## Documentation

| Document | What it covers |
| --- | --- |
| [docs/setup.md](docs/setup.md) | One-time setup — GitHub repo (done), Bolt.new hosting (not done), CMS token, Giscus, contact form, author details |
| [docs/writing.md](docs/writing.md) | Writing and publishing posts, frontmatter reference, drafts, images |
| [docs/architecture.md](docs/architecture.md) | How the site is built and why the awkward parts are that way |
| [docs/deployment.md](docs/deployment.md) | CI/CD — Bolt.new/bolt.host, including what's still unverified about it |
| [docs/troubleshooting.md](docs/troubleshooting.md) | Common failures, with the fix |
| [SECURITY.md](SECURITY.md) | Threat model, token hygiene, why deletion is not erasure |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | Ground rules for comments |
| [CLAUDE.md](CLAUDE.md) | Conventions for anyone (or anything) editing this repository |

## Commands

| Command | What it does |
| --- | --- |
| `npm install` | Install dependencies |
| `npm run dev` | Dev server at http://localhost:4321/ — **drafts visible** |
| `npm run build` | Production build into `dist/`, then Pagefind indexes it (`postbuild`) |
| `npm run preview` | Serve `dist/` — the only faithful test of search, `/admin/` and base paths |
| `npm run check` | TypeScript + Astro diagnostics |

### Windows: Smart App Control

If a build fails with *"An Application Control policy has blocked this file"*, Windows
Smart App Control is refusing Astro's unsigned native compiler binary. Install the WASI
fallback:

```bash
npm install --no-save --force @astrojs/compiler-binding-wasm32-wasi
```

Re-run it after every `npm install` or `npm ci`. CI on Linux is unaffected. Details in
[troubleshooting.md](docs/troubleshooting.md).

## Writing a post

Open `/admin/` → **New Post** → uncheck **Draft** → **Save**. That commits to `main` on
GitHub, but is not expected to update a live bolt.host site by itself (the site is not
published yet) — reopen the Bolt.new project and click **Publish** to actually ship it. See
[docs/deployment.md](docs/deployment.md).

Or write the file directly — posts are Markdown in `src/content/blog/`, and the filename
is the URL slug:

```yaml
---
title: "Post title"
description: "One or two sentences, used for cards, meta description and social."
date: "2026-09-02T09:00:00.000Z"
author: "Your Name"
featured_image: "../../assets/images/uploads/something.jpg"
category: "Engineering"
tags: [astro, performance]
draft: false
---
```

Frontmatter is validated at build time — a typo fails the build instead of shipping
broken output, and a failed build leaves the previous version live.

`draft: true` posts render in `npm run dev` and are dropped entirely from production:
no page, no feed entry, no archive listing, no search hit. Full reference in
[writing.md](docs/writing.md).

## Base-path safety

`base` is currently `/` (a guess pending an actual host) and a root-absolute `/foo/`
link happens to work. That is a coincidence of the placeholder config, not a licence to
hardcode paths: every internal link still goes through [`src/lib/url.ts`](src/lib/url.ts)
— `withBase()` for paths you author, `absFromBuiltPath()` for paths Astro produced,
`absUrl()` for absolute URLs.

To verify after any change, build and confirm every absolute URL points at this site:

```bash
grep -rhoE 'https?://[^"< ]+' dist --include=*.html --include=*.xml   | grep -v 'aidigitalsmb.bolt.host' | sort -u
```

## Status

An AIDigitalSMB sibling blog, started from an earlier sibling project's codebase and
re-initialised with its own fresh git history — see [`CLAUDE.md`](CLAUDE.md).

- [x] GitHub repository created and pushed — public, `AIDigitalSMB/bolt-blog`
- [ ] Not yet imported into Bolt.new or published. `astro.config.mjs`,
      `public/admin/config.yml` and `robots.txt` currently point at the placeholder
      domain `aidigitalsmb.bolt.host` (this project family's brand-word.host-tld
      convention) rather than a real Bolt.new project URL — update all three once one
      exists, see [docs/setup.md](docs/setup.md#1-boltnew-import-and-publish)
- [ ] CMS access token, Giscus IDs, contact form endpoint, author details in `src/consts.ts`
      (`AUTHOR_EMAIL` is the placeholder `hello@aidigitalsmb.com`, not a confirmed mailbox)

**Search snapshot.** Bolt.new's Publish does not run the Pagefind `postbuild` step, so
`public/pagefind/` holds a committed snapshot of the search index. Refresh it
(`npm run build`, then copy `dist/pagefind/` over `public/pagefind/` and commit) before
every Bolt Publish that changes post content — see
[docs/deployment.md](docs/deployment.md).

The site also builds and runs locally (`npm run dev` / `npm run build` / `npm run
preview`) independent of all of the above. Full checklist in
[docs/setup.md](docs/setup.md).
