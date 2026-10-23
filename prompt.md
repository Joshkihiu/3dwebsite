# BUILD PROMPT — "<BRAND>" scroll-driven landing page

Build a single, production-quality landing page from scratch. One file, no build step, no frameworks. The result must feel like a high-end design studio’s site: full-bleed artwork, huge editorial serif type, a warm ember accent on charcoal and bone, and a real 3D creature that flies, lands and takes off again as you scroll.

Reproduce the structure, layout values, copy shape, motion numbers and the 3D behaviour described below **exactly**. Only the brand, the copy and the image files change.

---

## 0. Fill these in first

| Placeholder | What to put |
| --- | --- |
| `<BRAND>` | Company name in one word, ALL CAPS in the hero (e.g. `PAPILIO`). **The last letter must be wrapped separately** — it becomes a landing perch. |
| `<TAGLINE>` | One line, sentence case, ends with a full stop. E.g. “Still images, given wings.” |
| `<KICKER>` | 2–6 words, lower case. E.g. “A motion atelier” |
| `<EST>` | Founding year, e.g. “Est. 2026” |
| `<CTA>` | 2–3 words, e.g. “Book a seat” |
| `<ACCENT>` | Accent colour. Default `#ff5a1f` (ember). |
| `<MODEL>` | URL of a rigged `.glb` with a looping hover/flap animation clip. Head should point along **+Z**. |
| `<IMG_HERO>` | Dark, cinematic background image (portrait or square, ≥1600px tall). |
| `<IMG_1> … <IMG_6>` | 6 portrait artworks. They are mixed across the light and dark sections. |
| `<SOCIAL_1/2>` | Two social links (Instagram, Dribbble, LinkedIn…). |

---

## 1. Non-negotiables

- **One file:** `index.html` containing all HTML, CSS and JS. No React, no Tailwind, no bundler, no external CSS/JS files.
- **three.js 0.160.0** loaded from a CDN through an import map:

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
  }
}
</script>
```

- Imports in the module script: `GLTFLoader` from `three/addons/loaders/GLTFLoader.js`, `RoomEnvironment` from `three/addons/environments/RoomEnvironment.js`.
- **No cursor-following.** The 3D object is driven by scroll, by its own wave drift, and by an autonomous land/take-off cycle.
- All decorative images use `alt=""`.
- The page must be served over HTTP (the `.glb` is fetched) — `python -m http.server 8000`.

---

## 2. Design tokens (copy these values verbatim)

```css
:root {
  --ink: #151013;
  --bone: #f2ede2;
  --ash: #ddd5c6;
  --muted: #6d655c;
  --ember: #ff5a1f;        /* = <ACCENT> */
  --void: #0b0a0d;
  --pad: clamp(18px, 3.4vw, 56px);
  --serif: "Instrument Serif", Georgia, "Times New Roman", serif;
  --sans: "Poppins", system-ui, -apple-system, sans-serif;
  --mono: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
}
```

Fonts (Google Fonts, preconnect to `fonts.googleapis.com` and `fonts.gstatic.com` with `crossorigin`):

```
https://fonts.googleapis.com/css2?family=Instrument+Serif:ital@0;1&family=Poppins:wght@300;400;500;600&display=swap
```

Base:

```css
* { box-sizing: border-box; margin: 0; padding: 0; }
html { scroll-behavior: auto; }
body { font-family: var(--sans); color: var(--ink); background: var(--bone); overflow-x: hidden; -webkit-font-smoothing: antialiased; }
img { display: block; }
a { color: inherit; text-decoration: none; }
::selection { background: var(--ember); color: #fff; }
```

**Paper grain** — a fixed overlay on top of everything except nav/loader:

```css
body::after {
  content: ""; position: fixed; inset: 0; z-index: 25; pointer-events: none;
  opacity: .045; mix-blend-mode: multiply;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='3'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)'/%3E%3C/svg%3E");
}
```

---

## 3. Page skeleton, in order

```
loader (fixed)
nav (fixed)
canvas#bee (fixed, z-20)
main
  section.hero        #top      data-theme="dark"
  section.s1          #craft    data-theme="dark"
  section.s2          #colour   data-theme="light"
  section.s3          #motion   data-theme="dark"
  section.s4          #form     data-theme="light"
  section.s5          #pattern  data-theme="dark"
  section.s6          #atelier  data-theme="light"
  section.s7          #fracture data-theme="dark"
  section.s8          #layers   data-theme="light"
  section.s9          #ember    data-theme="dark"
