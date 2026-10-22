# pearsonty9.github.io

Personal portfolio for Tyler Pearson, built with Jekyll and hosted on GitHub Pages.

## Editing content

All content lives in data files, so no HTML changes are needed:

| File | What it controls |
| --- | --- |
| `_config.yml` | Name, tagline, SEO description, social links |
| `_data/profile.yml` | Bio paragraphs, skills, education |
| `_data/experience.yml` | Work history (newest first) |
| `_data/projects.yml` | Project cards (`featured: true` sorts to the top) |

Project cover images go in `assets/img/games/`.

## Writing blog posts

1. Create `_posts/YYYY-MM-DD-your-title.md` (see `_drafts/example-post.md` for front matter and formatting).
2. Put images in `assets/img/blog/`.
3. Keep unfinished posts in `_drafts/` (no date in the filename); they are never published.
   Preview them with `bundle exec jekyll serve --drafts`.

Posts appear at `/blog/`, in the home page's Writing section (latest 3), and in the RSS feed at `/feed.xml`.

## Running locally

Requires Ruby 3.x+ (`brew install ruby`, then add `/opt/homebrew/opt/ruby/bin` to your PATH).

```bash
bundle install
bundle exec jekyll serve --livereload
```

Then open http://localhost:4000.

## Deploying

Push to `master`. The workflow in `.github/workflows/pages.yml` builds the site
with Jekyll 4 and deploys it. One-time setup: repo Settings → Pages → Build and
deployment → Source: **GitHub Actions**.
