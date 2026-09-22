# Blog Maintenance Guide

## Overview

This is a Jekyll blog using the **Chirpy** theme (`jekyll-theme-chirpy ~> 7.5`, installed 7.5.0),
deployed to Cloudflare Pages (primary, `tatos.net`) **and** mirrored to a GitHub Pages backup
(`tato418.github.io`). Site identity lives in `_config.yml`:

- **Title:** Escape from Shell
- **Tagline:** Breaking out. One shell at a time.
- **URL:** https://tatos.net
- **Author (social):** Tato (LinkedIn, GitHub `tato418`)
- **Theme mode:** dark, `lang: en`, `toc: true`
- **Page views:** `pageviews.provider: goatcounter` (ID left empty in `_config.yml`)
- **Comments:** provider unset = disabled

Content lives in `_posts/`, organized into subfolders. Post filenames are
`YYYY-MM-DD-slug-title.md`.

## Post Locations

| Directory | Content |
|---|---|
| `_posts/*.md` | Technical / DevSecOps articles, older standalone posts |
| `_posts/ctf/` | TryHackMe (THM) CTF writeups |
| `_posts/personal/` | Life / off-topic posts |

> `_config.yml` `defaults` auto-assign `categories` + `hidden: true` + `permalink`
> to files in `_posts/ctf` and `_posts/personal`, so a post's front matter and the
> config both declare the same values. Keep them consistent.

## Categories

Categories are **two-level**: one top-level category + one subcategory, declared in an
array. The values actually used by live posts:

| Post (examples) | `categories:` value |
|---|---|
| CTF writeups (all THM) | `[Cybersecurity, CTF]` |
| DevSecOps articles (code review, golang patching) | `[Cybersecurity, DevSecOps]` |
| Tool guides (CUDA for Hashcat) | `[Cybersecurity, Tools]` |
| Personal posts (Audiobook) | `[Personal, Books]` |

```yaml
categories: [Cybersecurity, DevSecOps]
categories: [Personal, Books]
```

> **Legacy outlier:** the 2023 post *Accessing Kubernetes Services Using Cloudflare
> Zero Trust* uses `[DevSecOps, Kubernetes]` — an older scheme where `DevSecOps` was a
> top-level category. New articles should prefix with `Cybersecurity` as shown above.

## Tags (Granular Topics)

`tags` are granular, lowercase, hyphenated, comma-separated in an array. Always tag
TryHackMe writeups with `thm`.

```yaml
tags: [golang, devsecops, sca]
tags: [thm, sqli, web, python, ssh]
tags: [kubernetes, cloudflare, zero-trust, devops, security]
```

## Creating a New Post

### 1. File location + name

```
_posts/2026-03-18-new-post-title.md          # article
_posts/ctf/2026-05-25-thm-room-slug.md       # CTF writeup
_posts/personal/2026-06-01-post-slug.md      # personal
```

The subfolder is what routes a post into its tab/permalink, not the category label.

### 2. Front Matter

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS +0200
categories: [CATEGORY, SUBCATEGORY]
tags: [tag1, tag2, tag3]
description: A one-sentence summary of the post.
image:
  path: /assets/img/commons/<header.webp>
  alt: Post Title
