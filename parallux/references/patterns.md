# Core animation patterns

Everything in `templates/` is built from a small set of recombinable techniques. This file explains each one — what it does, why it's built that way, and the snippet to lift if you're composing something the templates don't already cover. All of it assumes `gsap`, `ScrollTrigger`, and (where noted) `SplitText` are loaded and registered:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/ScrollTrigger.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/SplitText.min.js"></script>
<script>gsap.registerPlugin(ScrollTrigger, SplitText);</script>
```

| Effect the user wants | Pattern |
|---|---|
| "Pin this section while stuff happens" | Pinned scrub timeline |
| Headline text that slides/fades in word-by-word or line-by-line | Split-text reveal |
| Cards/panels that flip over in 3D on scroll | 3D card flip |
| A video/image that morphs into a smaller shape elsewhere on the page | Clip-path mask morph |
| Nav bar that goes glassy/frosted once you start scrolling | Header blur-on-scroll |
| Elements flying in from the sides as a section is revealed | Staggered fly-in |
| Text that resolves out of random/glitchy characters into real words | Text scramble reveal |
| Infinite logo/word belt that speeds up or reverses with scroll | Velocity marquee (horizontalLoop + Observer) |
| A floating image follows the cursor as you hover a list of rows | Magnetic cursor preview |
| A full-screen (or half-screen) menu slides in over the page | Slide-in nav overlay |
| Cards that stack on top of each other while scrolling past | Sticky stacking cards |
| Big text rows gliding sideways at different speeds as you scroll past | Horizontal text drift |
| Page background subtly retints depending on what's hovered | Hover mood-color shift |
| A loading screen with a real percentage before the page reveals itself | Asset-gated preloader |
| A rotating/floating 3D object as the hero centerpiece | 3D hero — see `references/3d-hero.md` |

## Pinned scrub timeline

The backbone of every scrollytelling section. The section is pinned (frozen in the viewport) while a timeline scrubs in lockstep with scroll position, instead of playing on a clock.

```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: section,
    start: "top top",
    end: "+=120%",      // how much extra scroll distance the animation "owns" — bigger = slower/more deliberate
    scrub: 1,            // 0 = perfectly tied to scrollbar, 0.5–1 = a touch of lag that feels smoother
    pin: true,
    anticipatePin: 1,    // prevents a visible jump right as pinning kicks in
    invalidateOnRefresh: true, // recompute any function-based values (e.g. clip-path insets) on resize
  },
});

tl.to(".thing-a", { scale: 0.8 }, 0);     // the trailing 0 = "start at the very beginning of the timeline"
tl.to(".thing-b", { opacity: 0 }, 0);     // multiple tweens at position 0 run in parallel
tl.from(".thing-c", { x: "-200%" }, "-=0.6"); // overlap with whatever came before by 0.6s of timeline time
```

The `end` distance is the main lever for pacing — too short and the effect feels rushed/jumpy, too long and scrolling feels like it's dragging. 100–400% of viewport height is the usual range; cards/flips that need a held "pause" at the end benefit from appending an empty tween (`tl.to({}, { duration: 2 }, lastPosition)`) so the pin doesn't release the instant the visible motion finishes.

## Split-text reveal

Splits text into words or lines, masks the overflow, and animates each piece in with a stagger — the "type each word in" look used in both templates.

```js
const split = SplitText.create(headlineEl, { type: "words", mask: "words" });
gsap.set(split.words, { opacity: 0, yPercent: 110 });
gsap.to(split.words, {
  yPercent: 0,
  opacity: 1,
  stagger: 0.05,
  ease: "power2.inOut",
  duration: 1,
});
```

For multi-line headlines, `type: "lines"` with `autoSplit: true` re-splits automatically on resize/font-load (important since line breaks shift with width):

```js
SplitText.create(headlineEl, {
  type: "lines",
  mask: "lines",
  autoSplit: true,
  onSplit(self) {
    return gsap.from(self.lines, { y: 420, duration: 0.8 });
  },
});
```

`mask: "words"/"lines"` is what makes each piece slide up *out of* a clipped box rather than just fading in place — it's doing most of the "premium" feel here. Drop the `mask` option if you want a plainer fade-stagger instead.

## 3D card flip

A card flips like a playing card: a 3D-perspective shell containing front/back faces, with the inner element's `rotationY` driven from 0 → 180.

```html
<div class="card-shell">                 <!-- has perspective -->
  <div class="card-flip-inner">          <!-- the thing that actually rotates -->
    <div class="card-face front"><img src="..."></div>
    <div class="card-face back">…back content…</div>
  </div>
