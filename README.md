# Parallux

**A Claude Skill that builds cinematic, scroll-driven, Awwwards-caliber websites — and sources its own stock photography while it's at it.**

Parallux teaches Claude how to build the kind of high-polish, scroll-driven one-page sites you'd see win Awwwards Site of the Day: heroes that morph as you scroll, text that decodes letter by letter, images that warp with WebGL noise on scroll, particle fields that scatter from the cursor, elastic cursor bubbles, SVG progress ring preloaders, infinite marquees that speed up with your scroll, magnetic buttons, and an optional 3D/WebGL hero — all stitched together from a focused, well-documented set of GSAP/ScrollTrigger/canvas techniques.

## What's new in this release

This version was enhanced by deep-studying 6 Awwwards-winning/nominated open-source codebases and extracting their core techniques as documented, framework-agnostic patterns:

| Source | Techniques extracted |
|---|---|
| **musabhassan.com** | WebGL simplex-noise image distortion + RGB-shift fragment shader; per-letter slide-in with BezierEasing |
| **Truus.co** | Elastic cursor bubble with context labels; Lenis smooth scroll wired to GSAP ticker; tab-visibility easter egg |
| **Kintaro** | Interactive particle field (canvas + mouse repulsion, auto-pauses off-screen); SVG progress ring preloader; blur-reveal on scroll |
| **portfolio-main** | SVG curved loader wipe (bezier path morph); magnetic button pull with elastic reset |
| **webgl-portfolio** | WebGL ball / Three.js shader reference |
| **Original Parallux** | GSAP ScrollTrigger patterns, templates, stock asset pipeline |

**10 new patterns added to `patterns.md`:**

1. Lenis smooth scroll (wired to GSAP ticker, exponential easing)
2. Custom cursor with elastic follower (`gsap.quickTo`, mix-blend-mode: difference)
3. Elastic cursor bubble with context label ("click", "drag", "view") — Truus.co
4. Per-letter slide-in with BezierEasing — musabhassan.com
5. Blur-reveal on scroll (`opacity + filter:blur → clear`, IntersectionObserver) — Kintaro
6. WebGL noise-distortion images (simplex noise vertex shader + RGB-shift fragment shader) — musabhassan.com
7. Interactive particle field (canvas, mouse repulsion, IntersectionObserver pause) — Kintaro
8. SVG progress ring preloader (animated `strokeDashoffset`, blur+scale entrance) — Kintaro
9. SVG curved loader wipe (bezier path morph, wave lifts away) — portfolio-main
10. Magnetic button pull with elastic reset

## Why this exists

Most "make me an animated website" prompts produce something flat: a generic template with a fade-in or two. Parallux exists to close that gap — it encodes the actual techniques that separate a competent landing page from one that feels expensive, plus the judgment calls around pacing, easing, and mobile fallbacks that usually only come from having built a few dozen of these by hand.

It also removes the most common bottleneck: waiting on the user to round up photos. By default, Parallux automatically sources real, freely-licensed stock photography and video from Pexels and Unsplash.

## Features

- **Three ready-to-customize templates** — a hero/gallery video reveal, a pinned 3D flip-card row, and a full multi-section "agency" build — each a complete, self-contained HTML file.
- **A documented pattern library** covering 20+ techniques: pinned scrub timelines, split-text reveals, 3D card flips, clip-path mask morphs, header blur-on-scroll, staggered fly-ins, text-scramble reveals, velocity marquees, magnetic cursor previews, slide-in nav overlays, sticky stacking cards, horizontal text drift, hover mood-color shifts, asset-gated preloaders — plus the 10 new patterns above.
- **Lenis smooth scroll** as a baseline on every premium build, wired correctly to GSAP.
- **Custom cursor** (two-layer dot + ring with elastic hover expand) included by default on desktop builds.
- **WebGL distortion** on images via a full simplex-noise vertex shader + RGB-shift fragment shader.
- **Canvas particle fields** that react to the mouse cursor and auto-pause when off-screen.
- **Automatic stock asset sourcing** from Pexels and Unsplash (real downloads, not hotlinks).
- **An optional 3D/WebGL hero** — vanilla Three.js for the single-file path, React Three Fiber for the Next.js path.
- **A Next.js/React integration path** for every pattern.
- **Zero build step by default.** One self-contained `.html` file loading everything from CDN.

