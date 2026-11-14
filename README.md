# afavaccio.cloud

Personal site built with [Hugo](https://gohugo.io/), styled as a bash terminal.
Pages are static HTML with normal links; a small script adds a prompt where
`help`, `ls`, `cd`, `cat` and a few other commands work.

## Local preview

```bash
hugo server
```

Then open <http://localhost:1313>. Requires Hugo 0.146 or newer (built with 0.169.0).

## Writing content

| What | Where |
|---|---|
| Home intro | `content/_index.md` |
| About / CV | `content/about.md` |
| Contact links | `content/contact.md` |
| A project | new file in `content/projects/` |
| A blog post | new file in `content/blog/` |

A blog post needs this front matter:

```yaml
---
title: "Post title"
description: "One-line summary."
date: 2026-11-12
tags: ["tag"]
---
```

Projects also accept `repo` and `demo` URLs, shown as links under the title.
The file name becomes the name shown by `ls`, so keep it short and lowercase.

Two shortcodes are available in any Markdown file:

- `{{< cmd "kubectl get nodes" >}}` renders a command the visitor can click to
  run in the prompt (used by `content/try-me.md`).
- `{{< social icon="github" name="GitHub" url="https://github.com/hyesr" handle="hyesr" >}}`
  renders a contact row with an icon (`github`, `linkedin` or `x`).

## Customising

- Prompt user and host, site description: `hugo.toml` under `[params]`.
- Top navigation: `[[menus.main]]` in `hugo.toml`.
- Colours: the variables at the top of `assets/css/main.css`.
- Code highlighting: regenerate with
  `hugo gen chromastyles --style=catppuccin-mocha > assets/css/syntax.css`.
- Commands: the `commands` object in `assets/js/terminal.js`.

## Deploy (GitHub Pages)

Every push to `main` runs `.github/workflows/hugo.yml`, which builds the site
with Hugo and publishes it to GitHub Pages. If the build fails nothing is
deployed and the previous version stays online.

The custom domain comes from `static/CNAME`. Its DNS records point to GitHub
Pages:

| Type | Name | Value |
|---|---|---|
| A | `@` | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153` |
| CNAME | `www` | `hyesr.github.io` |