</div>
```

```css
.card-shell { perspective: 1000px; }
.card-flip-inner { transform-style: preserve-3d; position: relative; }
.card-face { position: absolute; inset: 0; backface-visibility: hidden; }
.card-face.back { transform: rotateY(180deg); }
```

```js
gsap.set(flipInner, { rotationY: 0, transformStyle: "preserve-3d" });
gsap.set(backFace, { visibility: "hidden" }); // hide back face until mid-flip so it doesn't show through

const flipTl = gsap.timeline({ paused: true });
flipTl.to(flipInner, { rotationY: 180, duration: 0.8, ease: "power2.inOut", force3D: true });
flipTl.set(frontFace, { visibility: "hidden" }, 0.4); // swap visibility at the halfway point,
flipTl.set(backFace, { visibility: "visible" }, 0.4); // once the face showing is edge-on anyway
```

Drive `flipTl` from a scroll-progress threshold rather than scrubbing it directly, so the flip commits fully rather than getting stuck mid-rotation:

```js
let flipped = false;
// inside the pinned timeline's scrollTrigger:
onUpdate(self) {
  if (self.progress > 0.5 && !flipped) { flipped = true; flipTl.play(); }
  else if (self.progress <= 0.5 && flipped) { flipped = false; flipTl.reverse(); }
}
```

For a row of N cards with a fanned-out look once flipped, derive each card's rotation/offset from its distance to the center card instead of hardcoding per-card values — this scales cleanly to any card count:

```js
function flipParamsFor(index, total) {
  const center = (total - 1) / 2;
  const offset = index - center; // negative = left of center, positive = right
  return {
    rotateZ: offset * 6,
    moveX: offset * -70,
    moveY: offset === 0 ? -18 : 20,
  };
}
```

## Clip-path mask morph

Used for "the hero video/image shrinks and reshapes into a smaller frame elsewhere on the page" — the signature Sondr-style hero effect. The trick is computing the *target* shape from a real element already sitting in its final position (often an invisible spacer the size you want), then animating `clip-path: inset(...)` toward it.

```js
function getInset() {
  const heroRect = heroEl.getBoundingClientRect();
  const targetRect = targetSpacerEl.getBoundingClientRect();
  return {
    top: targetRect.top - heroRect.top,
    left: targetRect.left - heroRect.left,
    right: heroRect.right - targetRect.right,
    bottom: heroRect.bottom - targetRect.bottom,
  };
}

