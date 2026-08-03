# Saso Studio — portfolio hub & dev log

Source for [saso-studio.vercel.app](https://saso-studio.vercel.app) — studio intro, selected work (case studies), and a build-in-public dev log, all in one site.

Custom Jekyll site (no theme gem). Deployed by **Vercel** (project `saso-studio`, repo-connected, push to `main` → production).
The old address [chpark03.github.io](https://chpark03.github.io) is frozen as redirect stubs (`redirects/` + GitHub Actions).

## Structure

- `index.md` — home (studio intro + selected work + recent dev log)
- `_posts/YYYY-MM-DD-slug.md` — dev log posts (Korean-first, English version included)
- `_projects/*.md` — portfolio case studies (front-matter driven: `steps`, `status`, `links`)
- `_layouts/` — custom layouts (`default`, `home`, `post`, `page`, `project`)
- `assets/css/main.css` — full design system ("검증 원장" ledger: paper light single theme; seal-stamp verdict motif)
- `redirects/` — static redirect stubs served at chpark03.github.io (excluded from Jekyll build)
- `vercel.json` — Vercel build config (`bundle exec jekyll build` → `_site`)

## Local preview

```powershell
bundle install
bundle exec jekyll serve
```