toc: true
---
```

- Use **`description`** (SEO summary). Do NOT use `excerpt` — the theme and all live
  posts use `description`.
- `date` format `YYYY-MM-DD HH:MM:SS ±ZZZZ` (full seconds). Offsets vary by post
  (`+0000`, `+0100`, `+0200`, `-0400`) — `timezone:` is intentionally left empty in
  `_config.yml`, so the per-post offset is the source of truth. Keep the author's offset.
- CTF writeups also set `hidden: true`.

### 3. Header Images

Available headers in `assets/img/commons/` (these are the real files on disk):

| Image | Use for |
|---|---|
| `header_post_writeup.webp` | CTF writeups |
| `header_post_article_devsecops.webp` | DevSecOps / technical articles |
| `security_code_review_python_head.webp` | Code-review / dependency-scan (SCA) articles |
| `header_post_life.webp` | Personal posts |

Post-specific screenshots live in `assets/img/posts/<YYYY-MM-DD-slug>/`; shareable
headers in `assets/img/commons/`.

## CTF Writeup Template

Use `writeup-template.md` as the base for TryHackMe writeups. The canonical front matter
and section skeleton (Enumeration → Initial Foothold → Privilege Escalation → Lessons
Learned → Tools Used → References) live there. Start each writeup with the room card:

```markdown
> **Room:** [<ROOM>](https://tryhackme.com/room/<slug>)
> **Difficulty:** <Easy|Medium|Hard> | **Points:** <N>
{: .prompt-info }
```

## Chirpy Theme Features

### Callout Blocks

```markdown
> Info or neutral context.
{: .prompt-info }

> Warning or key insight.
{: .prompt-warning }

> Danger or critical note.
{: .prompt-danger }

> Success, discovered cred, or flag.
{: .prompt-success }

> Better-solution / takeaway tip.
{: .prompt-tip }
```

Put the `{: .prompt-* }` line directly under the blockquote (including multi-line
blockquotes).

### Code Blocks

Drop line numbers on shell output with `.nolineno` directly under the fence:

````markdown
```bash
nmap -sCV --min-rate=1500 -p- 10.10.x.x
```
{: .nolineno }
````

### Images

```markdown
![Description](/assets/img/posts/YYYY-MM-DD-slug/filename.webp){: .shadow }
```

Past posts also use sizing attributes: `{: w="700" h="400" }`.

### Media Embeds

- YouTube: `{% include embed/youtube.html id='...' %}`
- Spotify: raw `<iframe>` with `src="https://open.spotify.com/embed/..."`

## Custom Layouts & Tabs

The site overrides/extends Chirpy with hand-written files — treat these as owned code,
never clobbered by a theme upgrade:

| File | Purpose |
|---|---|
| `_layouts/ctf.html` | Card list + frontend pagination for `/ctfs/` |
| `_layouts/personal.html` | Card list + frontend pagination for `/personal/` |
| `_tabs/ctfs.md` | CTF tab (`layout: ctf`) |
| `_tabs/personal.md` | Personal tab (`layout: personal`) |

## Building and Testing

```bash
# Install dependencies (first time)
bundle install

# Build the site
bundle exec jekyll build

# Serve locally (with live reload)
bundle exec jekyll serve --livereload
```

Plugins: `jekyll-theme-chirpy`, `html-proofer`, `jekyll-compose`, `jemoji`.

## Deployment

The site is deployed to **two** hosts, both triggered by a push to `main`:

### 1. Primary — Cloudflare Pages (`tatos.net`)
Deploys via **Cloudflare Pages** on push to `main` (project connected directly to this
repo; CF Pages runs the Jekyll build). Pushes only trigger the build if the project's
production branch is `main` and auto-deploys are on; verify in the Cloudflare Pages
dashboard if a push doesn't go live. The stale `.github/workflows/pages-deploy.yml`
was removed in `d48c741` (Mar 2026).

### 2. Backup mirror — GitHub Pages (`tato418.github.io`)
Since 2026-09-22 the built site is ALSO mirrored to GitHub Pages at
`https://tato418.github.io/` (repo `Tato418/tato418.github.io`) as a backup of
tatos.net. `.github/workflows/gh-pages-mirror.yml` runs on push to `main` (same
trigger as CF Pages).

- **We deliberately do NOT use GitHub's built-in Pages builder** — it compiles with
  `--safe`, which silently ignores `_plugins/` (e.g. `posts-lastmod-hook.rb`, which
  bakes git-history `last_modified_at`) and would diverge from the CF build. Instead
  the mirror workflow runs its **own** `bundle exec jekyll build` (plugins enabled,
  `fetch-depth: 0` for git history, dest `_site_mirror`) and pushes the static
  artifact to the mirror repo's `main` via `peaceiris/actions-gh-pages` with
  `enable_jekyll: false` — CRITICAL: false (the default) makes the action write an
  EMPTY `.nojekyll` at the repo root, which tells GitHub's built-in
  `pages-build-deployment` to serve the static artifact and SKIP its own Jekyll run.
  Setting it `true` is inverted: GitHub then re-runs Jekyll in `--safe` mode, which
  fails on theme includes (e.g. `embed/youtube.html`) — that was the actual failure
  on the tato418.github.io mirror. If `.nojekyll` is ever absent from the mirror repo
  root, the mirror breaks exactly that way.
- **Credential:** requires repo secret `GH_PAGES_DEPLOY_KEY` — the private half of a
  write-scoped deploy key created on `Tato418/tato418.github.io`; the action needs
  push on that repo.
- **Config:** `url` stays `https://tatos.net`, `baseurl` empty → the mirror's
  canonical/sitemap keep pointing at the primary, so it won't compete for SEO; both
  sites serve at root.

**Do not treat the deploy as "CF only"** — a push to `main` runs BOTH. After a push,
check the GitHub Pages mirror as well as the live CF site.

## Newsletter — Buttondown (subscribe card + new-post automation)

The blog has a Buttondown newsletter integration (`username: tatosnet`) built on **free
tier**. Two moving parts:

### 1. Author/subscribe card (front-end)
Injected at the **end of every post** by JS in `_includes/metadata-hook.html` (posts
only — the code checks for `article .post-tail-wrapper`, so home/archive pages stay
clean). The card shows the avatar, name (**Tato**), role, a **LinkedIn** button
(hard-coded to `https://www.linkedin.com/in/cgutierrez01`) and a Buttondown embed form
posting to `https://buttondown.com/api/emails/embed-subscribe/tatosnet` with hidden
`embed=1`. Styling lives in `assets/css/jekyll-theme-chirpy.scss` under `.tato-author-card*`
— uses the theme's CSS vars, so it adapts to light/dark; the form wraps to a full-width
button on narrow screens (`@media (min-width: 480px)`).

### 2. New-post → draft email automation (CI)
`.github/workflows/buttondown-draft.yml` runs on push to `main` touching `_posts/**`,
diffs against the previous commit, and for each newly-added post calls
`.github/scripts/buttondown_draft.py` to POST a **draft** email to Buttondown.

- **Drafts only, never auto-sends.** The `status: draft` payload means the email lands
  in the dashboard for manual review + send. There is intentionally NO publish call.
- **Secrets:** requires repo secret `BUTTONDOWN_API_KEY` (from
  https://buttondown.com/settings/developers). No other credential.
- **Email body** is HTML styled to the blog's Nord dark palette (defined as bare-hex
  constants `DARK_BG`, `CARD_BG`, etc. in the script): dark page, `#3b4252` card, frost-blue
  CTA button. Table layout + inline styles only (Gmail/Outlook strip `<style>`/CSS3).
- **URLs:** mirrors Jekyll permalinks — strips the `YYYY-MM-DD-` date prefix and keeps the
  `ctf/` / `personal/` subdir, so it produces `/posts/<slug>/`, `/posts/ctf/<slug>/`,
  `/posts/personal/<slug>/`.

### Free-tier UI theming
Full custom HTML email templates are a **Professional** feature. On free tier, the
header / footer / accent color / CSS fields live in **Buttondown → Settings → Email**
(web UI, not API). Ready-to-paste snippets live in **`.github/BUTTONDOWN_EMAIL_THEME.md`**
(template: Modern, accent `#88c0d0`, Nord header/footer HTML, optional CSS).

> **Gotchas**
> - The embed form needs NO API key (public endpoint). The API key is only for the CI
>   draft script.
> - Toggling a workflow's `paths:` filter means **editing an existing post does not
>   trigger a draft** — only *added* `_posts/**` files do.
> - Free tier caps the subscriber list at **100**.

## Files Quick Reference

| File/Directory | Purpose |
|---|---|
| `_posts/` (+ `ctf/`, `personal/`) | Blog post Markdown files |
| `_tabs/` | Navigation tabs (About, Archives, Categories, Tags, CTF, Personal) |
| `_layouts/` | Custom `ctf.html` and `personal.html` card layouts |
| `_config.yml` | Site configuration |
| `.github/workflows/buttondown-draft.yml` | CI: new post → Buttondown draft email |
| `.github/workflows/gh-pages-mirror.yml` | CI: push to `main` → mirror built site to GitHub Pages (`tato418.github.io`) |
| `.github/scripts/buttondown_draft.py` | Builds + POSTs the Nord-styled draft email |
| `.github/BUTTONDOWN_EMAIL_THEME.md` | Free-tier Buttondown header/footer/CSS snippets |
| `assets/img/` | Images and assets |
| `writeup-template.md` | CTF writeup template |
| `AGENTS.md` | This guide |

## SEO and Metadata

- `title` / `description` / `tagline` in `_config.yml`
- `description` in post front matter — post metadata + social preview
- `image.path` in post front matter — header/social image

## Common Tasks

### Add a new tag
Add it to the `tags` array in the post front matter. Tags are auto-generated.

### Add a new category
Use a two-level `[Top, Sub]` value in front matter; the categories page auto-updates.
Prefer an existing `Cybersecurity` subcategory over inventing a new top-level one.

### Update about page
Edit `_tabs/about.md`.

### Add image to post
Place it in `assets/img/posts/<YYYY-MM-DD-slug>/` (or `assets/img/commons/` for headers),
reference with `![alt](/assets/img/path/image.webp){: .shadow }`.

### "New version available" update toast
`_includes/metadata-hook.html` also injects an update toast (`.tato-update-toast`).
Every page bakes the build-stamp `loadedVersion = {{site.time | date: "%Y%m%d%H%M%S"}}`;
the script re-fetches the homepage cache-busted (`/?vcheck=<ts>` to bypass browser +
Fastly CDN cache) every 5 min / on tab focus / 3s after load, and shows a Reload toast
whenever the live stamp differs from the page's loaded one. Any push (incl. styling-only)
retriggers it because `site.time` changes per build. No plugins/version.json needed.
If the homepage gets heavy: swap the fetch target to a lighter always-present page that
still contains the `loadedVersion` marker.

### Consistency checks before committing
- `description` (not `excerpt`) present on every post.
- Header `image.path` points at a file that actually exists in `assets/img/`.
- All `](/assets/...` image references resolve to files on disk.
- No empty `## ` headings left over from a template.