tl.to(videoWrapperEl, {
  clipPath: () => {
    const i = getInset();
    return `inset(${i.top}px ${i.right}px ${i.bottom}px ${i.left}px round 12px)`;
  },
  ease: "power2.out",
  duration: 1,
}, 0);
```

Because `clipPath` is a function here (not a fixed value), it's recalculated on every scrub tick — pair this with `invalidateOnRefresh: true` on the ScrollTrigger so it also stays correct after a window resize.

## Header blur-on-scroll

A short, separate ScrollTrigger (not part of the main pinned timeline) that turns the nav glassy as soon as the page starts moving:

```js
gsap.to("header", {
  backdropFilter: "blur(30px)",
  backgroundColor: "rgba(255,255,255,0.15)",
  scrollTrigger: { trigger: "main", start: "top top", end: "+=100", scrub: 1 },
});
```

Keep the trigger distance short (`+=100`) — this should resolve almost immediately on scroll, not stay mid-transition for a long stretch.

## Staggered fly-in

Elements entering from off-screen as part of a pinned timeline, timed to overlap the tail end of whatever animated just before them:

```js
tl.from(".col-left img", { x: "-200%", y: "150%", stagger: 0.15, ease: "power3.out", duration: 1 }, "-=0.6");
tl.from(".col-right img", { x: "200%", y: "150%", stagger: 0.15, ease: "power3.out", duration: 1 }, "-=0.8");
```

Opposing X directions for left/right groups plus a shared upward Y motion reads as things "settling into place" rather than just sliding in from one side.

## Text scramble reveal

Headline or nav text that resolves out of random characters into the real word — a glitchy, "decoding" feel used for preloader counters, nav links, and hero word-reveals. No dependency needed; this is ~15 lines of vanilla JS.

```js
function scrambleText(el, { duration = 1200, charset = "ABCDEFGHIJKLMNOPQRSTUVWXYZ" } = {}) {
  const original = el.textContent;
  const totalFrames = Math.round(duration / 33); // ~30fps
  let frame = 0;
  const tick = () => {
    frame++;
    const revealCount = Math.floor((frame / totalFrames) * original.length);
    el.textContent = original
      .split("")
      .map((ch, i) => (ch === " " || i < revealCount ? ch : charset[(Math.random() * charset.length) | 0]))
      .join("");
    if (frame < totalFrames) requestAnimationFrame(tick);
    else el.textContent = original; // guarantee the real text lands exactly, no drift
  };
  tick();
}

// fire on load, or stagger across several elements:
document.querySelectorAll("[data-scramble]").forEach((el, i) => {
  setTimeout(() => scrambleText(el), i * 150);
});
```

Revealing left-to-right (as above) reads as "decoding"; for a noisier look, shuffle every character on every frame instead of gating by `revealCount`, and only lock in the real text on the final frame. Keep `duration` short (600–1500ms) — this is a flourish, not the main event, and it shouldn't make the user wait to read a headline.

## Velocity marquee (infinite loop + scroll speed-up)

An infinitely-looping horizontal strip of words/logos that subtly speeds up — and reverses direction — in response to scroll velocity. This is the canonical GSAP "horizontal loop" technique, wired to the `Observer` plugin so the belt reacts to how the user is scrolling instead of running at one flat speed forever. Needs `Observer` loaded alongside the usual plugins:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/Observer.min.js"></script>
<script>gsap.registerPlugin(Observer);</script>
```

```js
function horizontalLoop(items, { speed = 1, repeat = -1, paddingRight = 0 } = {}) {
  items = gsap.utils.toArray(items);
  const tl = gsap.timeline({
    repeat,
    defaults: { ease: "none" },
    onReverseComplete: () => tl.totalTime(tl.rawTime() + tl.duration() * 100),
  });
  const widths = [], xPercents = [];
  const pixelsPerSecond = speed * 100;
  let totalWidth;

  gsap.set(items, {
    xPercent: (i, el) => {
      widths[i] = parseFloat(gsap.getProperty(el, "width", "px"));
      return (xPercents[i] = (parseFloat(gsap.getProperty(el, "x", "px")) / widths[i]) * 100 + gsap.getProperty(el, "xPercent"));
    },
  });
  gsap.set(items, { x: 0 });

  const last = items.length - 1;
  totalWidth = items[last].offsetLeft + (xPercents[last] / 100) * widths[last]
    - items[0].offsetLeft + items[last].offsetWidth * gsap.getProperty(items[last], "scaleX") + paddingRight;

  items.forEach((item, i) => {
    const curX = (xPercents[i] / 100) * widths[i];
    const distanceToStart = item.offsetLeft + curX - items[0].offsetLeft;
    const distanceToLoop = distanceToStart + widths[i] * gsap.getProperty(item, "scaleX");
    tl.to(item, { xPercent: ((curX - distanceToLoop) / widths[i]) * 100, duration: distanceToLoop / pixelsPerSecond }, 0)
      .fromTo(item,
        { xPercent: ((curX - distanceToLoop + totalWidth) / widths[i]) * 100 },
        { xPercent: xPercents[i], duration: (curX - distanceToLoop + totalWidth - curX) / pixelsPerSecond, immediateRender: false },
        distanceToLoop / pixelsPerSecond);
  });

  tl.progress(1, true).progress(0, true); // pre-render so there's no first-frame jump
  return tl;
}

const marqueeTl = horizontalLoop(".marquee-item", { repeat: -1, paddingRight: 30 });

Observer.create({
  onChangeY(self) {
    const factor = (self.deltaY < 0 ? 1 : -1) * 2.5;
    gsap.timeline({ defaults: { ease: "none" } })
      .to(marqueeTl, { timeScale: factor * 2.5, duration: 0.2, overwrite: true })
      .to(marqueeTl, { timeScale: factor / 2.5, duration: 1 }, "+=0.3");
  },
});
```

