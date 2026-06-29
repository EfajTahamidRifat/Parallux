---
name: parallux
description: Build cinematic, award-winning, scroll-driven "Awwwards-quality" animated websites — the kind that win Site of the Day. Covers: Lenis smooth scroll + GSAP ScrollTrigger scrub timelines, per-letter/word mask reveals, WebGL noise-distortion image warping, interactive particle fields, blob/liquid cursor followers, SVG morphing preloaders with progress rings, flip cards, velocity marquees, magnetic cursor, sticky stacking cards, horizontal text drift, clip-path morphs, mood-color shifts, and real 3D/WebGL heroes. Use whenever the user asks for an "animated website", "Awwwards website", "scroll animation site", "agency site", "portfolio site", "hero animation", "GSAP landing page", "parallax site", "premium landing page", "WebGL site", "interactive website", or describes scroll-driven or cinematic motion — even without naming any library. Always produce the most visually striking version the request supports. Lean toward Lenis + GSAP for smooth scroll, add canvas-based particles or WebGL effects when they'd elevate the piece, and use the user's own uploaded assets when available, otherwise auto-source real stock photos/video from Pexels and Unsplash.
---

# Awwwards-Winning Animated Website Builder

This skill produces the kind of high-polish, scroll-driven, interaction-rich site that wins Awwwards Site of the Day. Every deliverable should feel hand-crafted and alive — Lenis smooth scroll that glides like butter, text that mask-reveals letter by letter, images that distort on scroll with WebGL noise shaders, cursors that bubble and follow with elastic easing, preloaders with SVG progress rings, and particle fields that scatter away from the mouse.

The engine stack: **Lenis** (smooth scroll) + **GSAP + ScrollTrigger** (timelines) + **canvas / Three.js** (particles, WebGL heroes, distortion). Single self-contained HTML file by default. React/Next.js component versions available via `references/nextjs-integration.md`.

---

## Step 1 — Nail scope before writing a line of code

Before any code, decide:

1. **Which pattern(s) fit — and lean ambitious.** Read `references/patterns.md`. Default templates: `templates/hero-gallery.html` (hero video/image gallery + clip-path reveal), `templates/split-flip-cards.html` (3D flip cards), `templates/agency-scroll.html` (full multi-section: preloader → marquee → stacking cards → project list). For open-ended asks, default to `agency-scroll.html` plus at least one NEW pattern from patterns.md (e.g. add WebGL distortion to the hero, or interactive particles to the contact section). **If asked for a "portfolio" or "creative agency" site, include a Lenis smooth scroll + custom cursor + blur-reveal on scroll as a baseline minimum.**

2. **Assets.** Check `references/asset-guide.md`. Use user uploads if available. Otherwise auto-download real stock from Pexels/Unsplash per `references/stock-assets.md`. Never use flat color placeholders as the default — a mediocre real photo beats a gray box.

3. **Copy.** Headline, subhead, nav links, brand name, card copy. Ask or draft punchy placeholder copy clearly labeled as a draft.

4. **Vibe.** Dark/moody vs. light/airy, accent color. One question. This drives both photography choice and the token system.

---

## Step 2 — Build the page

### Foundation (always include these on any "premium" build)

**Lenis smooth scroll** — wired to GSAP's ticker so ScrollTrigger stays synced:

```html
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.14/dist/lenis.min.js"></script>
<script>
const lenis = new Lenis({
  duration: 1.2,
  easing: t => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  smoothWheel: true,
  touchMultiplier: 1.5,
});
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add(time => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
</script>
```

