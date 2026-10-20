# bessa75.github.io

Personal academic website, built as a plain Jekyll site (custom minimal layout,
no external theme). GitHub Pages builds and serves it automatically — no CI
needed.

## Structure

- `index.md` — home page / bio
- `publications.md` — publications list
- `cv.md` — CV page, embeds `assets/CV.pdf`
- `blog.md` — blog index (lists `_posts/`)
- `_posts/` — blog posts, one Markdown file per post
- `_layouts/`, `assets/style.css` — custom minimal academic styling

## Editing

- To add a blog post: add a file to `_posts/` named `YYYY-MM-DD-title.md` with
  front matter `layout: post`, `title`, `date`.
- To update the CV: replace `assets/CV.pdf`.
- To update publications: edit `publications.md` directly.

## Preview locally (optional)

```
gem install bundler jekyll
bundle init && bundle add jekyll
bundle exec jekyll serve
```

Then open http://localhost:4000.

## Deploy

Push to the `main` branch of `bessa75/bessa75.github.io` on GitHub, enable
GitHub Pages in the repo settings (Settings → Pages → Source: `main` branch,
`/ (root)`), and the site is live at https://bessa75.github.io within a
minute or two.
