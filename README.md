# jaaicode-site

Source for [jaaicode.org](https://jaaicode.org).

Built with [MkDocs](https://www.mkdocs.org/) + the
[Material theme](https://squidfunk.github.io/mkdocs-material/).
Deployed to [Cloudflare Pages](https://pages.cloudflare.com/).

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve
```

Open <http://localhost:8000>.

## Deploy

Cloudflare Pages is wired to this repo's `main` branch. Every push
deploys automatically. Build settings on Cloudflare:

- **Build command:** `pip install -r requirements.txt && mkdocs build`
- **Build output directory:** `site`
- **Environment variable:** `PYTHON_VERSION=3.12`
