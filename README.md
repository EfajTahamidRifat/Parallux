# Parallux

**A Claude Skill that builds cinematic, scroll-driven, award-show-caliber websites — and sources its own stock photography while it's at it.**

Parallux teaches Claude how to build the kind of high-polish, scroll-driven one-page sites you'd see featured on a design showcase: heroes that morph as you scroll, headlines that scramble into place, infinite marquees that speed up with your scroll, full-screen nav menus, cursor-following project previews, sticky stacking cards, even an optional 3D/WebGL hero — all stitched together from a focused, well-documented set of GSAP/ScrollTrigger techniques. Point it at a Claude Code, Claude Desktop, or claude.ai session, describe the site you want, and it'll scope it, source real photos and video for it, and ship a complete, self-contained, working build.

## Why this exists

Most "make me an animated website" prompts produce something flat: a generic template with a fade-in or two. Parallux exists to close that gap — it encodes the actual techniques (pinned scrub timelines, split-text masks, velocity marquees, magnetic cursor previews) that separate a competent landing page from one that feels expensive, plus the judgment calls around pacing, easing, and mobile fallbacks that usually only come from having built a few dozen of these by hand.

It also removes the most common bottleneck in practice: waiting on the user to round up photos. By default, Parallux automatically sources real, freely-licensed stock photography and video from Pexels and Unsplash and downloads it straight into the project — no placeholder gray boxes, no stalling the build on an upload.

## Features

- **Three ready-to-customize templates** — a hero/gallery video reveal, a pinned 3D flip-card row, and a full multi-section "agency" build (preloader → nav → marquee → services → work) — each a complete, self-contained HTML file.
- **A documented pattern library** covering pinned scrub timelines, split-text reveals, 3D card flips, clip-path mask morphs, header blur-on-scroll, staggered fly-ins, text-scramble reveals, velocity marquees, magnetic cursor previews, slide-in nav overlays, sticky stacking cards, horizontal text drift, hover mood-color shifts, and asset-gated preloaders — each with the snippet to lift and the reasoning behind the timing/easing choices.
- **Automatic stock asset sourcing** from Pexels and Unsplash (real downloads into an `assets/` folder, not hotlinks) when the user doesn't supply their own images or video.
- **An optional 3D/WebGL hero** — vanilla Three.js for the single-file path, React Three Fiber for the Next.js path.
- **A Next.js/React integration path** — every pattern above has a matching `useGSAP`-based component version for projects that want real components instead of a static file.
- **Zero build step by default.** The standard output is one self-contained `.html` file loading GSAP from a CDN — open it in a browser, drop it into any project, no npm install required.

## Installation

Parallux is a standard [Agent Skill](https://github.com/anthropics/skills) — a folder with a `SKILL.md` plus reference docs and templates. How you install it depends on where you use Claude:

**claude.ai**
1. Zip the `parallux/` folder (the zip must contain the folder itself at its root, not just the loose files inside it) — or grab a pre-built `parallux.skill` from this repo's [Releases](../../releases) if one's been published.
2. Settings → Features → Skills → Upload, and upload that zip.
3. Enable code execution in Settings → Capabilities if it isn't already on (Parallux downloads stock assets and writes files, which needs it).

**Claude Code / Claude Desktop**
```bash
# personal install — available in every project
git clone https://github.com/<your-username>/parallux.git ~/.claude/skills/parallux

# OR project-scoped — committed to one repo, shared via git
git clone https://github.com/<your-username>/parallux.git .claude/skills/parallux
```
Restart your session afterward — skills are loaded at startup.

**Other Agent Skills-compatible tools** (Cursor, Gemini CLI, etc.) — copy the `parallux/` folder into whatever skills directory that tool reads from; the `SKILL.md` format is the same everywhere.

Exact menu paths shift as Anthropic ships updates — if something above looks out of date, search Anthropic's current docs for "Agent Skills" and adjust accordingly.

## Usage

Once installed, just ask for what you want — Claude reads `SKILL.md` and pulls in the rest as needed:

> "Build me an animated portfolio site for a photography studio, dark and moody, with a marquee and a project list."

> "Make a scroll site for my SaaS landing page — hero video, then a row of flip cards for features."

> "I want a full agency site — preloader, nav, services, work section, the whole thing."

Parallux will scope the request (which template/patterns fit, what words it needs), source matching stock photography/video automatically if you haven't supplied your own, build the page, and hand back a working `.html` file plus its `assets/` folder.

## Project structure

```
parallux/
├── SKILL.md                        # entry point — when to trigger, how to scope, how to build
├── references/
│   ├── patterns.md                 # the core animation techniques, with snippets
│   ├── stock-assets.md             # sourcing & downloading from Pexels/Unsplash
│   ├── asset-guide.md              # what each template/pattern needs
│   ├── 3d-hero.md                  # optional 3D/WebGL hero (Three.js + React Three Fiber)
│   └── nextjs-integration.md       # React/useGSAP component versions of every pattern
└── templates/
    ├── hero-gallery.html           # hero video/image reveal + flanking gallery
    ├── split-flip-cards.html       # pinned row of cards that flip in 3D
    └── agency-scroll.html          # full multi-section site combining most patterns above
```

## How asset sourcing works

When you don't provide your own images or video, Parallux searches Pexels and Unsplash for matching photography, resolves the direct CDN URL, and downloads it locally into `assets/images/` and `assets/videos/` — full details in [`references/stock-assets.md`](./parallux/references/stock-assets.md). Both platforms license their libraries for free use, including commercial projects, with no attribution required, which is what makes this different from hotlinking an arbitrary image off the web. If your environment's network policy blocks those CDN domains, Parallux will say so directly rather than shipping a broken image reference.

## Roadmap / contributing

This skill grows by feeding it real-world reference codebases — open-source GSAP/Three.js showcase sites, awwwards-style portfolio templates, anything with a scroll effect worth extracting. See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for how to propose a new pattern or template.

Ideas not yet built in: WebGL shader transitions, audio-reactive motion, CMS/Astro integration, an eval suite for the skill itself.

## License

[MIT](./LICENSE) — use it, fork it, ship it.

## Acknowledgments

The pattern library was built and refined by studying techniques common to open-source GSAP/ScrollTrigger showcase projects and Awwwards-style portfolio codebases, then rewritten from scratch as documented, framework-agnostic snippets rather than copied verbatim. GSAP and its plugins (ScrollTrigger, SplitText, Observer) are made by [GreenSock](https://gsap.com) and have been free for commercial use since Webflow's acquisition of the company. Three.js and React Three Fiber power the optional 3D hero. Stock imagery, when auto-sourced, comes from [Pexels](https://www.pexels.com) and [Unsplash](https://unsplash.com) under their respective free-use licenses.
