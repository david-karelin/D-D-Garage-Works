# D&D Garage Works — "Garage Reset" Hero Animation — Build Prompt

Add a single signature animated hero to the existing `index.html` (the static,
hash-routed, single-file site). All CSS inline, all JS inside the existing IIFE
— no frameworks, no build step, no new external assets. The hero replaces/upgrades
the current static hero photo stack on the Home (`#/`) route only.

The animation's job is conversion: it dramatizes the core promise — **we organize,
we don't just empty** — across four stages of a real job, ending every cycle on a
CTA into the free on-site quote flow.

---

## Known Gotchas

1. **Don't break hash routing** — The hero lives on the `#/` route only. The
   slider's auto-advance timer and key listeners must pause/teardown when the
   route changes (on `hashchange` away from `#/`) and re-init when it returns.
   A timer left running on `#/quote` will fight the booking embed.

2. **Reflow trick is mandatory** — The incoming slide must be snapped to its
   off-screen start position with transitions disabled, then forced to reflow
   (`void el.offsetHeight`), then transitions re-enabled on the next
   `requestAnimationFrame` before swapping to active. Skip either step and the
   slide just teleports with no motion. Full sequence in the JS section — do
   not reorder.

3. **CSS can't interpolate `radial-gradient()`** — The per-stage background wash
   uses the dual-fixed-div crossfade trick (`#hero-bg-a` / `#hero-bg-b`). Do not
   try to transition `background` directly.

4. **`prefers-reduced-motion` is non-negotiable for this brand** — If `reduce` is
   set: kill the auto-advance timer, drop all transition durations to `0s`, and
   let the arrows do instant swaps. The hero must still be fully usable and
   readable. Build this in from the start.

5. **Photo fallback stays** — Each stage photo is `<img onerror>` → the existing
   striped-placeholder div. The placeholder is part of the brand language, not a
   bug. The animation must look correct even when a photo file is missing (it
   animates the placeholder card instead).

---

## Assets (already in the repo)

Stage photos live in `assets/photos/`. The four stages map onto the real photos
already shot:

| Stage       | Photo file            |
|-------------|-----------------------|
| Before      | `before-interior.jpg` |
| The Sort    | `sort-driveway.jpg`   |
| Organized   | `after-wide.jpg`      |
| The Details | `after-tools.jpg`     |

Floating "items" are inline SVG garage-tool icons (bin, wrench, broom,
label-tag, hook) — reuse the scattered-tool SVGs already in the codebase. 3 per
stage. No new image files.

## Fonts (already loaded)

- **Fraunces** — the giant stage headline
- **Inter** — the promise line + counter + CTA
- **Caveat** — the persistent "FREE on-site quote" badge and the big "01 / 04"
  counter numerals (the handmade accent — keep it)

## Colours (documented palette)

- Hero text: cream `#F6F2EA` / white, with `rgba(255,255,255,X)` for muted
- Accent / CTA / badge: rust `#8E3B13`
- Background washes crossfade per stage (deep teal and rust tones so cream text
  stays legible):

```js
const heroBg = [
  'radial-gradient(ellipse at 50% 45%, #3A5761 0%, #14282B 100%)', // Before  — deep teal
  'radial-gradient(ellipse at 50% 45%, #8E3B13 0%, #2A1206 100%)', // Sort    — rust
  'radial-gradient(ellipse at 50% 45%, #3A5761 0%, #102427 100%)', // After   — clean teal
  'radial-gradient(ellipse at 50% 45%, #C58059 0%, #5A2912 100%)', // Details — warm terracotta
];
```

---

## Hero Layout (per slide)

`.hero { height: 100vh; overflow: hidden; }` — the page still scrolls vertically
below it.

**Persistent (outside the slides):** the existing sticky header sits on top
(z-index above hero). Bottom-left of the hero: the Caveat "FREE on-site quote"
badge + a rust "Book Your Free Quote" button → `#/quote`. Bottom-right: prev/next
arrows + Caveat counter "01 / 04". Bottom-centre: scroll hint "See the 7-phase
method ↓" → `#/process`, chevron bounces (CSS).

**Per slide (`.slide`):**

