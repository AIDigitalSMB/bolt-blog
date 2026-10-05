# Working in this repository

A solo-author static blog: Astro 7 + TypeScript. One of six sibling blogs for the
AIDigitalSMB brand, each with the same feature set, independent content and git history,
and a deliberately different visual design (this one is a bento-grid / glassmorphism
design intended for Bolt.new hosting). Structurally derived from an earlier sibling
project for a different brand, then fully rebranded and re-initialised with a fresh git
history — no shared content or history carried over. Full detail in
[`docs/architecture.md`](docs/architecture.md).

**Hosting is meant to be Bolt.new/bolt.host**, but **this repo has not been imported into
Bolt.new or published**. `astro.config.mjs`'s `site` (and the matching values in
`public/admin/config.yml` and `public/robots.txt`) currently point at the placeholder
domain `aidigitalsmb.bolt.host` rather than a real project URL — update all three once
Bolt assigns one. The repo is named `bolt-blog` because it deploys through Bolt.new.
`SIBLING_SITES` in `src/consts.ts` is intentionally an empty array until all six siblings
are deployed; the footer renders no sibling row while it is empty.

**Pagefind snapshot.** Bolt.new's Publish does not run the `postbuild` Pagefind step, so
the search index is committed as a static snapshot in `public/pagefind/`. **Refresh that
snapshot before every Bolt Publish that changes post content** (`npm run build`, then
copy `dist/pagefind/` over `public/pagefind/` and commit) — see
[`docs/deployment.md`](docs/deployment.md).

Expect this Bolt.new behaviour once/if this repo is imported and published (observed on
the earlier sibling project this was derived from, not yet verified here): Bolt.new →
GitHub push-back is real — it commits a `package-lock.json` update from its own `npm install` shortly after import — but the
reverse does **not** hold. A `git push` made outside Bolt (e.g. a CMS save) does **not**
get pulled in and redeployed automatically; it needs a manual reopen-and-Publish inside
Bolt's editor. See [`docs/deployment.md`](docs/deployment.md).

## Development

Start the dev server in background mode:

```
astro dev --background
```

Manage it with `astro dev stop`, `astro dev status`, `astro dev logs`.

```bash
npm run dev      # localhost:4321/ — drafts visible
npm run build    # production build + Pagefind index
npm run preview  # serves dist/ — the only faithful test of search and base paths
npm run check    # TypeScript + Astro diagnostics; keep this at 0 errors
```

**On this Windows machine**, Smart App Control blocks Astro's native compiler binary.
After every `npm install` or `npm ci`:

```bash
npm install --no-save --force @astrojs/compiler-binding-wasm32-wasi
```

## Rules that are easy to get wrong

**Never write a root-absolute internal path.** It happens to work today — this is a
root-served site with `base: '/'` — but that is hosting, not design. Use the helpers in
`src/lib/url.ts` so the site can move again without a rewrite:

| Helper | For |
| --- | --- |
| `withBase(p)` | paths you author — `/about/` → `/about/` here, `/blog/about/` under a base |
| `absFromBuiltPath(p, site)` | paths Astro produced (`Astro.url.pathname`, `ImageMetadata.src`, `paginate()` URLs) — **already based** |
| `absUrl(p, site)` | absolute URL from a path you author |

Do not try to collapse these into one function that detects whether a path "already has
the base". That was tried on a sibling project and shipped two bugs: `/blog/my-post/`
is genuinely ambiguous because a `/blog/` route sits under a `/blog/` base.

**`paginate()` URLs already include the base.** Passing them through `withBase()`
doubles it the moment a base is configured. `Pagination.astro` takes them raw.

**Query posts through `getPosts()`** in `src/lib/posts.ts`, never `getCollection`
directly. That single call is where drafts are filtered out of production builds and
where date ordering happens.

**Frontmatter image paths are relative to the Markdown file** —
`../../assets/images/uploads/…` — because `image()` resolves them that way and the CMS
is configured to write exactly that. Both `media_folder` and `public_folder` in
`public/admin/config.yml` must stay in sync with wherever posts live.

**Site-wide settings live in `src/consts.ts` and nowhere else.** If you find yourself
hardcoding a title, an author name or a page size, put it there instead.

**The CMS schema and the Zod schema must match.** `public/admin/config.yml` field names
and `src/content.config.ts` are one contract; changing either alone breaks editing or
breaks the build.

**`public/admin/config.yml` is YAML.** Quote any string containing `: ` — an unquoted
colon-space silently breaks the whole CMS.

**The content repo (GitHub) and the host are meant to stay two separate systems**,
bridged by whatever sync the host offers — each sibling uses its own host's mechanism
(dashboard Git integration, GitHub Actions, GitLab CI, a v0.app→Vercel connection).
Sveltia CMS commits to the GitHub repo named in `public/admin/config.yml` regardless.
**For this sibling specifically, expect a CMS save NOT to reach the live site by
itself** — on the earlier sibling this was derived from, Bolt.new pulled from GitHub only
when its own editor was reopened and Publish was clicked again (not yet verified for this
repo, which has not been published). Plan to republish manually in Bolt after every CMS
save, and refresh the `public/pagefind/` snapshot first if post content changed.

## Before calling a change done

```bash
npm run check    # expect 0 errors
npm run build
grep -rhoE 'https?://[^"< ]+' dist --include=*.html | grep -v 'aidigitalsmb.bolt.host' | sort -u
```

The grep must print only genuinely external URLs (giscus, google maps, unpkg). If the
change is visible in a browser, verify with `npm run preview` rather than `npm run dev`
— search, `/admin/` and draft exclusion all behave differently between the two.

## Deployment

**Not yet live.** This repo has not been imported into Bolt.new or published. Once it
is: pushing to `main` — including a CMS save — is expected **not** to trigger a rebuild by
itself; someone has to reopen the Bolt.new project and click Publish again, the same manual
republish behaviour observed on the earlier sibling this was derived from (unverified
for this repo). See [`docs/deployment.md`](docs/deployment.md).

Local git authenticates as `AIDigitalSMB`, the GitHub account that owns every AIDigitalSMB
sibling repo — that account's access would be unrelated to whoever controls the Bolt.new
project itself, once one exists.

## Documentation

Full docs: https://docs.astro.build

- [Routing and dynamic routes](https://docs.astro.build/en/guides/routing/)
- [Content collections](https://docs.astro.build/en/guides/content-collections/)
- [Images](https://docs.astro.build/en/guides/images/)
- [Astro components](https://docs.astro.build/en/basics/astro-components/)

Bolt.new-specific: https://support.bolt.new/
