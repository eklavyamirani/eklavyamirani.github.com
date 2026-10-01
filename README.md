# eklavyamirani.github.com

Source for <https://github.ekkylab.uk>. GitHub Pages builds it with Jekyll
directly from `master`, so there is no build step or workflow to maintain.

## Editing content

All content lives in `_data/`. You shouldn't need to touch the HTML.

| File | Section |
|------|---------|
| `_data/projects.yml` | Selected work. The first entry is shown as the large card. |
| `_data/lab.yml` | Smaller builds, shown as a compact list. |
| `_data/music.yml` | Violin and guitar recordings, from a YouTube id or an audio file in `assets/media/`. |

**Resume:** commit a PDF as `resume.pdf` at the repo root. The resume buttons
appear on their own once that file exists.

Contact details and the location are set in `_config.yml`.

## Preview locally

```sh
gem install jekyll kramdown-parser-gfm webrick
jekyll serve
```

## History

The original blog (2012–2015) is kept on the `archive/legacy-blog` branch and
the `legacy-blog-2026-09-30` tag.