**Custom cursor** (always include on desktop builds — it's a signal quality marker):

```html
<div class="cursor" id="cursor"></div>
<div class="cursor-follower" id="cursorFollower"></div>
<style>
  .cursor, .cursor-follower {
    position: fixed; top: 0; left: 0; border-radius: 50%;
    pointer-events: none; z-index: 9999; transform: translate(-50%, -50%);
    will-change: transform;
  }
  .cursor { width: 8px; height: 8px; background: #fff; mix-blend-mode: difference; }
  .cursor-follower {
    width: 36px; height: 36px; border: 1px solid rgba(255,255,255,0.4);
    transition: width 0.3s, height 0.3s, background 0.3s;
  }
  body:hover .cursor { opacity: 1; }
  /* Expand follower on hover of interactive elements */
  a:hover ~ #cursorFollower, button:hover ~ #cursorFollower { width: 56px; height: 56px; background: rgba(255,255,255,0.1); }
</style>
<script>
const cursor = document.getElementById('cursor');
const follower = document.getElementById('cursorFollower');
const xTo = gsap.quickTo(cursor, 'x', { duration: 0.1, ease: 'power3.out' });
const yTo = gsap.quickTo(cursor, 'y', { duration: 0.1, ease: 'power3.out' });
const fxTo = gsap.quickTo(follower, 'x', { duration: 0.5, ease: 'power3.out' });
const fyTo = gsap.quickTo(follower, 'y', { duration: 0.5, ease: 'power3.out' });
window.addEventListener('mousemove', e => { xTo(e.clientX); yTo(e.clientY); fxTo(e.clientX); fyTo(e.clientY); });

// Blob/elastic cursor on hover — inspired by Truus.co Awwwards winner
document.querySelectorAll('a, button, [data-cursor]').forEach(el => {
  el.addEventListener('mouseenter', () => gsap.to(follower, { scale: 2, duration: 0.4, ease: 'elastic.out(1, 0.4)' }));
  el.addEventListener('mouseleave', () => gsap.to(follower, { scale: 1, duration: 0.3, ease: 'sine.inOut' }));
});
</script>
```

### Techniques — choose what fits, combine freely

Read `references/patterns.md` for all snippets. The techniques are:

**Core scroll mechanics**
- Pinned scrub timeline
- Staggered fly-in
- Clip-path mask morph (Sondr-style hero shrink)
- Sticky stacking cards
- Horizontal text drift
- Header blur-on-scroll

**Text effects**
- Split-text mask reveal (word-by-word or line-by-line, uses GSAP SplitText)
- Per-letter slide-in with BezierEasing (from musabhassan.com Awwwards winner)
- Text scramble reveal (glitchy decode)
- Blur-reveal on scroll (opacity + filter:blur, no library needed)

**Interactive / cursor**
- Magnetic cursor preview (project list hover)
- Elastic cursor bubble with context-aware label text (Truus.co pattern)
- Magnetic button pull effect

**Motion / atmosphere**
- Interactive particle field (canvas, mouse-repulsion, theme-aware — from Kintaro Awwwards winner)
- Velocity marquee (infinite loop + scroll speed-up + direction reversal)
- Hover mood-color shift

**WebGL / advanced**
- WebGL noise-distortion on images (simplex noise vertex + RGB-shift fragment shader — from musabhassan.com)
- SVG curved loader wipe (bezier curve path morph — from portfolio-main Awwwards winner)
- 3D hero — see `references/3d-hero.md`

**Loading**
- SVG progress ring preloader (animated strokeDashoffset — from Kintaro Awwwards winner)
- Asset-gated preloader with real progress tracking
- Blur+scale entrance after load (blur(10px)→blur(0), scale(0.8)→scale(1))

---

## Step 3 — Build rules

1. Copy the nearest template from `templates/` as your starting point. Read it fully first.
2. Source visuals before wiring up anything else.
3. All color/type variables go in a `:root {}` block at the top — edit values there, not scattered through the file.
4. Keep self-contained: inline `<style>` and `<script>`, relative asset paths.
5. GSAP 3.13+ — SplitText, ScrollTrigger, Observer are free (Webflow acquisition). No license key needed.
6. Fonts: system font stack by default (renders in-chat artifact). Mention Google Fonts tradeoff if user wants a specific typeface.
7. **Performance guard**: wrap `(hover: hover)` / `window.innerWidth > 768` around cursor effects and magnetic effects — touch devices have no cursor.
8. **`will-change: transform`** on animated elements. **`force3D: true`** on 3D flip tweens.
9. WebGL / canvas: always use `requestAnimationFrame` loop, respect `prefers-reduced-motion`, pause the loop when the element leaves the viewport (IntersectionObserver).

---

## Step 4 — Deliver

Save to `/mnt/user-data/outputs/<descriptive-name>.html`, copy `assets/` alongside it, call `present_files`. The HTML artifact also renders live in-chat.

For Next.js/React integration, read `references/nextjs-integration.md`.

---

## Reference map

- `references/patterns.md` — every animation technique with copy-paste snippets, including the new WebGL distortion, interactive particles, SVG loader, blur-reveal, elastic cursor, letter slide-in, and curved loader wipe patterns added from the Awwwards-winning codebases.
- `references/stock-assets.md` — find/download Pexels + Unsplash images/video.
- `references/3d-hero.md` — Three.js single-file and React Three Fiber hero implementations.
- `references/asset-guide.md` — what each pattern/template needs.
- `references/nextjs-integration.md` — React/Tailwind/GSAP component versions.
- `templates/hero-gallery.html` — blur-on-scroll nav, image gallery, pinned mask reveal, split-text headline.
- `templates/split-flip-cards.html` — split-text intro, pinned 3D flip card row.
- `templates/agency-scroll.html` — full multi-section: asset-gated preloader, slide-in nav, velocity marquee, sticky stacking cards, magnetic-cursor project list.
