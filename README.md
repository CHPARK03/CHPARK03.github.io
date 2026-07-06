# SASO Dev Log

Source for [chpark03.github.io](https://chpark03.github.io) — a build-in-public dev log & portfolio.

Custom Jekyll site (no theme gem). Deployed by GitHub Actions on push to `main`.

## Structure

- `_posts/YYYY-MM-DD-slug.md` — dev log posts (Korean-first, English version included)
- `_projects/*.md` — portfolio case studies (front-matter driven: `steps`, `status`, `links`)
- `_layouts/` — custom layouts (`default`, `home`, `post`, `page`, `project`)
- `assets/css/main.css` — full design system (dark default, `prefers-color-scheme` light)
- `index.md` — home (hero + selected work + recent dev log)

## Local preview

```powershell
bundle install
bundle exec jekyll serve
```
