# Deploying Flourish or Flounder to your VPS with Coolify

The site is a handful of static files. There's no build step, no framework, no
database. That makes this about as simple as web hosting gets.

**Keep all of these in the same folder — they must deploy together:**

| File | What it is | Size |
|------|-----------|------|
| `index.html` | The entire site — all content, styling and behaviour | ~50KB |
| `logo.png` | Your full logo lockup, used as the hero headline | ~250KB |
| `mark.png` | The F mark, used small in the navigation bar | ~25KB |
| `favicon.png` / `favicon.ico` | Browser tab icons | ~26KB |
| `apple-touch-icon.png` | Icon when someone saves the site to an iPhone home screen | ~11KB |
| `leaf-hang.png` / `leaf-rise.png` | Foliage bands cut from your logo artwork, framing several sections | ~280KB |

Later you'll add one more: `cate.jpg`, your portrait.

All the images were generated from the logo and favicon you supplied — trimmed
of their white margins, cut out with transparent backgrounds where needed, and
compressed for the web. Your originals are untouched.

There are two routes below. **Route A (Git) is the one to use** — it takes ten
minutes to set up once and then every future change is a one-line push that
deploys itself. Route B is a fallback if you'd rather not touch Git at all.

---

## Before you start — three things to edit

Your LinkedIn URL and the three testimonials are already in. Open `index.html`
in any plain-text editor (VS Code, Notepad++, TextEdit in plain-text mode —
**not** Word, which will mangle it). Search for `EDIT ME` and you'll find three
marked spots:

| # | What | Where |
|---|------|-------|
| 1 | Your real domain, in the `canonical`, `og:url` and `og:image` tags | Near the top, in `<head>` |
| 2 | Your email address — appears in **4** `mailto:` links | Nav button, CTA button, footer |
| 3 | Your portrait photo | The "A bit about me" section |

The fastest way to do 1 and 2 is find-and-replace across the whole file:

- Replace `hello@flourishorflounder.co.uk` → your real address
- Replace `flourishorflounder.co.uk` → your real domain

### Adding your photo

1. Save the photo as `cate.jpg` in the same folder as `index.html`.
   Portrait orientation, roughly 800 × 1000 pixels, under ~300KB.
2. In `index.html`, find `EDIT ME (3/3)`.
3. Delete the whole `<div class="portrait-placeholder"> ... </div>` block.
4. Uncomment the line above it by removing `<!--` and `-->`:
   ```html
   <img src="cate.jpg" alt="Cate Hulme, Fractional CFO">
   ```

Until you do that, the site shows a neat marked placeholder — it won't look broken.

**To check your edits before deploying:** double-click `index.html` and it opens
in your browser exactly as it will appear live.

---

## Route A — Git repository (recommended)

### Step 1: Put the files in a Git repository

Create a new **private** repository on GitHub (or GitLab) called something like
`flourishorflounder`. Then, in a terminal, from the folder containing the
site files:

```bash
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin https://github.com/roger296/flourishorflounder.git
git push -u origin main
```

If you'd rather avoid the command line entirely, GitHub's web interface has an
"uploading an existing file" link on any empty repository — drag the whole set
of files in and commit. That works just as well. Just make sure every file
listed in the table above goes up, not only `index.html`.

### Step 2: Connect the repository to Coolify

1. Log into your Coolify dashboard.
2. Go to your **Project** → **Environment** (usually `production`) →
   **+ New Resource**.
3. Choose **Public Repository** if you made the repo public, or **Private
   Repository (with GitHub App)** if private. The GitHub App route asks you to
   authorise Coolify against your GitHub account once — worth doing, because it
   also enables automatic redeploys.
4. Paste the repository URL and pick the `main` branch.

### Step 3: Configure it as a static site

This is the important screen. Set:

| Field | Value |
|-------|-------|
| **Build Pack** | `Static` |
| **Base Directory** | `/` |
| **Publish Directory** | `/` |
| **Ports Exposes** | `80` |
| **Install Command** | *leave empty* |
| **Build Command** | *leave empty* |

Coolify's `Static` build pack wraps your folder in a small nginx container and
serves it. Because there's nothing to compile, the install and build commands
stay blank.

> If you don't see a `Static` option in your Coolify version, choose **Nixpacks**
> and set the Publish Directory to `/` — or use the Dockerfile in Route B, which
> works on every version.

### Step 4: Point your domain at the server

**In your domain registrar's DNS settings**, create two records:

| Type | Name | Value |
|------|------|-------|
| `A` | `@` | your VPS's IPv4 address |
| `A` | `www` | your VPS's IPv4 address |

(If your registrar wants a `CNAME` for `www`, pointing it at your root domain
is fine too.)

