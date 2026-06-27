# Asset guide — what each build needs, and where it comes from

The whole point of these sites is that the motion is choreographed *around* real visuals. Use this as a checklist when scoping the request in Step 1 — adapt the exact count to what the user describes, this is a starting point, not a rigid quota.

**Where the visuals come from:** if the user has their own images/video (uploaded, or clearly described brand assets), use those. Otherwise, the default is to automatically source and download matching stock photography/video from Pexels and Unsplash per `references/stock-assets.md` — not to ask the user to go find and upload photos, and not to fall back to flat color blocks. The counts/aspect-ratios below apply either way; they just tell you what to ask for (real assets) or what to go find (stock).

## `templates/hero-gallery.html` (hero video/image reveal + gallery)

- **1 hero video** (short, looping, muted — a few seconds is plenty) **or**, if no video, **1 large hero image** instead. If the user has one, ask which; otherwise source a looping clip or a strong hero shot matching the vibe — the template works with either (delete the unused `<video>`/`<img>` tag and its sibling).
- **4 supporting images** for the gallery flanking the hero media — 2 on the left column, 2 on the right. Mixed aspect ratios read well (e.g. one wide "large" shot + one squarer "small" shot per side), but 4 images of any reasonable photo aspect ratio will work.
- **Headline** (the big statement text — keep it short, 2–4 words reads best at hero scale).
- **Nav items** (2–4 links) and a **brand/logo wordmark**.
- **Accent color** and light vs. dark theme.

If the user has more or fewer images than 4, the gallery grid still works with 2 per side minimum — use their best 4 (or source that many), or extend the grid columns if there are more to include.

## `templates/split-flip-cards.html` (pinned flipping card row)

- **1 image per card**, ideally a consistent aspect ratio (portrait, roughly 2:3) since they sit in a row — 3 cards is the sweet spot for the fanned flip effect, but it generalizes to any odd number. Source a visually consistent set (same rough mood/color) if pulling from stock — a mismatched grab-bag of styles undercuts the effect.
- **Per card**: a short title/heading and a 1–2 sentence blurb for the back face, plus an optional index/number treatment ("01", "02"...).
- **Intro headline + subhead** for the full-screen intro section before the pinned cards.
- **Background treatment**: a solid color, gradient, or a subtle background image/texture behind the whole page.

## `templates/agency-scroll.html` (full multi-section agency/portfolio site)

This one's a richer, longer build — a full-screen nav, an asset-gated preloader, a marquee belt, a stack of service cards, and a magnetic-cursor project list. Scope it carefully since it asks for the most:

- **Brand/logo wordmark** + 3–5 **nav links**.
- **Hero headline** (short) and a one-line subhead.
- **Marquee phrase or word list** (3–6 short words/phrases, repeated automatically by the loop — e.g. a tagline, capabilities, or "Let's talk" repeated).
- **3–5 services/offerings**: a title + 1–2 sentence description each, for the sticky stacking section.
- **3–6 projects**: a title + 1 preview image each (consistent-ish aspect ratio helps but isn't required), for the magnetic-cursor project list. If the user has no real project shots (this is common — "projects" here are often illustrative, not a real client roster), source one strong stock image per project that matches its title/theme. An optional accent color per project powers the hover mood-color shift — sample one from each sourced image's general mood if the user doesn't specify.
- **Contact info**: email, and 2–4 social links.

If the user has fewer real assets than a section needs (e.g. only 2 real projects, or no marquee phrase in mind), fill the gap with sourced stock and/or drafted placeholder copy rather than leaving the section thin — say plainly when something is a draft or stock-sourced stand-in.

## Patterns that need their own assets, regardless of template

- **3D hero** (`references/3d-hero.md`): needs an actual `.glb`/`.gltf` 3D model — there's no good placeholder for a specific branded model. Ask explicitly before building one; don't default into a 3D hero just because a request sounds ambitious. If they don't have a model, offer the procedural-primitive fallback described in that reference, or suggest a quick path to getting a real one.
- **Velocity marquee / text scramble / mood-color shift**: text-only, no extra assets — just copy (the marquee phrase, the words to scramble) and, for mood-color shift, one accent color per item if they want it tied to specific colors rather than auto-picked ones.

## General questions worth asking up front

- Do they want **one** of these patterns/templates, or a **combined page** (e.g. hero-gallery section followed by a flip-cards section, or an agency-scroll site with a 3D hero swapped in for the flat one)?
- Any existing brand colors/fonts to match, or should the build pick something that fits the mood they describe?
- Rough length/scope — a single hero moment, or a longer multi-section scroll story?

Keep the actual ask in chat short and concrete — the headline/copy and vibe questions above double as the only real input you need, since visuals are now sourced automatically by default rather than requested.