Mark up the strip as one row of repeated `.marquee-item`s — duplicate the list 2–3 times in the HTML so the loop never visibly runs out — wrapped in `overflow: hidden; white-space: nowrap;`. Two marquees moving opposite directions, stacked, reads great as a contact-section flourish; call `marqueeTl.reverse()` once right after creating the second one so it starts going the other way.

## Magnetic cursor preview

A list of rows (services, projects) where hovering a row reveals a floating image that smoothly follows the cursor — a signature portfolio "Work list" effect. Uses `gsap.quickTo` for the follow, which is far cheaper than re-tweening x/y from scratch on every `mousemove`, plus a clip-path wipe for the row's own hover state.

```html
<div class="project-list">
  <div class="project-row" data-preview-src="images/project-1.jpg">
    <span class="overlay"></span>
    <h3>Project One</h3>
  </div>
  <!-- ...more rows... -->
</div>
<div class="preview-image"><img src="" alt=""></div>
```

```css
.project-row { position: relative; cursor: pointer; }
.project-row .overlay {
  position: absolute; inset: 0; background: #000; z-index: -1;
  clip-path: polygon(0 100%, 100% 100%, 100% 100%, 0 100%);
}
.preview-image {
  position: fixed; top: 0; left: 0; width: 320px; pointer-events: none;
  z-index: 50; opacity: 0; overflow: hidden;
}
.preview-image img { width: 100%; display: block; }
```

```js
const preview = document.querySelector(".preview-image");
const previewImg = preview.querySelector("img");
const moveX = gsap.quickTo(preview, "x", { duration: 0.6, ease: "power3.out" });
const moveY = gsap.quickTo(preview, "y", { duration: 0.8, ease: "power3.out" });

document.addEventListener("mousemove", (e) => {
  moveX(e.clientX + 24);
  moveY(e.clientY + 24);
});

document.querySelectorAll(".project-row").forEach((row) => {
  row.addEventListener("mouseenter", () => {
    previewImg.src = row.dataset.previewSrc;
    gsap.to(row.querySelector(".overlay"), { clipPath: "polygon(0 0, 100% 0, 100% 100%, 0 100%)", duration: 0.4, ease: "power2.out" });
    gsap.to(preview, { opacity: 1, scale: 1, duration: 0.3 });
  });
  row.addEventListener("mouseleave", () => {
    gsap.to(row.querySelector(".overlay"), { clipPath: "polygon(0 100%, 100% 100%, 100% 100%, 0 100%)", duration: 0.3, ease: "power2.in" });
    gsap.to(preview, { opacity: 0, scale: 0.95, duration: 0.3 });
  });
});
```

Guard this behind a pointer check (`matchMedia("(hover: hover)").matches`, or simply `window.innerWidth > 768`) and fall back to a static inline image inside the row on touch devices — there's no cursor to follow on mobile, so don't leave the effect half-wired there.

## Slide-in nav overlay