DNS changes usually propagate in a few minutes but can take up to a few hours.
You can check progress at `dnschecker.org`.

**In Coolify**, on your resource's **Configuration** page, set the **Domains**
field to:

```
https://flourishorflounder.co.uk
```

Use `https://` and no trailing slash. Coolify reads the protocol from this field
to decide whether to issue a certificate — this is the single most common thing
people get wrong.

### Step 5: Deploy

Hit **Deploy**. Watch the log panel; it should finish in under a minute with a
green success.

Coolify's built-in proxy (Traefik or Caddy, depending on your setup) will
request a Let's Encrypt certificate automatically once DNS resolves to your
server. If the padlock doesn't appear within a few minutes, see Troubleshooting
below.

### Step 6: Redirect www → root (optional but tidy)

Add `https://www.flourishorflounder.co.uk` to the **Domains** field as a second
entry (comma-separated), and Coolify will serve both. To force one canonical
address, enable **Redirect to non-www** (or **to www**, if you prefer that) in
the resource's advanced settings.

### Making changes later

Edit `index.html`, then:

```bash
git add .
git commit -m "Added testimonials"
git push
```

If you connected via the GitHub App, Coolify redeploys automatically within
seconds. Otherwise, click **Deploy** in the dashboard.

---

## Route B — Dockerfile (no Git service needed)

If you'd rather not use GitHub, you can deploy from a Dockerfile. Create a file
called `Dockerfile` (no extension) next to `index.html`:

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
```

Then in Coolify choose **+ New Resource → Dockerfile**, paste those three lines,
and set **Ports Exposes** to `80`. Domain and SSL setup are identical to Steps 4
and 5 above.

To update the site with this route you edit the Dockerfile's contents in Coolify
or re-upload — which is more fiddly than Route A, which is why Route A is
recommended.

---

## Troubleshooting

**"Bad Gateway" or a blank page after a successful deploy**
Almost always the **Ports Exposes** setting. It must be `80` — the port *inside*
the container, not the port you visit. Set it, redeploy.

**No padlock / certificate error**
Three things to check, in order:
1. Does `flourishorflounder.co.uk` actually resolve to your VPS IP yet? Run
   `dig +short flourishorflounder.co.uk` or use dnschecker.org. Let's Encrypt
   cannot issue a certificate until it does.
2. Is the Domains field written as `https://...` with no trailing slash?
3. Are ports 80 **and** 443 open on the VPS firewall? Let's Encrypt validates
   over port 80 even though the site serves on 443. On Ubuntu:
   `sudo ufw allow 80 && sudo ufw allow 443`.

**Moving a leaf band exposes a straight cut line**
Each foliage strip was cut out of the logo artwork, so one horizontal edge is a
straight cut. The CSS deliberately parks that edge outside the section using the
`--leaf-bleed` value. `leaf-hang.png` must hang from a section's **top** edge and
`leaf-rise.png` must rise from its **bottom** edge — swap them, or reduce the
bleed too far, and the cut shows.

**The logo has a visible white box around it**
The logo files have a white background baked in, and the site uses CSS
`mix-blend-mode: multiply` to drop it out over the tinted sections. Every
current browser supports this. If you ever want a truly transparent logo, ask
your designer for a PNG with an alpha channel and drop it in over `logo.png` —
no code change needed.

**Fonts don't load / the site looks like plain Times New Roman**
The site pulls Fraunces and Inter from Google Fonts. If your VPS or a visitor's
network blocks Google Fonts, the browser falls back to system serif and sans
fonts — readable, but not the intended look. If you'd prefer zero external
dependencies, tell me and I'll produce a version with the fonts embedded
directly in the file.

**Changes don't appear after redeploying**
Hard-refresh your browser: `Ctrl + Shift + R` (Windows) or `Cmd + Shift + R`
(Mac). Browsers cache HTML aggressively.

---

## A few things worth knowing

- **No dependencies.** Nothing to patch, nothing to keep updated, no plugins
  that break. It will still work identically in five years.
- **It's fast.** Around 620KB in total including the logo and leaf artwork — it will score well
  on PageSpeed and load quickly on mobile.
- **Accessibility.** Semantic headings, a skip link, visible keyboard focus
  states, AA-contrast text, and `prefers-reduced-motion` support for anyone who
  finds animation uncomfortable.
- **No cookies, no analytics, no tracking.** That means no cookie banner and no
  GDPR consent obligations. If you later want visitor numbers, Plausible or
  Fathom are privacy-friendly options that stay cookie-free — a single line to
  add.
- **Backup.** If you use Route A, your Git repository *is* your backup. Keep a
  copy of the folder somewhere else too.
