# joshfrye.com

Personal site built with Jekyll 4. Deployed on Cloudflare Pages.

## Requirements

- Ruby 4.0+ (managed via [mise](https://mise.jdx.dev/) or rbenv)
- Bundler (`gem install bundler`)

## Development

Install dependencies:

```sh
bundle install
```

Start the local server with live reload:

```sh
bundle exec jekyll serve --livereload
```

Site is available at `http://localhost:4000`. Jekyll watches for file changes and rebuilds automatically; the browser reloads on save.

To preview with draft posts included:

```sh
bundle exec jekyll serve --livereload --drafts
```

## Build

Generate the static site into `_site/`:

```sh
bundle exec jekyll build
```

For a production build (also what Cloudflare Pages runs):

```sh
JEKYLL_ENV=production bundle exec jekyll build
```

The `_site/` directory is not committed — it is generated on deploy.

## Content

### Blog posts

Create a file in `_posts/` named `YYYY-MM-DD-slug.md`:

```yaml
---
title: "Post title"
date: 2026-05-17
categories: [platform-devops]       # one of: software-engineering, platform-devops, cars, hardware-hacking
tags: [kubernetes, networking]
excerpt: "One-sentence teaser shown on index pages."
---

Post body here.
```

### TIL entries

Create a file in `_til/` named `YYYY-MM-DD-slug.md`:

```yaml
---
title: "Short descriptive title"
date: 2026-05-17
lang: bash                          # required — drives the language filter on /til/
tags: [shell, cli]
---

One to three sentence explainer.

```bash
# code snippet here
```
```

## Deploy

The site deploys automatically via Cloudflare Pages on every push to `main`.

**Cloudflare Pages settings:**

| Setting | Value |
|---|---|
| Build command | `bundle exec jekyll build` |
| Build output directory | `_site` |
| Root directory | `/` |
| Environment variable | `JEKYLL_ENV=production` |

**First-time setup:**

1. Push the repo to GitHub.
2. In the Cloudflare dashboard, go to **Workers & Pages → Create → Pages → Connect to Git**.
3. Select the repo and enter the build settings above.
4. Add the custom domain `joshfrye.com` under **Custom domains**.
5. Cloudflare will provision the SSL cert and update DNS automatically if the domain's nameservers point to Cloudflare.

Subsequent deploys happen on push — no manual steps.
