# Papilio — Wings in Motion

A single-file landing page for a 3D motion atelier, built with plain HTML/CSS/JS and [three.js](https://threejs.org) 0.160.0 (via import map, no build step).

## Run it locally

The page loads a local `.glb` model, so it has to be served over HTTP — opening the file from disk will block the model.

```bash
python -m http.server 8000
```

Then open <http://localhost:8000/newindex.html>.

## What is in here

| File | What it is |
| --- | --- |
| `newindex.html` | The page: markup, styles and the scroll-driven three.js scene in one file |
| `Assets/animated_flying_fluttering_butterfly_loop_v2.0.glb` | Rigged butterfly, with the `hover` clip driving the wings |
| `Assets/*.jpg`, `Assets/download.png` | Artwork used in the sections |
| `index.html` | The earlier Melli page |

## The butterfly

- Follows the scroll path from section to section, drifting in slow waves and pausing between moves.
- Lands on the artwork in each section (and on the **O** of PAPILIO), settling into a top-down wing view.
- Takes off again on its own, or immediately when you scroll.
- No cursor following, no external animation library.

## Deploy

It is a static site — any static host works (GitHub Pages, Netlify, Vercel, Cloudflare Pages). Just keep `Assets/` next to the HTML.