- Giant Fraunces headline centred, `clamp(40px, 7vw, 96px)`, cream — class `.headline-anim`
- Scene card (the framed photo, rounded 22px, drop-shadow), centred,
  `clamp(280px, 42vw, 620px)` wide — class `.scene-anim`
- Promise line (Inter, `rgba(246,242,234,0.85)`, max-width 460px) bottom-left
  above the badge — class `.promise-anim`
- 3 floating tool icons that fall onto the scene — classes `.tool-1` / `.tool-2`
  / `.tool-3`, each `.tool.anim`
  - `tool-1`: top-left, `clamp(54px, 6vw, 88px)`, `top: 14%; left: 9%`, base tilt `-16deg`
  - `tool-2`: top-right, `clamp(46px, 5vw, 76px)`, `top: 10%; right: 10%`, base tilt `12deg`
  - `tool-3`: lower-right, `clamp(40px, 4.5vw, 64px)`, `bottom: 24%; right: 20%`, base tilt `-9deg`

### Slide copy

| #  | Stage       | Headline (Fraunces)            | Promise (Inter)                                                              |
|----|-------------|--------------------------------|-----------------------------------------------------------------------------|
| 01 | Before      | EVERY RESET STARTS HERE        | Packed wall-to-wall? That's our favourite kind of garage.                   |
| 02 | The Sort    | YOU MAKE THE CALLS             | Everything comes out onto the driveway — you stay in the room for every keep-or-toss. |
| 03 | Organized   | WE ORGANIZE, NOT JUST EMPTY    | Clear zones, labelled bins, and a floor you can park on again.              |
| 04 | The Details | THE SAME TWO DAVIDS            | Start to finish, same two people. One honest written number.               |

---

## Animation System — Implement Exactly

No CSS keyframes anywhere. All motion is driven by class swaps on the slide; CSS
transitions interpolate between states. Read this whole section before writing code.

### Five slide states

| Class         | Meaning                          |
|---------------|----------------------------------|
| `active`      | On screen, settled               |
| `enter-right` | Off-screen right, about to slide in |
| `enter-left`  | Off-screen left, about to slide in  |
| `exit-left`   | Flying off to the left           |
| `exit-right`  | Flying off to the right          |

### Three element types, three behaviours

1. **Text (headline + promise)** — slides horizontally with spring easing;
   overshoots and snaps back. This is the "bounce."
2. **Scene card (the photo)** — slides horizontally and rotates in the same
   transform (translate + rotate together). Entering from the right starts at
   `+30°` and travels in from the right; settles to `0°`. The combined transform
   is what makes it feel like the garage "swings" into place.