footer.foot           #contact
```

Nine content sections between hero and footer. Sections alternate dark / light so the page breathes: **dark, light, dark, light, dark, light, dark, light, dark**. Two of the dark sections are adjacent to the hero’s dark — that is intentional.

---

## 4. Fixed chrome

### Nav

```css
.nav { position: fixed; top: 0; left: 0; right: 0; z-index: 30; display: flex; align-items: center;
  padding: 20px var(--pad) 0; pointer-events: none; color: var(--ink); transition: color .5s ease; }
.nav > * { pointer-events: auto; }
.nav, .badge { transition: color .5s ease; }
body.on-dark .nav, body.on-dark .badge { color: #f6efe3; }
.logo { width: 40px; height: 40px; display: grid; place-items: center; margin-right: clamp(26px, 5vw, 72px); }
.logo svg { width: 34px; height: 34px; transition: transform .7s cubic-bezier(.2,.8,.2,1); }
.logo:hover svg { transform: rotate(180deg); }
.nav__links { display: flex; gap: clamp(20px, 3.4vw, 44px); font-size: 11.5px; font-weight: 500;
  letter-spacing: .14em; text-transform: uppercase; }
.nav__links a { position: relative; }
.nav__links a::after { content: ""; position: absolute; left: 0; right: 0; bottom: -5px; height: 1px;
  background: currentColor; transform: scaleX(0); transform-origin: left;
  transition: transform .4s cubic-bezier(.2,.8,.2,1); }
.nav__links a:hover::after { transform: scaleX(1); }
.nav__cta { margin-left: auto; display: inline-flex; align-items: center; font-size: 11px; font-weight: 600;
  letter-spacing: .14em; text-transform: uppercase; border: 1px solid currentColor; border-radius: 999px;
  padding: 11px 18px; transition: background .4s ease, color .4s ease, border-color .4s ease; }
.nav__cta:hover { background: var(--ember); border-color: var(--ember); color: #fff; }
```

Markup: logo link → `#top` with an inline SVG mark; two links (“The craft” → `#craft`, “Motion” → `#motion`); CTA `<CTA>` → `#contact`.

### Canvas

```css
#bee { position: fixed; inset: 0; z-index: 20; width: 100%; height: 100%; pointer-events: none;
  opacity: 0; transition: opacity 1.4s ease; }
#bee.ready { opacity: 1; }
```

---

## 5. Hero (dark, full-bleed)

```html
<section class="hero" id="top" data-theme="dark" data-fly="0.30,0.08,1,150,4,-4">
  <div class="hero__bg" data-speed="0.14"><img alt="" src="<IMG_HERO>"></div>
  <div class="hero__veil"></div>
  <div class="hero__inner">
    <span class="hero__kicker"><KICKER></span>
    <h1 class="hero__word"><span class="line"><span>PAPILI<span class="o" data-perch="0.5,0.18">O</span></span></span></h1>
    <p class="hero__tag"><TAGLINE></p>
    <div class="hero__row">
      <a class="btn" href="#craft">Enter <svg …chevron…/></a>
      <span class="hero__meta"><EST></span>
    </div>
  </div>
  <div class="hero__hint"><span>scroll</span><i></i></div>
</section>
```

```css
.hero { background: #0a0908; color: #f6efe3; }
.hero__bg { position: absolute; inset: -8% 0; z-index: 0; will-change: transform; }
.hero__bg img { width: 100%; height: 100%; object-fit: cover; object-position: 50% 44%; }
.hero__veil { position: absolute; inset: 0; z-index: 1; pointer-events: none;
  background:
    linear-gradient(180deg, rgba(6,6,8,.6) 0%, rgba(6,6,8,.18) 42%, rgba(6,6,8,.86) 100%),
    radial-gradient(62% 58% at 14% 18%, rgba(255,90,31,.2), rgba(255,90,31,0) 70%); }
.hero__inner { position: relative; z-index: 3; min-height: 100vh; display: flex; flex-direction: column;
  align-items: center; justify-content: center; text-align: center; padding: 0 var(--pad); }
.hero__word { font-family: var(--serif); font-weight: 400; font-size: clamp(70px, 17.5vw, 310px);
  line-height: .8; letter-spacing: -.045em; text-shadow: 0 30px 90px rgba(0,0,0,.45); }
.hero__word .line { display: block; overflow: hidden; padding-bottom: .02em; }
.hero__word .line > span { display: inline-block; transform: translateY(108%); }
.hero__word .o { display: inline-block; }
.loaded .hero__word .line > span { animation: rise 1.5s cubic-bezier(.16,.84,.2,1) forwards; }
@keyframes rise { to { transform: translateY(0); } }
```

- `hero__kicker`: mono, 9px, `.32em`, uppercase, 36px rule in `::before`, `rgba(246,239,227,.72)`.
- `hero__tag`: serif italic, `clamp(17px, 1.8vw, 28px)`, `rgba(246,239,227,.74)`, `margin-top: 26px`.
- `hero__row`: `margin-top: 40px`, centred flex, `gap: 24px`.
- `.btn` (shared): `#f6efe3` bg, ink text, 10.5px, 600, `.16em`, uppercase, `padding 16px 24px`, `border-radius 999px`; hover → ember bg, white text, `translateY(-2px)`.
- Entrance: kicker fades at .2s, tag at .6s, row at .8s, hint at 1.1s (all `@keyframes fade` from `opacity 0 / translateY(12px)`).
- `hero__hint`: bottom 26px, mono 8.5px `.32em`, with a 1px vertical gradient line that pulses (`drop` 2.4s).

**The `O` perch:** the last letter of `<BRAND>` sits in `<span class="o" data-perch="0.5,0.18">` so the 3D creature can land on it.

---

## 6. The nine sections — layout grid

Shared pieces:

```css
section { position: relative; min-height: 100vh; height: 100vh; overflow: hidden; }
.poster { position: absolute; overflow: hidden; background: #171310; outline: 1px solid rgba(0,0,0,.08);
  box-shadow: 0 50px 80px -40px rgba(25,12,5,.6); will-change: transform; }
.poster img { width: 100%; height: 100%; object-fit: cover; }
.ghost { position: absolute; font-family: var(--serif); font-style: italic;
  font-size: clamp(150px, 24vw, 400px); line-height: .7; color: rgba(242,237,226,.05);
  letter-spacing: -.04em; pointer-events: none; user-select: none; }
```

Text block:

```css
.eyebrow { display: block; font-size: 10px; font-weight: 500; letter-spacing: .26em;
  text-transform: uppercase; color: var(--ember); margin-bottom: 24px; }
.title { font-family: var(--serif); font-weight: 400; font-size: clamp(34px, 4.4vw, 72px);
  line-height: .98; letter-spacing: -.02em; }
.title em { font-style: italic; color: var(--ember); }
.copy { margin-top: 26px; font-size: clamp(13px, 1.02vw, 16px); line-height: 1.72;
  color: var(--muted); max-width: 400px; }
```

Title pattern: **two short lines, the last word in `<em>`** (italic, ember). Copy pattern: **one sentence, max 22 words, no marketing clichés**.

| # | id | Theme | Eyebrow | Title | Copy |
| --- | --- | --- | --- | --- | --- |
| 01 | `craft` | dark | The craft | Cut from paper,<br>moved by `<em>light</em>` | Layered depth, torn edges and warm shadows. We build tactile worlds from a flat sheet — and never lose the handmade feel. |
| 02 | `colour` | light | Colour | Ink, bloom<br>and `<em>balance</em>` | A single red note against black and white can carry a whole frame. We grade until it lands with weight. |
| 03 | `motion` | dark | Motion | Ash, ember<br>and `<em>flight</em>` | Rig a pair of wings, breathe life into them with curves and loops, then let them fly anywhere. |
| 04 | `form` | light | Form | Shape,<br>then `<em>shadow</em>` | Open frames floating above the page. Geometry that keeps its calm while everything around it moves. |
| 05 | `pattern` | dark | Pattern | Repetition,<br>then `<em>quiet</em>` | Two leaves, two weights of ink. The eye fills in the rest without being asked. |
| 06 | `atelier` | light | Surface | Every<br>`<em>surface</em>` | Layer upon layer until the flat page starts to lean toward you. |
| 07 | `fracture` | dark | Fracture | Break it<br>to `<em>fit</em>` | A thousand pieces, one picture. The seams are the point. |
| 08 | `layers` | light | Layers | Three quiet<br>`<em>notes</em>` | *(no copy — heading only)* |
| 09 | `ember` | dark | Ember | Everything<br>`<em>rises</em>` | *(no copy — centred over a full-bleed image)* |

Rewrite the words for `<BRAND>`, keep the rhythm: 2–4 words per line, one italic accent word per title.

### Per-section composition (exact values)

**01 — flag layout (dark):** poster left / text right.
```css
.s1 { background: var(--void); color: var(--bone); z-index: 3; }
.s1::before { content: ""; position: absolute; inset: 0; pointer-events: none;
  background: radial-gradient(70% 60% at 16% 100%, rgba(255,90,31,.22), rgba(255,90,31,0) 66%),
              radial-gradient(50% 45% at 96% 0%, rgba(122,84,255,.14), rgba(122,84,255,0) 70%); }
.s1 .poster { left: clamp(18px, 6vw, 120px); top: 14vh; width: clamp(240px, 27vw, 400px); height: 66vh; z-index: 1; }
.s1__text { position: absolute; left: 56vw; top: 33vh; width: min(470px, 36vw); z-index: 2; }
.s1 .ghost { right: 3vw; bottom: -3vh; }
.s1 .copy { color: rgba(242,237,226,.62); }
```

**02 — circular disc (light):** text left / disc right, disc in a hairline ring.
```css
.s2 { background: linear-gradient(180deg, #f2ede2 0%, #efe6d6 100%); z-index: 2; }
.s2__text { position: absolute; left: clamp(18px, 6vw, 120px); top: 30vh; width: min(450px, 38vw); z-index: 2; }
.s2__art { position: absolute; right: clamp(18px, 6vw, 110px); top: 18vh; width: clamp(260px, 30vw, 470px); z-index: 1; will-change: transform; }
.s2__art::before { content: ""; position: absolute; inset: -26px; border: 1px solid rgba(21,16,19,.16); border-radius: 50%; }
.s2__disc { position: relative; width: 100%; aspect-ratio: 1 / 1; border-radius: 50%; overflow: hidden;
  box-shadow: 0 60px 90px -45px rgba(60,30,10,.5); }
```

**03 — tall poster bleeding off-screen (dark):** text left / poster right, `right: -3vw`.
```css
.s3__text { left: clamp(18px, 6vw, 120px); top: 28vh; width: min(460px, 38vw); }
.s3 .poster { right: -3vw; top: 8vh; width: clamp(250px, 28vw, 420px); height: 76vh; }
.s3 .ghost { left: 30vw; bottom: -6vh; }
```

**04 — flag with ghost number top-right (light):**
```css
.s4 { background: linear-gradient(180deg, #f4efe5 0%, #ece3d3 100%); }
.s4__text { left: clamp(18px, 6vw, 120px); top: 32vh; width: min(440px, 36vw); }
.s4 .poster { right: clamp(18px, 6vw, 118px); top: 16vh; width: clamp(230px, 26vw, 380px); height: 64vh; }
.s4 .ghost { right: 2vw; top: 2vh; color: rgba(21,16,19,.05); }
```

**05 — diptych (dark):** two overlapping posters on the left, text right at `58vw`.
```css
.s5__text { left: 58vw; top: 32vh; width: min(440px, 34vw); z-index: 3; }
.s5 .poster--a { left: clamp(16px, 5vw, 88px); top: 15vh; width: clamp(160px, 18vw, 260px); height: 50vh; z-index: 1; }
.s5 .poster--b { left: calc(clamp(16px, 5vw, 88px) + clamp(120px, 15vw, 220px)); top: 34vh;
  width: clamp(150px, 17vw, 248px); height: 44vh; z-index: 2; }
```

**06 — square poster (light):**
```css
.s6 { background: linear-gradient(180deg, #ece3d3 0%, #f2ede2 100%); }
.s6__text { left: clamp(18px, 6vw, 120px); top: 28vh; width: min(440px, 36vw); }
.s6 .poster { right: clamp(18px, 7vw, 130px); top: 20vh; width: clamp(240px, 30vw, 430px); height: clamp(240px, 30vw, 430px); }
.s6 .ghost { left: 46vw; bottom: -4vh; color: rgba(21,16,19,.05); }
```

**07 — tall poster (dark), no ghost number:**
```css
.s7__text { left: clamp(18px, 6vw, 120px); top: 30vh; width: min(450px, 38vw); }
.s7 .poster { right: clamp(18px, 6vw, 118px); top: 12vh; width: clamp(240px, 26vw, 390px); height: 70vh; }
```

**08 — three-panel band (light):** heading centred at the top, three posters filling the bottom.
```css
.s8 { background: linear-gradient(180deg, #f4efe5 0%, #ece3d3 100%); }
.s8__head { position: absolute; left: 0; right: 0; top: 13vh; text-align: center; z-index: 3; }
.s8__head .title { font-size: clamp(34px, 4.6vw, 78px); }
.s8__band { position: absolute; left: 0; right: 0; bottom: 0; height: 58vh; display: flex; }
.s8__band .poster { position: relative; flex: 1; height: 100%; width: auto; top: auto; left: auto; right: auto; object-fit: cover; }
.s8__band .poster:nth-child(2) { margin-left: -4.5%; z-index: 2; }   /* panels overlap, no gaps */
.s8__band .poster:nth-child(3) { margin-left: -4.5%; z-index: 3; }
```

**09 — full-bleed band (dark):** image behind, centred heading over it.
```css
.s9 .bleed { position: absolute; inset: -7% 0; z-index: 0; will-change: transform; }
.s9 .bleed img { width: 100%; height: 100%; object-fit: cover; object-position: 50% 40%; }
.s9::after { content: ""; position: absolute; inset: 0; z-index: 1; pointer-events: none;
  background: linear-gradient(180deg, rgba(8,7,10,.78) 0%, rgba(8,7,10,.3) 48%, rgba(8,7,10,.92) 100%); }
.s9__inner { position: absolute; inset: 0; z-index: 2; display: flex; flex-direction: column;
  align-items: center; justify-content: center; text-align: center; padding: 0 var(--pad); }
.s9__inner .title { font-size: clamp(42px, 6.2vw, 116px); }
```

Each section’s text block gets `class="reveal"`, `reveal d1`, `reveal d2` on eyebrow, title and copy respectively.

---

## 7. Small footer

```css
.foot { position: relative; background: var(--bone); padding: 70px var(--pad) 54px; }
.foot::before { content: ""; position: absolute; top: 0; left: var(--pad); right: var(--pad);
  height: 1px; background: rgba(21,16,19,.14); }
.foot__row { display: flex; align-items: flex-end; justify-content: space-between; gap: 34px; flex-wrap: wrap; }
.foot strong { display: block; font-family: var(--serif); font-weight: 400; font-size: clamp(24px, 2.4vw, 36px); }
.foot .tag { font-size: 9.5px; letter-spacing: .24em; text-transform: uppercase; color: var(--muted); }
.foot .cols { display: flex; gap: clamp(26px, 4vw, 60px); font-size: 12.5px; }
.foot .cols a { display: block; line-height: 2; color: var(--muted); transition: color .3s ease; }
.foot .cols a:hover { color: var(--ember); }
.foot .made { margin-top: 46px; font-family: var(--mono); font-size: 8.5px; letter-spacing: .26em;
  text-transform: uppercase; color: rgba(21,16,19,.35); }
```

Markup: brand `<strong>` + tagline span on the left; three link columns (section links + `<SOCIAL_1>` / `<SOCIAL_2>`) on the right; `Built by <BRAND> · <YEAR>` in `.made`.

---

## 8. Motion system

**1 · Reveal on enter.** `IntersectionObserver` (`threshold: .25`) on every `main > section`; add class `in` when intersecting and **never remove it**.

```css
.reveal { opacity: 0; transform: translateY(38px);
  transition: opacity 1s ease, transform 1.1s cubic-bezier(.2,.8,.15,1); }
.in .reveal { opacity: 1; transform: none; }
.in .reveal.d1 { transition-delay: .1s; }
.in .reveal.d2 { transition-delay: .22s; }
```

**2 · Parallax.** Every `[data-speed]` element:

```js
const sec = el.closest('section');
const p = -sec.getBoundingClientRect().top / innerHeight;
el.style.transform = `translate3d(0, ${p * speed * innerHeight}px, 0) rotate(${p * rot}deg)`;
```

Speeds used: hero bg `0.14`; poster 01 `0.3 / -6°`; disc 02 `-0.24 / 10°`; poster 03 `-0.2 / 7°`; 04 `-0.26 / 8°`; 05 `0.22 / -7°` and `0.42 / 6°`; 06 `-0.22 / 9°`; 07 `0.26 / -7°`; band 08 `0.14/-4°`, `0.22/3°`, `0.18/-3°`; band 09 `0.1`.

**3 · Nav theme.** The section whose rect spans `innerHeight * .06` decides the chrome colour: `body.classList.toggle('on-dark', section.dataset.theme === 'dark')`.

**4 · Keep it in sync.** Run the parallax + theme pass from a passive scroll listener **and** from the rAF loop whenever `scrollY` changed since the last frame (some browsers coalesce scroll events).

---

## 9. The 3D creature

### Scene

```js
renderer: antialias, alpha:true, pixelRatio = min(devicePixelRatio, 2),
          outputColorSpace = SRGBColorSpace, toneMapping = ACESFilmicToneMapping, exposure 1.2
scene.environment = new PMREMGenerator(renderer).fromScene(new RoomEnvironment(), .04).texture
camera = PerspectiveCamera(32, aspect, .1, 100) at (0, 0, 10)
lights: HemisphereLight(0xfff4e6, 0x2b1b3a, 1.2)
        DirectionalLight(0xffd9b8, 3)   at (4, 6, 6)     ← warm key
        DirectionalLight(0x9fc0ff, 1.7) at (-6, 1, -5)   ← cool rim
```

Hierarchy: `scene → flight (position, scale, yaw) → root (rotation order "YZX") → holder → model`.

### Load + normalise

1. `model.rotation.set(0, Math.PI / 2, 0)` so the head points along **+X**.
2. Measure `Box3` of a wrapper, scale the model by `1 / max(size.x, size.y, size.z)`.
3. Re-measure, subtract the box centre from `model.position` so the creature sits at the origin, 1 unit wide.
4. Every mesh: `frustumCulled = false`, `material.envMapIntensity = .85`.
5. Animation: `AnimationMixer`, clip `hover` (fallback to the first clip), loop forever.

### Scroll path

Each section carries its anchor in `data-fly="x,y,scale,yaw,pitch,roll"` (x/y are −1…1 in viewport units; yaw 0 = facing right, 90 = facing camera, 180 = facing left):

| Section | x | y | scale | yaw | pitch | roll |
| --- | --- | --- | --- | --- | --- | --- |
| hero | 0.30 | 0.08 | 1.00 | 150 | 4 | −4 |
| 01 | −0.02 | 0.28 | 0.94 | 70 | 8 | 4 |
| 02 | 0.03 | 0.26 | 0.90 | 210 | −6 | −5 |
| 03 | −0.03 | 0.26 | 0.90 | 150 | 6 | 2 |
| 04 | 0.05 | 0.24 | 0.90 | 62 | 8 | 5 |
| 05 | −0.05 | 0.22 | 0.86 | 205 | −6 | −6 |
| 06 | 0.04 | 0.24 | 0.88 | 148 | 6 | 3 |
| 07 | 0.03 | 0.24 | 0.90 | 68 | 8 | 4 |
| 08 | −0.03 | 0.20 | 0.88 | 200 | −6 | −5 |
| 09 | 0.02 | 0.22 | 0.88 | 150 | 6 | 2 |

Blend between anchors using the viewport centre:

```js
const c = scrollY + innerHeight * .5;                  // viewport centre in page space
// find the two sections either side of c (by their page centres)
const t = ease(clamp((raw - .16) / .64, 0, 1));        // hold at each section, fly in between
const dive = Math.sin(t * Math.PI) * -.12;             // dips mid-transition
```

### Idle behaviour (no cursor involved)

The creature never sits still while cruising — it rides slow **waves** and picks new **roam** targets:

```js
const env = clamp(Math.sin(t * 0.16 + phase) * 1.55, 0, 1);   // 0 = chilling, 1 = full drift
const wx  = Math.sin(t * 0.6  + phase)         * 0.17 * env;  // drifts left, then right
const wy  = Math.sin(t * 0.43 + phase * 1.7)   * 0.085 * env;
// roam: every 2.8–6.4s pick a goal of ±0.14 x / ±0.11 y and ease toward it (k ≈ 0.8/s)
x = anchor.x + wx + roam.x;  y = anchor.y + wy + roam.y;
```

### Landing (autonomous)

- Every **3.2–6.6s** while cruising, pick a random visible `[data-perch]` element in the current section (skip one another creature is using).
- **approach** (1.5s, eased) → **perched** (4.2s) → **takeoff** (1.3s) → cruise again.
- A scroll jump of **more than 14px** that frame forces an immediate take-off.
- While perched the pose is a **top-down wing view**: `root.rotation.x = 86°` (roll), `yaw = 24°`, `pitch = 0`, size `0.78`, wing animation slowed to `0.35` speed, and the hover bob damped to 20%.
- Perch position on an element: `left + width * fx`, `top + height * fy`, where `fx/fy` come from `data-perch="fx,fy"` (default `0.68,0.12`; the hero letter uses `0.5,0.18`).

### Applying the transform each frame

```js
const halfH = Math.tan((camera.fov * DEG) / 2) * camera.position.z;
const halfW = halfH * camera.aspect;

flight.position.set(cur.x * halfW, cur.y * halfH, Math.sin(t * 0.9) * 0.15 + introZ);
flight.scale.setScalar(halfH * 2 * 0.30 * cur.s * introScale);   // ≈ 30% of viewport height
flight.rotation.set(0, -cur.yaw * DEG, 0);

// pitch/roll react to motion
const pitch = (cur.pitch - clamp(vy * 20, -24, 24) + Math.sin(t * 2.1 + 1) * 3 * calm) * DEG;
const roll  = (cur.roll  + clamp(vx * 16, -24, 24) + Math.sin(t * 1.7) * 3.5 * calm) * DEG;
root.rotation.set(roll, 0, pitch, 'YZX');
```

- Smoothing: `cur += (target - cur) * (1 - Math.exp(-dt * 3.2))`; yaw uses `1 - Math.exp(-dt * 2.6)` with shortest-arc interpolation.
- Wing speed: cruise `1.05 + moving * 0.5`, perched `0.35`, approach/take-off `1.5`; ease toward the target with `1 - Math.exp(-dt * 4)`.
- **Intro:** the creature starts at `z = -8` and `scale × 0.22`, easing in over **2.8s** on load.
- `dt` is clamped to `0.05`.

---

## 10. Loader

```html
<div class="loader" id="loader">
  <svg viewBox="0 0 40 40" fill="none">
    <circle cx="20" cy="20" r="16" stroke="rgba(21,16,19,.14)" stroke-width="2"/>
    <path d="M20 4a16 16 0 0 1 16 16" stroke="#ff5a1f" stroke-width="2" stroke-linecap="round"/>
  </svg>
</div>
```

Full-screen bone overlay, z-50, spinner rotates 1.1s linear. It fades out (`.done`) either when the GLB finishes loading **or** after a **2.2s** timeout. Both paths also add `body.loaded`, which is what triggers the hero’s rise/fade entrance. On success the canvas also gets `.ready` (opacity 0 → 1 over 1.4s).

---

## 11. Responsive (≤ 760px) — the same layout, scaled down

**Do not re-flow the sections.** The phone layout is a 1:1 replica of the desktop: same 100vh stage, same left/right arrangement, same absolute positions — the artwork and the type just get smaller.

```css
:root { --pad: 16px; }
section { height: 100svh; min-height: 100svh; }   /* keep the one-screen-per-section stage */
body::after { display: none; }                     /* grain off on phones (perf) */

.nav__links { display: none; }
.nav__cta { font-size: 8px; padding: 8px 12px; }
.hero__word { font-size: clamp(56px, 19vw, 110px); }
.hero__tag { font-size: 12.5px; }
.hero__hint { display: none; }

.eyebrow { font-size: 7.5px; margin-bottom: 10px; }
.title { font-size: clamp(19px, 5.8vw, 27px); }
.copy { font-size: 10.5px; line-height: 1.52; margin-top: 10px; max-width: none; }
.ghost { font-size: 46vw; }

/* each section becomes ONE flex row: artwork keeps its own proportions, height follows the content */
section { height: auto; min-height: 0; }
.s1, .s2, .s3, .s4, .s5, .s6, .s7 { display: flex; align-items: center; gap: 5vw; padding: 13vh var(--pad) 9vh; }
.s1__text, .s2__text, .s3__text, .s4__text, .s5__text, .s6__text, .s7__text {
  position: static; inset: auto; flex: 1 1 auto; width: auto; max-width: none;
}
.s1 .poster, .s2__art, .s3 .poster, .s4 .poster,
.s5 .poster--a, .s5 .poster--b, .s6 .poster, .s7 .poster {
  position: static; inset: auto; flex: 0 0 auto;
  width: 40vw; height: auto; aspect-ratio: 2 / 3; object-fit: cover;   /* never stretched */
}
.s2__art { aspect-ratio: 1 / 1; }
.s1 .poster { order: 0; } .s1__text { order: 1; }
.s2__text { order: 0; } .s2__art { order: 1; }
.s3__text, .s4__text, .s6__text, .s7__text { order: 0; }
.s3 .poster, .s4 .poster, .s6 .poster, .s7 .poster { order: 1; }
.s5 .poster--a { order: 0; width: 26vw; }
.s5 .poster--b { order: 1; width: 24vw; margin-left: -6vw; }
.s5__text { order: 2; }
.s8 { display: block; padding: 13vh 0 0; }
.s8__head { position: static; inset: auto; padding: 0 var(--pad); margin-bottom: 5vh; text-align: left; }
.s8__band { position: relative; inset: auto; height: 46vh; }
.s9__inner .title { font-size: clamp(28px, 8.4vw, 48px); }
```

Rules of thumb for the shrink: artwork ≈ 40vw wide with `aspect-ratio: 2/3` (or `1/1` for the disc), titles 19–27px, copy ≈ 10.5px, and every block keeps the desktop side of the page (poster left → text right, and so on). Because the section height follows its content, there is **no empty strip left under the artwork** — the next section starts a comfortable 9vh below it. Never force a fixed `height` on the artwork; that is what stretches it.

**The creature on phones:** scale it to **0.19** of the viewport height (desktop is 0.30) and offset its flight anchors by `+0.16` in x and `+0.30` in y so it hovers above-right of the copy instead of over it.

**Smoothness on phones:**
- `renderer.setPixelRatio(min(dpr, 1.5))` and `antialias: false` — desktop keeps `2` / `true`.
- Drop the grain overlay (`body::after`) on mobile.
- Cache the parallax elements at start-up and skip any section more than ~1.6 viewports away, so the scroll handler only writes a handful of transforms per frame.
- Multiply the parallax amplitude by **0.2** on phones (desktop stays 1) so the artwork drifts gently instead of sliding out of frame.
- Same for `resize`: re-apply the pixel ratio.

On mobile the creature simply keeps its anchors — they are already in viewport units.

---

## 12. Performance & accessibility

- `pixelRatio` capped at 2; `will-change: transform` only on the parallax elements.
- Everything decorative is `alt=""`; nav links have real labels; the logo link has `aria-label`.
- Respect `prefers-reduced-motion` if you add to it: disable the parallax/waves but keep the layout.
- The canvas is `pointer-events: none` and `aria-hidden` — it must never block clicks.

---

## 13. Definition of done

1. Opening the page shows a dark, full-bleed hero with `<BRAND>` in one huge serif line, the tagline, a pill CTA and a scroll cue; the 3D creature flies in from far away.
2. Scrolling moves the creature to a new spot in every section, with a mid-flight dip, and it occasionally lands and takes a top-down rest on the artwork (and on the last letter of `<BRAND>`).
3. Each section’s text rises in once, and never re-animates.
4. Posters parallax at different speeds and tilt a few degrees as you pass.
5. The nav flips to bone-white over dark sections and back to ink over light ones.
6. The page ends with the small footer (brand + links + credit), no “full-height” CTA section.
7. No console errors, no horizontal scrollbar, and it still reads well at 390px and 1440px wide.
8. Only one file changed: `index.html` (plus the images/model it references).
