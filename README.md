# certctl.io

Source for the [certctl.io](https://certctl.io) landing page.

Single-file static site (`index.html`) plus a logo, favicon, and four
dashboard screenshots. No build step. No JavaScript dependencies — one
tiny inline `<script>` fetches the latest [certctl](https://github.com/certctl-io/certctl)
release tag from the GitHub API for the version chip in the hero, and
falls back gracefully if the API isn't reachable.

## Local preview

```bash
python3 -m http.server 8080
# then open http://localhost:8080/
```

## Hosting

Cloudflare Pages, deployed automatically on push to `main`. Every commit
to this repo triggers a redeploy that finishes globally in under a minute.

## Editing

Everything lives in `index.html`. CSS variables at the top of the
`<style>` block control colors, typography, and spacing site-wide.
Sections are delimited by HTML comments matching the eyebrow numbering
on each section.

After any edit:

```bash
git add -A
git commit -m "site: <what changed>"
git push
```

## Issues / suggestions

For anything site-related, open an issue on this repo. For questions
about certctl itself (the product), use the
[main repo's discussions](https://github.com/certctl-io/certctl/discussions).

## License

BSL 1.1 — same as the parent [certctl](https://github.com/certctl-io/certctl) project.
