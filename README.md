# D&D Garage Works

Marketing site for D&D Garage Works — a two-person garage cleanout service
in Thornhill, ON. Single-file static site, no build step.

## Files

| Path | What it is |
|---|---|
| `index.html` | The whole site — six hash-routed sections in one file |
| `assets/` | Logo variants, color reference, downloadable PDF checklist |
| `robots.txt` / `sitemap.xml` | Search engine basics |
| `site.webmanifest` | PWA / mobile install metadata |
| `404.html` | Friendly "lost page" page |
| `BRIEF.md` | Full site brief — premise, IA, design system, and a rebuild prompt |

## How to deploy (any of these works)

You don't need any developer tools. Pick one:

**Netlify Drop (easiest, free):** open https://app.netlify.com/drop and drag
the entire folder onto the page. You get a live URL in 30 seconds. Buy a
domain later and point it at the site from Netlify's dashboard.

**GitHub Pages:** in this repo's GitHub settings → Pages → set the source
to `main` branch, root folder. Done in a minute.

**Vercel:** import the repo at https://vercel.com/new — auto-detected as a
static site, no config needed.

## Before going live — two 5-minute tasks

### 1. Wire up the quote form

Right now the form falls back to an in-page "success" screen but doesn't
actually email anyone. To make it send leads to your inbox:

1. Sign up at https://formspree.io/ (free tier handles 50 submissions/month).
2. Create a new form. Formspree gives you an ID like `abcd1234`.
3. Open `index.html` and find this line (search for `YOUR_FORMSPREE_ID`):

   ```html
   action="https://formspree.io/f/YOUR_FORMSPREE_ID"
   ```

   Replace `YOUR_FORMSPREE_ID` with your real ID. Save. Re-deploy.

The form already has a honeypot field for spam, sends a useful subject line,
and shows the success state without leaving the page.

### 2. Set your real domain and phone

Search and replace in `index.html`:

- `ddgarageworks.ca` → your actual domain (used in OG tags, JSON-LD, and the
  canonical link)
- `(905) 555-0100` → your real phone (appears in the footer, JSON-LD, and
  alt-contact area)
- `hi@ddgarageworks.ca` → your real email

Also update the same strings in `sitemap.xml` and `robots.txt` once your
domain is decided.

## Photos

Every photo slot in the site is currently a striped placeholder with a
caption describing the shot needed. Once you have real photos:

1. Save them as `.jpg` or `.webp` in `assets/photos/`.
2. In `index.html`, find each `<div class="photo …">` block and replace
   the inner `<span class="photo-label">` with `<img src="assets/photos/your-photo.jpg" alt="Description">`.

Keep the striped placeholder design — it's part of the brand language for
slots that don't have photos yet.
