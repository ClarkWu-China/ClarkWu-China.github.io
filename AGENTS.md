# Repository Guidelines

## Project Structure & Module Organization

This repository is an al-folio/Jekyll academic website. Site-wide settings live in `_config.yml`; structured content is stored in `_data/` and `_bibliography/`. Author pages, posts, news, projects, and books belong in `_pages/`, `_posts/`, `_news/`, `_projects/`, and `_books/`. Reusable Liquid markup is split between `_layouts/` and `_includes/`, while custom Ruby extensions live in `_plugins/`. Put Sass partials in `_sass/`, browser scripts in `_scripts/` or `assets/js/`, and static media in the appropriate `assets/` subdirectory. Generated `_site/` content should not be committed.

## Build, Test, and Development Commands

- `bundle install` installs the Ruby/Jekyll dependencies from `Gemfile.lock`.
- `bundle exec jekyll serve` builds the site and serves it locally with rebuilds.
- `bundle exec jekyll build` performs the production-style build used by CI.
- `npm install` installs the pinned Prettier and Liquid plugin dependencies.
- `npx prettier . --check` verifies formatting; use `npx prettier . --write` to fix it.
- `pre-commit run --all-files` checks YAML, whitespace, EOF newlines, and oversized additions.
- `docker compose up --build` runs the development site in Docker when a local Ruby setup is unavailable.

## Coding Style & Naming Conventions

Use two-space indentation in YAML, Liquid, SCSS, and JavaScript, and preserve existing front-matter patterns. Prettier is authoritative, configured for a 150-character line width, ES5 trailing commas, and Liquid formatting. Name posts `YYYY-MM-DD-short-slug.md`; use lowercase, descriptive filenames elsewhere. Keep Liquid components focused and place shared markup in `_includes/` rather than duplicating it across layouts.

## Testing Guidelines

There is no unit-test suite. Treat a clean `bundle exec jekyll build` and `npx prettier . --check` as the minimum validation. For navigation or content changes, inspect the locally served pages at desktop and mobile widths. CI also checks links with Lychee and runs accessibility checks with Axe; avoid introducing broken internal paths, missing alt text, or invalid heading order.

## Commit & Pull Request Guidelines

Recent history uses short, imperative subjects such as `Update about.md` and `Add files via upload`. Prefer a more descriptive equivalent, for example `Update research interests on about page`, and keep each commit focused. Pull requests should explain the change, list validation performed, link the relevant issue for bugs or features, and include before/after screenshots for visual changes. Do not commit secrets, local environment files, or generated build output.
