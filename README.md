# Coding Aloud

Source for **Coding Aloud** / *Programando en voz alta*, the personal professional blog of Ricardo
Sotolongo, published at <https://blog.ricardosotolongo.com>.

The site is built with [Hugo](https://gohugo.io/) and served by GitHub Pages from the `docs/`
folder of the `main` branch. Posts cover software engineering (.NET, MSBuild, NuGet, OpenAPI,
deployment) and, in Spanish, a long-running series of professional memoirs.

## Repository layout

| Path          | Contents                                                                 |
| ------------- | ------------------------------------------------------------------------ |
| `config.toml` | Site configuration: languages, menus, theme parameters, analytics.       |
| `content/post`| Posts as Markdown, one file per language: `<slug>.en.md`, `<slug>.es.md`. |
| `static/`     | Post images, one folder per post slug (`static/<slug>/01.png`, …).       |
| `archetypes/` | Front-matter template used by `hugo new`.                                |
| `themes/jane` | A vendored, locally customized copy of [hugo-theme-jane](https://github.com/xianmin/hugo-theme-jane). |
| `docs/`       | **Generated** site output (`publishDir`). Do not edit by hand.           |
| `.github/workflows/main.yml` | Build-and-publish workflow.                               |

## Languages

English (`en`) is the default language and Spanish (`es`) is served under `/es/`. A post exists in
a language when a file with that language suffix exists; many posts are Spanish-only. Translations
of the same post share the base filename and `slug`, which is what links them in the language
chooser.

## Local development

Requires Hugo **extended** (the theme compiles Sass). Recent versions work; the workflow uses the
latest release.

```bash
hugo server -D
```

Then open <http://localhost:1313>. `-D` includes drafts.

To build the static site into `docs/` exactly as CI does:

```bash
hugo
```

## Writing a post

Create the file from the archetype:

```bash
hugo new content/post/my-post.en.md
```

Then complete the front matter:

```yaml
---
title: "My post"
date: 2026-10-29
draft: false
slug: my-post
tags: ["Visual Studio", "MSBuild"]
categories: ["Development"]
---
```

- Use `Development` / `General` for English categories and `Desarrollo` / `General` / `Memorias`
  for Spanish ones, so posts land in the existing category pages.
- Put images in `static/my-post/` and reference them with an absolute path:
  `![Caption](/my-post/01.png)`.
- For a Spanish version, add `content/post/my-post.es.md` with the same `slug`.
- The archetype creates posts with `draft: true`; set it to `false` to publish.

## Publishing

Pushing to `main` triggers the **Hugo Publish** workflow, which:

1. Checks out the repository and installs the latest Hugo.
2. Runs `hugo`, regenerating `docs/`.
3. Commits any changes in `docs/` as `Published: …` and pushes them back to `main`.

GitHub Pages then serves `docs/`. Because CI regenerates and commits the output, committing a local
build of `docs/` is optional; pull before pushing again, since CI adds a commit after each push.

`docs/CNAME` holds the custom domain and exists only in `docs/`. Hugo does not clean the publish
directory, so it survives builds, but do not delete `docs/` wholesale or the domain mapping is lost.

## Theme customizations

`themes/jane` is committed directly rather than as a submodule, because it carries local changes:
logo font and "modern" logo styling (desktop and mobile) and a Google Analytics 4 partial. Updating
the theme from upstream means re-applying those changes.

## License

The theme is under its own license in `themes/jane/LICENSE.md`. Post content is © Ricardo
Sotolongo; all rights reserved unless a post states otherwise.