## Installation

Parallux is a standard [Agent Skill](https://github.com/anthropics/skills) — a folder with a `SKILL.md` plus reference docs and templates.

**claude.ai**
1. Zip the `parallux/` folder (the zip must contain the folder itself at its root, not just the loose files inside it) — or grab a pre-built `parallux.skill` from this repo's [Releases](../../releases).
2. Settings → Features → Skills → Upload.
3. Enable code execution in Settings → Capabilities (Parallux downloads stock assets and writes files).

**Claude Code / Claude Desktop**
```bash
# personal install — available in every project
git clone https://github.com/YOUR_USERNAME/Parallux.git ~/.claude/skills/parallux

# OR project-scoped — committed to one repo, shared via git
git clone https://github.com/YOUR_USERNAME/Parallux.git .claude/skills/parallux
```
Restart your session afterward — skills are loaded at startup.

**Other Agent Skills-compatible tools** (Cursor, Gemini CLI, etc.) — copy the `parallux/` folder into whatever skills directory that tool reads from.

## Usage

Once installed, just describe what you want:

> "Build me an animated portfolio site for a photography studio, dark and moody, with a marquee and a project list."

> "Make a scroll site for my SaaS landing page — hero video, then a row of flip cards for features."

> "I want a full agency site with a WebGL distortion hero, particle field, preloader, nav, services, work section."

> "Build me something that could win Awwwards."

Parallux will scope the request, source matching stock photography/video automatically, build the page, and hand back a working `.html` file plus its `assets/` folder.

## Project structure

```
parallux/
├── SKILL.md                        # entry point — when to trigger, how to scope, how to build
├── references/
│   ├── patterns.md                 # 20+ animation techniques with copy-paste snippets
│   ├── stock-assets.md             # sourcing & downloading from Pexels/Unsplash
│   ├── asset-guide.md              # what each template/pattern needs
│   ├── 3d-hero.md                  # optional 3D/WebGL hero (Three.js + React Three Fiber)
│   └── nextjs-integration.md       # React/useGSAP component versions of every pattern
└── templates/
    ├── hero-gallery.html           # hero video/image reveal + flanking gallery
    ├── split-flip-cards.html       # pinned row of cards that flip in 3D
    └── agency-scroll.html          # full multi-section site combining most patterns
```

## How asset sourcing works

When you don't provide your own images or video, Parallux searches Pexels and Unsplash for matching photography, resolves the direct CDN URL, and downloads it locally into `assets/images/` and `assets/videos/`. Both platforms license their libraries for free use, including commercial projects. Full details in [`references/stock-assets.md`](./parallux/references/stock-assets.md).

## Roadmap / contributing

Parallux grows by extracting good techniques from real reference codebases. See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for how to propose a new pattern or template.

Ideas not yet built in: page transition animations (clip-path wipes between routes), audio-reactive motion, CMS/Astro integration, scroll-jacked horizontal sections, an eval suite for the skill itself.

## License

[MIT](./LICENSE) — use it, fork it, ship it.

## Acknowledgments

Enhanced by studying techniques from the following open-source Awwwards-winning/nominated codebases: musabhassan.com, Truus.co, Kintaro, and portfolio-main. All patterns were rewritten as framework-agnostic, documented snippets rather than copied verbatim. GSAP and its plugins are made by [GreenSock](https://gsap.com) and are free for commercial use since Webflow's acquisition. Lenis is made by [Studio Freight](https://github.com/darkroomengineering/lenis). Three.js and React Three Fiber power the optional 3D hero. Stock imagery comes from [Pexels](https://www.pexels.com) and [Unsplash](https://unsplash.com) under their respective free-use licenses.
