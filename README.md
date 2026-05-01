# certctl.io — landing page

Source for [certctl.io](https://certctl.io). Single-file static site
(`index.html`) plus four screenshots, three logo variants, and a favicon.
No build step. No JavaScript dependencies (one tiny inline script that
fetches the latest GitHub release tag — graceful fallback if it fails).
Designed to ship on Cloudflare Pages from the `certctl-io` GitHub org.

---

## Files in this directory

| File                          | Size   | What it is                                                  |
| ----------------------------- | ------ | ----------------------------------------------------------- |
| `index.html`                  | ~37 KB | The whole site — HTML and CSS inline                        |
| `certctl-logo.png`            | 376 KB | Master logo, 886×864. Source-of-truth, not loaded by users  |
| `logo-icon.png`               | 18 KB  | 80×80 icon, used in the header at 32px                      |
| `logo-footer.png`             | 70 KB  | 240×234 full composite, used in the footer at 120px         |
| `favicon-32.png`              | 13 KB  | 64×64 icon-only, used as browser tab favicon                |
| `screenshot-dashboard.png`    | 234 KB | Hero image + dashboard tile                                 |
| `screenshot-certificates.png` | 300 KB | Certificate inventory tile                                  |
| `screenshot-fleet.png`        | 184 KB | Agent fleet tile                                            |
| `screenshot-jobs.png`         | 184 KB | Issuance pipelines tile                                     |
| `README.md`                   | this   | Deployment + edit notes (you are here)                      |

Total: ~1.6 MB. First-paint weight (HTML + above-the-fold images) ~430 KB.

---

## Local preview

```bash
cd /Users/shankar/Desktop/cowork/certctl.io
python3 -m http.server 8080
# then open http://localhost:8080/
```

The page is fully self-contained. The version chip in the hero (`v2.0`)
will hit the live GitHub API on page load and update to the actual
latest release tag. If the API call fails (offline, rate-limited,
firewalled) the static `v2.0` fallback stays on screen.

---

## Step-by-step deploy

This walks you from "files in a folder on my Mac" to "live at
https://certctl.io" with no skipped steps. Total time: ~10 minutes.

### Step 1 — Create the repo on the `certctl-io` org

1. Go to <https://github.com/organizations/certctl-io/repositories/new>
   (you must be logged in as `shankar0123` — you're the org's founding
   member so you have permission).
2. Fill in the form:
   - **Repository name:** `certctl.io`
   - **Description:** `Landing page source for certctl.io`
   - **Visibility:** Public
   - **Initialize this repository with:** leave ALL boxes unchecked.
     Don't add a README, .gitignore, or license — we have those locally.
3. Click **Create repository**.
4. The next page shows you a "quick setup" panel with the repo URL.
   Copy the SSH URL (looks like `git@github.com:certctl-io/certctl.io.git`)
   — you'll need it in Step 2.

### Step 2 — Push this directory to the new repo

In your terminal:

```bash
cd /Users/shankar/Desktop/cowork/certctl.io

# Verify you're in the right place — should list index.html, README.md, screenshots, etc.
ls

# Sanity-check your git identity. Commits will be authored as whoever
# this is — orgs don't have their own commit identity. You want to see
# your personal `shankar0123` / `skreddy040@gmail.com` here.
git config --get user.name
git config --get user.email
# If either returns nothing, set them globally:
#   git config --global user.name  "shankar0123"
#   git config --global user.email "skreddy040@gmail.com"

# Initialize the git repo
git init
git branch -M main

# Stage and commit everything
git add .
git commit -m "feat: initial certctl.io landing page

Single-file static site, technical/monospace aesthetic. Sections:
hero, problem, pricing, how-it-works, capabilities, dashboard,
community, footer. Tiny inline script fetches the latest GitHub
release tag for the version chip. No build step required."

# Wire to the new GitHub repo (paste the SSH URL from Step 1)
git remote add origin git@github.com:certctl-io/certctl.io.git

# Push
git push -u origin main
```

If `git push` fails with "Permission denied (publickey)", your SSH
key isn't set up for GitHub. Two paths:
- **Easier:** swap to HTTPS — `git remote set-url origin https://github.com/certctl-io/certctl.io.git`. GitHub will prompt for username + a personal access token (NOT your password — go to GitHub Settings → Developer settings → Tokens to create one).
- **Cleaner long-term:** set up an SSH key — <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>.

After the push succeeds, refresh the GitHub repo page — you should see
all your files listed.

### Step 3 — Connect the repo to Cloudflare Pages

1. Go to <https://dash.cloudflare.com> and pick your account.
2. Left sidebar → **Workers & Pages** → **Create**.
3. Pick the **Pages** tab → **Connect to Git**.
4. If this is your first Cloudflare Pages project, it'll ask you to
   authorize Cloudflare to access GitHub. Click through the consent —
   when GitHub asks which orgs/repos to grant access to, pick
   **`certctl-io`** (and your personal account if you want; not
   required for this setup).
5. After authorization, you're back in Cloudflare. You'll see a list
   of repos. Pick **`certctl-io/certctl.io`**. Click **Begin setup**.
6. **Build settings** — fill in EXACTLY these values:
   - **Project name:** `certctl-io` (this becomes the
     `*.pages.dev` subdomain — you can change it later, but pick
     something memorable now)
   - **Production branch:** `main`
   - **Framework preset:** `None`
   - **Build command:** *(leave completely empty)*
   - **Build output directory:** `/` (just a single forward slash)
   - Skip everything else (env vars, root directory, etc.)
7. Click **Save and Deploy**.
8. Cloudflare will deploy your site. The first deploy takes ~30
   seconds. When done, you'll see a `https://certctl-io.pages.dev`
   URL (or similar). Click it to verify the site renders.

### Step 4 — Bind the `certctl.io` domain to the Pages project

1. Inside the Cloudflare Pages project (you should still be in the
   project dashboard from Step 3) → **Custom domains** tab → **Set up
   a custom domain**.
2. Type `certctl.io` and click **Continue**.
3. Cloudflare detects that the domain is registered on Cloudflare
   Registrar and shows a one-click **Activate domain** button — click
   it. Cloudflare automatically creates the DNS record pointing
   `certctl.io` at your Pages project. No manual DNS edits needed.
4. (Optional) Repeat for `www.certctl.io` if you want both apex and
   www to work — Cloudflare can 301 `www` to apex automatically.
5. Universal SSL provisions the TLS certificate within ~60 seconds.
   While it's pending, the domain shows a "Verifying" status; refresh
   in a minute and it should be live.

### Step 5 — Verify the site is live

```bash
# DNS resolves?
dig +short certctl.io
# Should return a Cloudflare IP (104.x.x.x or 172.x.x.x).

# HTTPS works?
curl -sI https://certctl.io | head -3
# Should return: HTTP/2 200, server: cloudflare, content-type: text/html

# Page actually serves?
curl -s https://certctl.io | grep "<title>"
# Should print the <title> line from index.html.
```

If all three pass, open <https://certctl.io> in a browser. Verify:

- Header logo (cycle + shield icon) visible next to "certctl"
- Hero code block shows three separate `$`-prefixed lines
- Version chip shows `v2.0.67` (or whatever your latest tag is) — if
  it shows `v2.0`, the API call is being blocked or rate-limited;
  refresh once
- "We scale with you. Our prices don't." pricing banner is dark
- "Talk to us →" button on the pricing banner shows a mailto link
- Footer logo + 3-line tagline all render
- 4 dashboard screenshots load in the dashboard section

### Step 6 — Set up `hello@certctl.io`

The pricing CTA links to `mailto:hello@certctl.io`. You need to make
that email actually deliver mail to you. Cloudflare Email Routing is
free and takes 2 minutes.

1. <https://dash.cloudflare.com> → **certctl.io** zone (you should
   see it in your zones list since it's on Cloudflare Registrar).
2. Left sidebar → **Email** → **Email Routing**.
3. If Email Routing isn't enabled yet, click **Get Started**. It
   adds the required MX + TXT records automatically.
4. Once enabled, **Routes** tab → **Create address**:
   - **Custom address:** `hello`
   - **Action:** Send to an email
   - **Destination:** your real email (e.g.
     `skreddy040@gmail.com`)
5. Save. Cloudflare sends a verification email to your destination
   address — click the verification link.
6. Test it: from any other email account, send a message to
   `hello@certctl.io`. Should land in your real inbox within seconds.

You can add more aliases the same way — `support@`, `enterprise@`,
`security@` — all forwarding to whatever destination you want.

---

## Subdomains (future phases)

| Subdomain         | What it serves                                  | When                   |
| ----------------- | ----------------------------------------------- | ---------------------- |
| `docs.certctl.io` | MkDocs Material build of `certctl/docs/`        | Strategy doc Phase 3   |
| `demo.certctl.io` | Read-only certctl instance behind a Caddy proxy | Strategy doc Phase 2   |

Each gets its own Cloudflare Pages project (or in the demo case, an
A record to a VPS IP). Same Custom Domains flow as Step 4 above.

---

## What to do later

These aren't required for launch but are the next logical steps:

### Transfer the main `certctl` repo to the org

Right now the main code repo lives at `shankar0123/certctl`; the
website now lives at `certctl-io/certctl.io`. The split works
short-term but the right end-state is `certctl-io/certctl` so the
two sit under one identity.

How:
1. <https://github.com/shankar0123/certctl> → **Settings** → scroll
   to **Danger Zone** → **Transfer ownership**.
2. New owner: `certctl-io`. New repo name: `certctl` (default).
3. Confirm. Stars, forks, issues, PRs, watchers, and external
   redirects all transfer cleanly. Existing
   `github.com/shankar0123/certctl/...` URLs auto-redirect forever.
4. After the transfer, update the ~10 `shankar0123/certctl` URLs in
   `index.html` to `certctl-io/certctl`. The old links keep working
   via redirect, so this is a one-shot follow-up commit, not a hard
   deadline.

### Update the GitHub-stars badge

The header currently uses a shields.io image embed for the star
count. It works but flickers on first paint (image loads after
layout). Once the site is live, you can swap to a fetch-and-render
approach (same pattern as the version chip) — eliminates the flicker
and removes the third-party-CDN dependency. Tell me when ready.

### Open Graph / Twitter card image

The page currently uses `screenshot-dashboard.png` as the og:image.
For a sharper Twitter/Discord/Slack preview, the convention is a
1200×630 PNG showing the wordmark + tagline + dashboard glimpse.
Side project; ship it whenever.

---

## Editing the page

Everything is in `index.html`. CSS lives in the `<style>` block at
the top of the file; HTML follows. To change colors site-wide, edit
the CSS variables in `:root`:

```css
--accent: #0b5fff;        /* primary CTA + link color */
--text: #0f1115;          /* primary text */
--text-muted: #5c6470;    /* secondary text */
--border: #e4e7eb;        /* card/section borders */
--bg-soft: #fafaf7;       /* warm cream for callouts */
--bg-code: #f4f4f0;       /* code-block background */
```

Sections are clearly delimited by HTML comments
(`<!-- 01 — Problem -->` etc.) matching the eyebrow numbering on
each section. Search for `class="eyebrow"` to find the pattern.

After any edit:

```bash
cd /Users/shankar/Desktop/cowork/certctl.io
git add -A
git commit -m "site: <what changed>"
git push
```

Cloudflare Pages auto-deploys on every push to `main`. Build takes
~10 seconds. The site is updated globally within ~60 seconds.

---

## Regenerating logo variants

The three derived logo files (`logo-icon.png`, `logo-footer.png`,
`favicon-32.png`) are generated from `certctl-logo.png` via Pillow.
If you replace the master logo, regenerate the derivatives:

```bash
cd /Users/shankar/Desktop/cowork/certctl.io
python3 << 'PY'
from PIL import Image
src = Image.open('certctl-logo.png')
w, h = src.size

# Icon: top 62% of the source, center-square cropped
icon = src.crop((0, 0, w, int(h * 0.62)))
sq = min(icon.size); left = (icon.size[0] - sq) // 2
icon_sq = icon.crop((left, 0, left + sq, sq))

icon_h = icon_sq.copy(); icon_h.thumbnail((80, 80), Image.LANCZOS)
icon_h.save('logo-icon.png', optimize=True, compress_level=9)

icon_f = icon_sq.copy(); icon_f.thumbnail((64, 64), Image.LANCZOS)
icon_f.save('favicon-32.png', optimize=True, compress_level=9)

footer = src.copy(); footer.thumbnail((240, 240), Image.LANCZOS)
footer.save('logo-footer.png', optimize=True, compress_level=9)
PY
```

---

## Troubleshooting

| Symptom                                         | What's wrong                       | Fix                                                              |
| ----------------------------------------------- | ---------------------------------- | ---------------------------------------------------------------- |
| `git push` returns "Permission denied (publickey)" | SSH key not set up for GitHub    | Switch the remote to HTTPS (see Step 2) or set up an SSH key     |
| Cloudflare Pages "Begin setup" doesn't show the repo | GitHub authorization didn't include the org | Go back to the GitHub auth screen, add `certctl-io` to authorized orgs |
| Page deploys but shows 404 | Build output directory is wrong | In Pages → Settings → Builds & deployments → set output dir to `/` |
| Custom domain stuck "Verifying" >5 min | DNS propagation delay or Universal SSL provisioning slow | Wait 10 more minutes; if still stuck, click "Retry" in the Custom Domains panel |
| Version chip stays "v2.0" instead of latest tag | GitHub API rate-limited (60/hr unauth) | Refresh once; localStorage caches for 6h after a successful fetch |
| Screenshots don't load | File names / paths broken after a `git mv` | Check `index.html` for `src="screenshot-..."`; verify files exist |
| Email to `hello@certctl.io` bounces | Email Routing not enabled or alias not created | dash.cloudflare.com → certctl.io zone → Email → Email Routing → check enabled + verify the destination address |
| Commits show generic gravatar instead of your `shankar0123` avatar | `git config user.email` doesn't match an email verified on your GitHub account | Set `git config --global user.email "<email-verified-on-your-GitHub-account>"`, then amend the last commit with `git commit --amend --reset-author --no-edit && git push --force-with-lease` |
| Styles look broken in Safari | Browser cache | Hard refresh (Cmd+Shift+R) or open in private window |

For anything not on this table, the Cloudflare Pages project's
**Deployments** tab shows the build log for every deploy. If
something broke in the most recent deploy, the log is the first
place to look.

---

## What's NOT on the page (and won't be without explicit decision)

Per the strategy doc:

- No pricing tiers (V3 isn't public)
- No competitor comparison table
- No "AI-powered" language
- No V3 feature details
- No signup form (nothing to sign up for)

The pricing CTA points to `hello@certctl.io` — that's the lightest-
touch enterprise funnel that doesn't commit to features or pricing.
The MCP server gets one mention in the capabilities grid as power-
user tooling, never as a headline.

---

## When v1 outgrows a single file

If/when the site needs blog posts, multi-page navigation, or shared
components, the migration path is **Astro + Tailwind** (the strategy
doc's "alternative" option). The content here transfers in roughly
an hour: sections become `.astro` components, the CSS tokens become
a Tailwind config, the screenshots stay as-is.

Until then, single file. Easier to read, easier to ship, easier to
debug.