A full-screen (or half-screen) menu panel that slides in from off-canvas, with its own staggered link reveal and a separate timeline that morphs the hamburger icon into an X. Keep the panel and icon as two independent paused timelines, played/reversed together — they're easier to tune with different easing that way than as one combined timeline.

```css
.nav-panel {
  position: fixed; top: 0; right: 0; height: 100%; width: 100%; max-width: 480px;
  background: #000; color: #fff; z-index: 40; transform: translateX(100%);
}
.nav-panel a { display: block; font-size: 3rem; }
.burger {
  position: fixed; top: 1.5rem; right: 1.5rem; z-index: 50; width: 44px; height: 44px;
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 6px; cursor: pointer;
}
.burger span { width: 26px; height: 2px; background: #fff; transform-origin: center; }
```

```js
gsap.set(".nav-panel", { xPercent: 100 });
gsap.set([".nav-panel a", ".nav-panel .contact-line"], { autoAlpha: 0, x: -20 });

const panelTl = gsap.timeline({ paused: true })
  .to(".nav-panel", { xPercent: 0, duration: 0.8, ease: "power3.out" })
  .to(".nav-panel a", { autoAlpha: 1, x: 0, stagger: 0.08, duration: 0.5, ease: "power2.out" }, "<");

const iconTl = gsap.timeline({ paused: true })
  .to(".burger span:first-child", { rotate: 45, y: 4, duration: 0.3, ease: "power2.inOut" })
  .to(".burger span:last-child", { rotate: -45, y: -4, duration: 0.3, ease: "power2.inOut" }, "<");

let open = false;
document.querySelector(".burger").addEventListener("click", () => {
  open ? (panelTl.reverse(), iconTl.reverse()) : (panelTl.play(), iconTl.play());
  open = !open;
});
```

For a half-screen panel (page content still visible beside it) instead of full-bleed, just change `.nav-panel`'s `width`/`max-width` — the slide math (`xPercent: 100 → 0`) stays identical either way.

## Sticky stacking cards

Cards (service offerings, feature blocks) that each become `position: sticky` and visually stack on top of one another as the user scrolls past — each one pinned a little lower than the last, so the stack reads like a deck of cards being dealt downward.

```css
.stack-card { position: sticky; padding: 3rem 2rem; border-top: 1px solid rgba(255,255,255,0.2); }
```

```js
document.querySelectorAll(".stack-card").forEach((card, i) => {
  card.style.top = `calc(8vh + ${i * 2.5}rem)`; // each card sticks a bit lower than the one before it

  gsap.from(card, {
    y: 120,
    opacity: 0,
    duration: 1,
    ease: "circ.out",
    scrollTrigger: { trigger: card, start: "top 85%" },
  });
});
```

The stacking illusion comes entirely from CSS (`position: sticky` + an increasing `top`) — GSAP here only handles each card's entrance, not the stacking itself. Don't pin the *container*; pin each card independently, and don't cap the container's height beyond what its content naturally needs — `sticky` requires room to scroll within its parent to work at all.

## Horizontal text drift

Several rows of big text, each gliding a different distance and direction as the user scrolls past — a quieter, non-looping cousin of the velocity marquee. Good for a "services" or "capabilities" word-cloud section.

```js
document.querySelectorAll("[data-drift]").forEach((row) => {
  gsap.to(row, {
    xPercent: parseFloat(row.dataset.drift), // e.g. 20, -30, 100, -100 per row
    ease: "none",
    scrollTrigger: { trigger: row, scrub: true },
  });
});
```

Alternate signs and magnitudes per row (e.g. `20, -30, 100, -100`) rather than a uniform drift — the unevenness is what makes it feel alive instead of mechanical. Set `overflow-x: clip` on a parent ancestor so the drifting rows don't introduce a horizontal scrollbar.

## Hover mood-color shift

