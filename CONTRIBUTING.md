# Contributing to Parallux

Parallux grows mainly by extracting good techniques from real reference codebases and turning them into documented, reusable patterns. That's the highest-leverage contribution by far.

## Adding a new pattern

1. Open an issue or PR with the reference codebase (or a link to one) the technique came from, and a one-line description of the effect.
2. Rewrite the technique as a framework-agnostic vanilla JS/CSS snippet — don't paste the original verbatim; Parallux's snippets need to be understood and recombined by Claude, not just reproduced, and originals are often tightly coupled to a specific framework/build setup.
3. Add it to `parallux/references/patterns.md`:
   - A row in the lookup table at the top (effect description → pattern name).
   - A section with the snippet, a short explanation of *why* it's built that way, and any gotchas (easing choices, reverse-direction behavior, mobile fallbacks).
4. If it's substantial enough to need its own framework variant, add the React/`useGSAP` version to `parallux/references/nextjs-integration.md` too.
5. Update `SKILL.md` if the new pattern warrants a mention in the "Foundation" or technique list sections.

## Feeding it reference codebases

The fastest way to improve Parallux is to point it at Awwwards-winning or otherwise high-quality open-source codebases. The process:

1. Upload (or link) the codebase alongside the Parallux skill files in a Claude session.
2. Ask Claude to study the codebase, extract the core animation/interaction techniques, and add them to `patterns.md` as documented snippets — rewritten to be framework-agnostic rather than copied verbatim.
3. Review the extracted patterns, clean up any framework-specific artifacts, and submit as a PR.

Codebases already studied and incorporated: musabhassan.com, Truus.co, Kintaro, portfolio-main, webgl-portfolio.

Good candidates to add next: sites featuring page transition animations, audio-reactive motion, scroll-jacked horizontal sections, or WebGL shader transitions not yet covered.

## Adding or editing a template

Templates in `parallux/templates/` are complete, working, single-file sites — not snippets. If you add one:

- Keep it self-contained (inline `<style>`/`<script>`, GSAP + Lenis loaded from CDN, no build step).
- Centralize theme values (colors, radius, font) in a `:root` CSS block at the top.
- Reference image/video assets via `assets/images/...` / `assets/videos/...`.
- Mark customization points with `<!-- CUSTOMIZE: ... -->` comments.
- Validate before submitting: extract the `<script>` block and run `node --check` on it; confirm HTML tags/CSS braces balance.
- Add a matching entry to `parallux/references/asset-guide.md` and to the reference map in `SKILL.md`.

## Reporting a bad build

If Parallux produces something broken or visually off, the most useful report includes:
- The prompt you used.
- What template/pattern it picked.
- What went wrong (broken animation, wrong reverse-scroll behavior, asset sourcing failure, etc.) — a screenshot or screen recording helps a lot for anything visual.

## Code of conduct

Be direct, be kind, assume good faith. Disagreements about taste (easing curves, color choices) are normal — argue from "here's why it reads better" rather than just preference.