3. **Tool icons** — no horizontal motion. They only fall vertically. On exit
   they fly straight up (`translateY(-110vh)`); on enter they start above the
   screen (`translateY(-90vh)`) and drop into place with gravity easing (no
   bounce — they're being set down, not thrown). Each tool always preserves its
   base tilt; only `translateY` changes between states. This literally reads as
   items being placed onto the shelves — the brand's whole pitch.

### Stagger

Tools land after the scene has settled. Scene settles ~0.85s; tools cascade at
0.30s / 0.38s / 0.46s via `transition-delay`.

### Exit vs enter timing

Exit is faster (0.5s, ease-in) than enter (0.85s, spring) — the old garage gets
snapped away, the new stage glides in. Never make them equal.

### Easing values

```
Spring  → cubic-bezier(0.34, 1.56, 0.64, 1)   scene + text entering — overshoot then settle
Gravity → cubic-bezier(0.16, 1, 0.3, 1)        tools dropping in — smooth deceleration, no bounce
Snap    → cubic-bezier(0.4, 0, 1, 1)            everything exiting — ease-in
```

### Full Animation CSS

```css
.anim {
  opacity: 0;
  transition: transform 0.85s cubic-bezier(0.34, 1.56, 0.64, 1), opacity 0.55s ease;
}
.tool.anim {
  transition: transform 0.9s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.55s ease;
}
.tool-1 { transform: rotate(-16deg); }
.tool-2 { transform: rotate(12deg); }
.tool-3 { transform: rotate(-9deg); }
/* ACTIVE */
.slide.active .headline-anim { opacity: 1; transform: translateX(0); }
.slide.active .scene-anim    { opacity: 1; transform: translateX(0) rotate(0deg); }
.slide.active .promise-anim  { opacity: 1; transform: translateX(0); }
.slide.active .tool-1 { opacity: 1; transform: rotate(-16deg) translateY(0); }
.slide.active .tool-2 { opacity: 1; transform: rotate(12deg)  translateY(0); }
.slide.active .tool-3 { opacity: 1; transform: rotate(-9deg)  translateY(0); }
.slide.active .headline-anim { transition-delay: 0s; }
.slide.active .promise-anim  { transition-delay: 0.04s; }
.slide.active .scene-anim    { transition-delay: 0.07s; }
.slide.active .tool-1        { transition-delay: 0.30s; }
.slide.active .tool-2        { transition-delay: 0.38s; }
.slide.active .tool-3        { transition-delay: 0.46s; }
/* ENTER RIGHT */
.slide.enter-right .headline-anim { opacity: 0; transform: translateX(65vw); }
.slide.enter-right .scene-anim    { opacity: 0; transform: translateX(72vw) rotate(30deg); }
.slide.enter-right .promise-anim  { opacity: 0; transform: translateX(55vw); }
.slide.enter-right .tool-1 { opacity: 0; transform: rotate(-16deg) translateY(-90vh); }
.slide.enter-right .tool-2 { opacity: 0; transform: rotate(12deg)  translateY(-90vh); }
.slide.enter-right .tool-3 { opacity: 0; transform: rotate(-9deg)  translateY(-90vh); }
/* ENTER LEFT */
.slide.enter-left .headline-anim { opacity: 0; transform: translateX(-65vw); }
.slide.enter-left .scene-anim    { opacity: 0; transform: translateX(-72vw) rotate(-30deg); }
.slide.enter-left .promise-anim  { opacity: 0; transform: translateX(-55vw); }
.slide.enter-left .tool-1 { opacity: 0; transform: rotate(-16deg) translateY(-90vh); }
.slide.enter-left .tool-2 { opacity: 0; transform: rotate(12deg)  translateY(-90vh); }
.slide.enter-left .tool-3 { opacity: 0; transform: rotate(-9deg)  translateY(-90vh); }
/* EXIT LEFT */
.slide.exit-left .headline-anim,
.slide.exit-left .scene-anim,
.slide.exit-left .promise-anim {
  transition: transform 0.5s cubic-bezier(0.4, 0, 1, 1), opacity 0.4s ease-in;
}
.slide.exit-left .tool.anim {
  transition: transform 0.42s cubic-bezier(0.4, 0, 1, 1), opacity 0.32s ease-in;
}
.slide.exit-left .headline-anim { opacity: 0; transform: translateX(-55vw); }
.slide.exit-left .scene-anim    { opacity: 0; transform: translateX(-60vw) rotate(-26deg); }
.slide.exit-left .promise-anim  { opacity: 0; transform: translateX(-45vw); }
.slide.exit-left .tool-1 { opacity: 0; transform: rotate(-16deg) translateY(-110vh); }
.slide.exit-left .tool-2 { opacity: 0; transform: rotate(12deg)  translateY(-110vh); }
.slide.exit-left .tool-3 { opacity: 0; transform: rotate(-9deg)  translateY(-110vh); }
/* EXIT RIGHT */
.slide.exit-right .headline-anim,
.slide.exit-right .scene-anim,
.slide.exit-right .promise-anim {
  transition: transform 0.5s cubic-bezier(0.4, 0, 1, 1), opacity 0.4s ease-in;
}
.slide.exit-right .tool.anim {
  transition: transform 0.42s cubic-bezier(0.4, 0, 1, 1), opacity 0.32s ease-in;
}
.slide.exit-right .headline-anim { opacity: 0; transform: translateX(55vw); }
.slide.exit-right .scene-anim    { opacity: 0; transform: translateX(60vw) rotate(26deg); }
.slide.exit-right .promise-anim  { opacity: 0; transform: translateX(45vw); }
.slide.exit-right .tool-1 { opacity: 0; transform: rotate(-16deg) translateY(-110vh); }
.slide.exit-right .tool-2 { opacity: 0; transform: rotate(12deg)  translateY(-110vh); }
.slide.exit-right .tool-3 { opacity: 0; transform: rotate(-9deg)  translateY(-110vh); }
/* Reduced motion — instant, no spring, no fall */
@media (prefers-reduced-motion: reduce) {
  .anim, .tool.anim,
  .slide.exit-left .headline-anim, .slide.exit-left .scene-anim, .slide.exit-left .promise-anim,
  .slide.exit-right .headline-anim, .slide.exit-right .scene-anim, .slide.exit-right .promise-anim,
  .slide.exit-left .tool.anim, .slide.exit-right .tool.anim {
    transition-duration: 0s !important;
    transition-delay: 0s !important;
  }
}
```

### JavaScript: `goTo` (order is critical — do not reorder)

```js
function goTo(next, direction) {
  if (animating || next === current) return;
  animating = true;
  const currSlide = slides[current];
  const nextSlide = slides[next];
  const enterClass = direction === 'next' ? 'enter-right' : 'enter-left';
  const exitClass  = direction === 'next' ? 'exit-left'   : 'exit-right';
  // 1. Disable transitions, snap incoming to off-screen start
  const anims = nextSlide.querySelectorAll('.anim');
  anims.forEach(el => el.style.transition = 'none');
  nextSlide.classList.remove('active', 'exit-left', 'exit-right');
  nextSlide.classList.add(enterClass);
  // 2. Force reflow so the browser registers the start position
  void nextSlide.offsetHeight;
  // 3. Re-enable transitions on the next paint, then swap
  requestAnimationFrame(() => {
    anims.forEach(el => el.style.transition = '');
    currSlide.classList.remove('active');
    currSlide.classList.add(exitClass);
    nextSlide.classList.remove(enterClass);
    nextSlide.classList.add('active');
    setHeroBg(next);
    updateCounter(next);
    current = next;
    setTimeout(() => {
      currSlide.classList.remove(exitClass);
      animating = false;
    }, 850);
  });
}
```

### Background crossfade (dual fixed divs)

`#hero-bg-a` and `#hero-bg-b`: `position: absolute; inset: 0; transition: opacity
0.85s ease;` inside `.hero`.

```js
let activeBg = 'a';
function setHeroBg(i) {
  const g = heroBg[i];
  if (activeBg === 'a') { bgB.style.background = g; bgB.style.opacity = '1'; bgA.style.opacity = '0'; activeBg = 'b'; }
  else                  { bgA.style.background = g; bgA.style.opacity = '1'; bgB.style.opacity = '0'; activeBg = 'a'; }
}
```

### Controls + auto-advance

- Prev/Next arrows and `←` `→` arrow keys call `goTo()`. Counter format "01 / 04".
- Auto-advance every 5.5s (`goTo(next, 'next')`), looping. Pause on hover/focus
  within the hero, and on `document.visibilitychange` when hidden.
- Tear down the timer and key listener on `hashchange` away from `#/`; re-init
  on return.
- If `prefers-reduced-motion: reduce`: do not auto-advance; arrows do instant
  swaps only.

---

## Hooking it to the funnel (the "backend" part)

The animation itself is front-end, but it should feed the existing lead flow:

- The persistent hero CTA ("Book Your Free Quote") and the Caveat "FREE on-site
  quote" badge both link to `#/quote`, where the Cal.com embed + Formspree
  fallback already live.
- Add a `data-source` so you can see the hero is driving leads: append `?src=hero`
  to the quote link, and in the quote form include a hidden field
  `<input type="hidden" name="lead_source" value="hero_reset_slider">` (Formspree
  will pass it through in the email; harmless for the Cal.com booking). That's the
  one small backend touch — it tells you, in plain text in your inbox, that the
  new animation is the thing converting.

---

## Variant note

The stage→photo mapping above uses the real before/sort/after/details story the
uploaded photos already tell, which is more persuasive than a generic loop — it
*is* the sales pitch. If you'd rather the four slides be the three service tiers
instead, the structure is identical; only the copy table and photos change.