The page's background subtly retints depending on which item (a project row, an image) the user is currently hovering — a cheap but effective way to make a list feel connected to its imagery without an actual color-extraction step. Tag each item with its own accent color and transition the `<html>` background on a plain CSS transition (not GSAP), so it stays smooth even under heavy scroll-driven GSAP load elsewhere on the page.

```css
html { transition: background-color 0.6s cubic-bezier(0.19, 1, 0.22, 1); }
```

```html
<div class="project-row" data-mood="#1a354e">...</div>
```

```js
document.querySelectorAll("[data-mood]").forEach((row) => {
  row.addEventListener("mouseenter", () => { document.documentElement.style.backgroundColor = row.dataset.mood; });
  row.addEventListener("mouseleave", () => { document.documentElement.style.backgroundColor = ""; }); // back to the CSS default
});
```

Pick moods dark/desaturated enough that they don't fight with foreground text contrast — this is meant to be felt at the edges of the screen, not read.

## Asset-gated preloader

A loading screen that tracks real image/video load progress (rather than faking a count-up) and only releases once everything the first viewport needs is ready — this avoids the "content pops in and reflows" flash that undermines the premium feel these sites are going for. Pair the percentage display with a quick **text scramble reveal** (above) on the brand name once it hits 100 for a nice finishing touch.

```html
<div class="loader"><span class="loader-pct">0</span></div>
```

```js
function gateOnAssetsLoaded(onProgress, onDone) {
  const assets = [...document.querySelectorAll("img, video")];
  if (assets.length === 0) return onDone();
  let loaded = 0;
  assets.forEach((el) => {
    const mark = () => { loaded++; onProgress(loaded / assets.length); if (loaded === assets.length) onDone(); };
    if (el.tagName === "IMG") el.complete ? mark() : el.addEventListener("load", mark, { once: true });
    else el.readyState >= 3 ? mark() : el.addEventListener("loadeddata", mark, { once: true });
    el.addEventListener("error", mark, { once: true }); // never let one broken asset hang the loader forever
  });
}

const pct = document.querySelector(".loader-pct");
gateOnAssetsLoaded(
  (fraction) => { pct.textContent = Math.floor(fraction * 100); },
  () => {
    gsap.timeline()
      .to(".loader", { yPercent: -100, duration: 0.8, ease: "power3.inOut" })
      .from(".hero-content", { opacity: 0, y: 40, duration: 1 }, "-=0.3");
  }
);
```

Cap how long this can run — race the real gate against a hard timeout (e.g. `setTimeout(onDone, 6000)`) so a slow connection can't block the page indefinitely. For pages with very few/light images, a real asset gate is overkill — a simple 1–2s simulated count-up (`gsap.to({}, { duration: 1.5, onUpdate: () => { pct.textContent = Math.floor(this.progress() * 100) } })`) is fine instead.

## 3D / WebGL hero (optional)

For a hero built around an actual 3D object (a model that rotates/floats, GPU-rendered) instead of a flat image/video, see `references/3d-hero.md` — it covers both a single-file Three.js version (CDN, matches this skill's default deliverable) and a React Three Fiber version (for the Next.js integration path). Only reach for this when the user has, or explicitly wants, a real 3D model — it's a meaningfully bigger lift than the CSS/image-based patterns above and needs a `.glb`/`.gltf` asset to be worth it.

## General notes

- **Easing**: `power2.inOut`/`power2.out`/`power3.out` cover almost everything here — reach for something fancier only if the user asks for a specific feel (bouncy, elastic, etc.).
- **`force3D: true`** on any transform-heavy tween (especially 3D flips) keeps it GPU-composited and smooth.
- **Test the reverse direction.** Scrubbed timelines run backward when the user scrolls up — always sanity-check that reversing the effect still looks intentional, not broken (this is why the card-flip threshold pattern explicitly handles both directions).
- **Hover-dependent effects** (magnetic cursor preview, mood-color shift, hover overlays) need a `matchMedia("(hover: hover)")` or `window.innerWidth` guard and a touch-friendly fallback — there's no cursor on mobile, so don't ship the effect half-wired there.
