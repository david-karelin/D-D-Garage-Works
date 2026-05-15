# D&D Garage Works — Site Brief

A description of the current marketing site and a rebuild prompt for Claude Code.

---

## Premise

**D&D Garage Works** is a two-person residential garage cleanout and organizing service in Thornhill / Richmond Hill, Ontario. Run by two childhood friends, both named David. Hyperlocal, hand-done, no franchise, no subcontractors.

Value propositions the site must communicate:

- **Free, in-person on-site quote** a day or two before the job. No virtual estimates from photos.
- **Same two people every time.** No rotating crews.
- **They organize, not just empty.** Clear zones and labels, not a swept concrete box.
- **You stay in the room.** Homeowner makes the keep/toss calls.
- **One honest written number.** No curb upcharges.

**Tone:** warm, neighborly, slightly hand-crafted. Small craft business, not a service franchise. Not "junk removal."

---

## Information Architecture

Six sections. Currently a single HTML file with hash routing; the rebuild moves to real routes.

### Home (`/`)
- Hero: headline, 3-photo stack, trust line
- "How it starts" — 3-step on-site quote callout
- Services preview — 3 cards with price
- "Why people hire us" — 4 reasons
- "How it works" teaser — 3 of the 7 phases
- Featured before/after pair
- 3 testimonials + aggregate rating
- Final CTA band (teal)

### Services (`/services`)
Three pricing tiers. Each card: tag, price, time estimate, "Right for you if" fit-box, description, included list, "Not included" list.

| Tier | Name | Price | Time | Truck? |
|---|---|---|---|---|
| 1 | Declutter & Organize | $325 | ~4–5 hrs | No |
| 2 | Garage Cleanout (popular) | $400–$575 | Full reset | Yes, haul-away |
| 3 | Estate & Full-Day Cleanout | $900–$1,500+ | Full day+ | Yes |

Plus:
- "Extras & materials" note — disposal fees, shelving, bins at cost
- "What we don't do" — hazmat, paint, propane, etc.

### Process (`/process`) — the signature offering
Sticky left phase-nav (1–7). Each phase: big numeral, name, time estimate, body copy, callout box (rules or checklist).

1. Walkthrough
2. Setup
3. The Sort
4. Haul
5. Organize — 4 zones: Active / Seasonal / Long-term / Workbench
6. Clean
7. Final walkthrough

### About (`/about`)
- Hero with the two Davids' story (childhood friends)
- Story section, pull-quote, portrait placeholders
- Values grid on dark teal band (3 values)
- Duo portrait (David K. + David R.)
- Trust strip: licensed, insured, local

### FAQ (`/faq`)
Accordion list with animated +/− toggle.

### Quote (`/quote`)
- 3-step mini-recap of the quote flow
- Form: name, phone, email, address, garage size (radio), project type (radio), preferred times (chip selector), notes (textarea), optional photo upload (drag-drop)
- Success state
- Alt-contact (phone / text / email) below

### Chrome
- Sticky header: logo + 4 nav links + CTA button, scrolls compact
- Mobile hamburger → full-screen overlay
- Dark teal footer, 3-column grid

---

## Design System

### Palette (derived from original logo)
**Teal scale**
- `#183B3F` primary
- `#3A5761`
- `#75868E`
- `#DDE7E8` soft

**Rust / terracotta accents**
- `#682C10`
- `#8E3B13`
- `#C58059`
- `#F3DECC` soft

**Warm surfaces**
- `#F6F2EA` paper
- `#ECE6D8` deeper band
- `#EEF1F0` cool tint

**Ink:** `#14282B`

### Typography
- **Fraunces** (variable serif) — headlines, prices, pull-quotes
- **Inter** — body, UI, nav
- **Caveat** (handwritten) — step numerals, big numbers, "FREE" mark, hero badge — the signature handmade accent

### Motifs
- Big Caveat numerals as step counters
- Short rust underline as section/heading delimiter (`::before` 36×2px bar)
- Photo placeholders: 135° striped fill + dashed inner border + monospace caption — these are deliberate markers for real photography
- Rotated teal "D & D" badge on hero with terracotta drop shadow

