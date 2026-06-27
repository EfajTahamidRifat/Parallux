# Contributing to Parallux

Parallux grows mainly by extracting good techniques from real reference codebases and turning them into documented, reusable patterns. That's the highest-leverage contribution by far.

## Adding a new pattern

1. Open an issue or PR with the reference codebase (or a link to one) the technique came from, and a one-line description of the effect.
2. Rewrite the technique as a framework-agnostic vanilla JS/CSS snippet — don't paste the original verbatim; Parallux's snippets need to be understood and recombined by Claude, not just reproduced, and originals are often tightly coupled to a specific framework/build setup.
3. Add it to `parallux/references/patterns.md`:
   - A row in the lookup table at the top (effect description → pattern name).
   - A section with the snippet, a short explanation of *why* it's built that way, and any gotchas (easing choices, reverse-direction behavior, mobile fallbacks).
4. If it's substantial enough to need its own framework variant, add the React/`useGSAP` version to `parallux/references/nextjs-integration.md` too.

## Adding or editing a template

Templates in `parallux/templates/` are complete, working, single-file sites — not snippets. If you add one:

- Keep it self-contained (inline `<style>`/`<script>`, GSAP loaded from a CDN, no build step).
- Centralize theme values (colors, radius, font) in a `:root` CSS block at the top.
- Reference image/video assets via `assets/images/...` / `assets/videos/...` to match the existing convention.
- Mark customization points with `<!-- CUSTOMIZE: ... -->` comments, the same way the existing templates do.
- Validate it before submitting: check the inline `<script>` with `node --check`, and confirm HTML tags/CSS braces balance. (A quick way: extract the `<script>` and `<style>` blocks and run them through `node --check` / a brace count — see the repo's commit history for the approach used on existing templates.)
- Add a matching entry to `parallux/references/asset-guide.md` (what it needs, and whether that's something to ask the user for or to auto-source) and to the reference map / pattern table in `SKILL.md`.

## Reporting a bad build

If Parallux produces something broken or visually off, the most useful report includes:
- The prompt you used.
- What template/pattern it picked.
- What went wrong (broken animation, wrong reverse-scroll behavior, asset sourcing failure, etc.) — a screenshot or screen recording helps a lot for anything visual.

## Code of conduct

Be direct, be kind, assume good faith. Disagreements about taste (easing curves, color choices) are normal in a project like this — argue from "here's why it reads better" rather than just preference.
