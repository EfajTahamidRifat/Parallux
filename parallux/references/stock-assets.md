# Sourcing stock photos &amp; video (Pexels, Unsplash)

**Default behavior:** when the user hasn't supplied their own images/video (no uploads, no "use my own assets" instruction), automatically source real, high-quality stock photography — and stock video where a pattern needs it — from Pexels and Unsplash, and download it into an `assets/` folder. Don't ask the user to go find/upload photos first, and don't fall back to flat color/gradient blocks — a site built around real (if generic) photography reads as dramatically more finished than one built around solid rectangles. Real, user-supplied brand assets still win whenever they're available; this is the default for everything else.

Both Pexels and Unsplash license their full libraries for free use, including commercial projects, with no attribution required (Pexels License / Unsplash License) — that's what makes this different from hotlinking a random image found on the web, which is still fragile and a real copyright risk. Downloading from these two specifically, and saving a local copy, is the legitimate, intended use case for both platforms.

## Workflow

1. **Turn the shot list into search phrases.** `references/asset-guide.md` gives the count/aspect-ratio per slot (hero video, gallery images, card images, project shots, etc.); combine each slot with the vibe/keywords already gathered in Step 1 — e.g. "moody architecture at night," "minimalist product studio shot," "warm editorial portrait." Specific, descriptive phrases beat generic ones ("coffee shop interior natural light" finds better results than "coffee").

2. **Search.** Use `web_search` scoped to one site at a time, e.g. `site:pexels.com moody architecture night` or `site:unsplash.com minimalist product studio`. For video, search `site:pexels.com <description> video` (Unsplash doesn't host video).

3. **Resolve the direct CDN URL.** `web_fetch` a couple of promising result pages and pull the real asset URL out of the page rather than the search-result thumbnail:
   - **Pexels photos** serve from `images.pexels.com/photos/<id>/pexels-photo-<id>.jpeg` — append `?auto=compress&cs=tinysrgb&w=1600` (or similar) to get a web-appropriate size instead of the multi-MB original.
   - **Pexels videos** serve from `videos.pexels.com/video-files/<id>/...mp4` — pick the resolution that matches where it'll be used (a full-bleed hero usually wants the largest available; a small background loop doesn't).
   - **Unsplash photos** serve from `images.unsplash.com/photo-<id>-<hash>` — append `?w=1600&q=80&fm=jpg&fit=crop` to control size/format instead of pulling the raw original.

4. **Download into the project's `assets/` folder**, not just linked to remotely:

   ```bash
   mkdir -p assets/images assets/videos
   curl -L -o assets/images/hero.jpg "https://images.pexels.com/photos/<id>/pexels-photo-<id>.jpeg?auto=compress&cs=tinysrgb&w=1600"
   curl -L -o assets/videos/hero-loop.mp4 "https://videos.pexels.com/video-files/<id>/<id>-hd.mp4"
   ```

   Reference them from the HTML the same way the templates already expect: `assets/images/hero.jpg`, `assets/videos/hero-loop.mp4`.

5. **If the download fails** (a `403`/network error rather than a normal HTTP error from the CDN itself), it's almost always the sandbox's network allowlist blocking `images.pexels.com` / `videos.pexels.com` / `images.unsplash.com`, not a problem with the URL. Say so plainly and tell the user they can add those domains in their network settings — don't silently ship a broken `<img src>` or invent a placeholder without saying what happened.

## Picking good ones, not just any ones

This is in service of "stunning," not just "populated" — a couple of extra seconds choosing matters:

- Prefer a shot whose **native orientation already matches the slot** (portrait for a tall card, wide for a hero) over a great shot that needs an awkward crop.
- Favor **editorial-feeling, well-composed, well-lit** images over generic/clip-art-y stock — both platforms have plenty of both; scan a few results rather than grabbing the first hit.
- Keep a **consistent visual mood** across all images on one page (same rough color temperature/lighting style) — a jarring mix of stock photo styles undercuts the premium feel faster than almost anything else.
- For hero video specifically, look for genuinely **loop-friendly** footage (steady camera, no hard cuts) — Pexels' video search results usually note duration, which is a quick signal.

## Crediting (optional, but don't fabricate it)

Neither license requires attribution. If the user wants a credit line anyway (some portfolio/agency sites include a small "Photos via Pexels/Unsplash" footer note as a nice touch), pull the actual photographer name and source link from the page you fetched — never guess or invent a name.

## When to skip this entirely

- The user has uploaded their own images/video, or clearly has brand assets in mind — use those instead, full stop.
- The user explicitly says they'll supply their own assets later, or asks for plain placeholder blocks — honor that and don't go fetch stock photos they didn't ask for.