### Components
- Buttons: primary (rust), ghost (teal outline), on-teal (terracotta)
- Service tier card, pricing tier row
- Before/after pair with corner tags
- Testimonial card with terracotta top-rule
- FAQ accordion with custom border-circle +/− toggle
- Form: text inputs, radio chip group, time-slot chip group, file drop-zone, success state
- Callout boxes — rust left-border, paper-cool bg, for rules and checklists
- Sticky phase-nav for the Process page

---

## What "a better version" means

Ranked by impact.

**Must**
1. **Real routes, real `<head>`.** Hash routing breaks crawlability and per-page OG/social previews. Move to Astro pages with proper `<title>`, meta description, OG tags, and `LocalBusiness` + `Service` JSON-LD.
2. **Real form submission.** Currently no backend. Wire to a single endpoint with validation, honeypot, and email delivery.
3. **Real photography pipeline.** Every image is a striped placeholder. Use `<Picture>` with AVIF/WebP and responsive `srcset`. Emit a shot list for the homeowners to fill.
4. **Content out of markup.** Tiers, phases, FAQs, testimonials become typed content collections, not hardcoded HTML.

**Should**
5. **Accessibility.** Focus management, `aria-current` on nav, keyboard support for chip selectors and file drop, proper labels.
6. **Performance.** Self-host fonts subset to Latin. Defer non-critical CSS. Inline above-the-fold styles.
7. **Mobile polish.** Hero photo stack collapses awkwardly; tier cards stack without rhythm.
8. **Trust signals.** Google reviews embed, insurance/license proof, service-area map.

**Nice-to-have**
9. Real before/after slider component
10. Analytics + lead-source tracking (Plausible, UTM passthrough)
11. Recent jobs feed

**Deliberately out of scope for v1**
- File uploads on the quote form (defer — adds backend complexity, photo quote isn't part of their offer anyway)
- Embedded calendar (Cal.com) — they do in-person estimates a day or two ahead, so a scheduler overcommits them; "preferred times" chips match the brand better

---

## Rebuild Prompt for Claude Code

> Rebuild the D&D Garage Works site as an Astro project with six routed pages: Home (`/`), Services (`/services`), Process (`/process`), About (`/about`), FAQ (`/faq`), Quote (`/quote`). Preserve the brand exactly as documented in `BRIEF.md`: teal `#183B3F`, rust `#8E3B13`, cream `#F6F2EA`; Fraunces for headlines and prices, Inter for body, Caveat for step numerals and the hero "FREE" mark; rust 36×2px `::before` underline as section delimiter.
>
> **Content.** Move every piece of copy — 3 service tiers, 7 process phases, FAQs, testimonials — into typed content collections under `src/content/`. Do **not** invent copy I haven't given you; where copy is missing, leave a `TODO:` marker with a short note about what belongs there.
>
> **Photos.** Replace the striped placeholders with an `<Image>` or `<Picture>` component reading from a `photos` collection. Emit `PHOTOS-TODO.md` listing every slot with suggested shot, target dimensions, and alt text. Keep the existing striped-placeholder visual as the fallback when a photo file is missing — the placeholders are part of the design language, not a bug.
>
> **Quote form.** Single Astro POST endpoint at `/api/quote`. Validate with Zod. Honeypot field. Rate-limit by IP. Send the lead via Resend to a configurable `LEAD_EMAIL`. Reply with the existing success state. No file uploads in v1. No captcha in v1 — honeypot + rate limit is enough for a two-person business.
>
> **SEO.** Per-page `<title>` and meta description. OG image per page. `LocalBusiness` JSON-LD on Home (Thornhill, ON service area). `Service` JSON-LD on Services. `FAQPage` JSON-LD on FAQ. `sitemap.xml` and `robots.txt`.
>
> **Fonts.** Self-host Fraunces (variable), Inter, and Caveat. Subset to Latin. `font-display: swap`. Preload only Inter 400 and Fraunces variable.
>
> **Quality bar.** Lighthouse 90+ on mobile across Performance, Accessibility, Best Practices, SEO. Keyboard navigable. `aria-current="page"` on the active nav link. Chip groups are real radio groups with labels.
>
> **Tone.** Warm, neighborly, slightly hand-crafted. Two friends, not a franchise. Don't corporatize the copy.
>
> **Don't.**
> - Don't invent testimonials, prices, or phase descriptions.
> - Don't add file uploads, captchas, or a booking widget.
> - Don't replace the handwritten Caveat motif with a generic sans — it is the brand.
> - Don't add a blog, a careers page, or social links I haven't asked for.
