# Core animation patterns — enhanced with Awwwards-winning techniques

Everything in `templates/` is built from recombinable techniques. This file covers both the original patterns and new ones extracted from 6 Awwwards-winning/nominated codebases: musabhassan.com (WebGL distortion), Truus.co (elastic cursor bubble, Lenis smooth scroll), Kintaro (interactive particles, SVG progress ring preloader, blur-reveal), and portfolio-main (curved SVG loader wipe, magnetic buttons).

Load order — always include these before your script block:

```html
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.14/dist/lenis.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/ScrollTrigger.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/SplitText.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/Observer.min.js"></script>
<script>gsap.registerPlugin(ScrollTrigger, SplitText, Observer);</script>
```

| Effect the user wants | Pattern |
|---|---|
| Buttery smooth scroll that ties to ScrollTrigger | Lenis smooth scroll |
| Animated dot/ring cursor that reacts to hover | Custom cursor with elastic follower |
| Cursor that shows context label ("click", "drag") on hover | Elastic cursor bubble (Truus.co) |
| Pin section while animations play as you scroll | Pinned scrub timeline |
| Headline text slides/fades in word-by-word or line-by-line | Split-text mask reveal |
| Each letter slides in with staggered mask, fancy easing | Per-letter slide-in (musabhassan.com) |
| Text resolves from random/glitchy characters | Text scramble reveal |
| Elements fade+blur into view on scroll | Blur-reveal on scroll (Kintaro) |
| Images warp/distort as you scroll with WebGL noise | WebGL noise-distortion image (musabhassan.com) |
| Particle field floating in background, scatters from mouse | Interactive particle field (Kintaro) |
| Curved/wave overlay that slides away on load | SVG curved loader wipe (portfolio-main) |
| Progress ring spinner with logo center | SVG progress ring preloader (Kintaro) |
| Loading screen with real asset progress gating | Asset-gated preloader |
| Cards/panels flip over in 3D on scroll | 3D card flip |
| Hero video/image shrinks and reshapes into frame | Clip-path mask morph |
| Nav that goes glassy/frosted on scroll | Header blur-on-scroll |
| Elements flying in from sides as section enters | Staggered fly-in |
| Infinite logo/word belt, speeds up with scroll | Velocity marquee |
| Floating image follows cursor over project list | Magnetic cursor preview |
| Button pulls magnetically toward cursor | Magnetic button pull |
| Full-screen menu slides in with staggered links | Slide-in nav overlay |
| Cards stack on top of each other while scrolling | Sticky stacking cards |
| Big text rows glide sideways at different speeds | Horizontal text drift |
| Background retints depending on what's hovered | Hover mood-color shift |
| Rotating/floating 3D object as hero centerpiece | 3D hero — see `references/3d-hero.md` |

---

## Lenis smooth scroll

Lenis replaces native scroll with a smoothed, lagless equivalent. **Always include this on premium builds** — it's what gives Awwwards sites their signature "butter" feel. Wire it to GSAP's ticker so ScrollTrigger positions stay accurate:

```js
const lenis = new Lenis({
  duration: 1.2,
  easing: t => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // exponential out
  smoothWheel: true,
  touchMultiplier: 1.5,
});

// Critical: keep ScrollTrigger synced with Lenis's virtual scroll position
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add(time => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);

// Bonus: tab visibility easter egg (seen on Truus.co)
const originalTitle = document.title;
document.addEventListener('visibilitychange', () => {
  document.title = document.hidden ? 'Hey, come back! 👋' : originalTitle;
});
```

`duration: 1.2` is the sweet spot — shorter feels jerky, longer feels sluggish. Adjust `easing` to taste: a custom cubic-bezier or the exponential shown above both read as "premium" compared to a plain `linear`.

---

## Custom cursor with elastic follower

A two-layer cursor: a small fast dot tracks exactly at the pointer, a larger ring follows with lag and elastically expands on hover. **Include on all desktop builds** — it's an immediate quality signal.

```html
<div id="cursor-dot"></div>
<div id="cursor-ring"></div>

<style>
  * { cursor: none; }
  #cursor-dot, #cursor-ring {
    position: fixed; top: 0; left: 0; border-radius: 50%;
    pointer-events: none; z-index: 9999;
    transform: translate(-50%, -50%);
    will-change: transform;
  }
  #cursor-dot { width: 6px; height: 6px; background: currentColor; }
  #cursor-ring {
    width: 32px; height: 32px;
    border: 1.5px solid rgba(255,255,255,0.5);
    background: transparent;
    mix-blend-mode: difference;
  }
</style>

<script>
const dot = document.getElementById('cursor-dot');
const ring = document.getElementById('cursor-ring');

// quickTo is much more performant than gsap.to on every mousemove
const dotX = gsap.quickTo(dot, 'x', { duration: 0.1, ease: 'power3.out' });
const dotY = gsap.quickTo(dot, 'y', { duration: 0.1, ease: 'power3.out' });
const ringX = gsap.quickTo(ring, 'x', { duration: 0.55, ease: 'power3.out' });
const ringY = gsap.quickTo(ring, 'y', { duration: 0.55, ease: 'power3.out' });

window.addEventListener('mousemove', e => {
  dotX(e.clientX); dotY(e.clientY);
  ringX(e.clientX); ringY(e.clientY);
});

// Elastic hover expand
document.querySelectorAll('a, button, [data-cursor-hover]').forEach(el => {
  el.addEventListener('mouseenter', () => {
    gsap.to(ring, { scale: 2.5, duration: 0.5, ease: 'elastic.out(1, 0.4)' });
    gsap.to(dot, { scale: 0, duration: 0.2 });
  });
  el.addEventListener('mouseleave', () => {
    gsap.to(ring, { scale: 1, duration: 0.35, ease: 'sine.inOut' });
    gsap.to(dot, { scale: 1, duration: 0.2 });
  });
});

// Hide on mobile (no cursor)
if (!window.matchMedia('(hover: hover)').matches) {
  dot.style.display = 'none'; ring.style.display = 'none';
}
</script>
```

---

## Elastic cursor bubble with context label (Truus.co pattern)

A label bubble ("click", "drag", "view") that springs into view when hovering specific elements, with elastic bounce easing. Seen on the Truus.co Awwwards winner.

```html
<div class="cursor-bubble" id="cursorBubble">click</div>

<style>
  .cursor-bubble {
    position: fixed; top: 0; left: 0; z-index: 9999;
    background: #fff; color: #000;
    padding: 6px 14px; border-radius: 99px;
    font-size: 0.75rem; font-weight: 600; letter-spacing: 0.05em;
    pointer-events: none; opacity: 1;
    transform: translate(-50%, -50%) scale(0) rotate(-30deg);
    will-change: transform;
  }
</style>

<script>
const bubble = document.getElementById('cursorBubble');
gsap.set(bubble, { rotation: -30 });

const bx = gsap.quickTo(bubble, 'x', { duration: 0.5, ease: 'power3' });
const by = gsap.quickTo(bubble, 'y', { duration: 0.5, ease: 'power3' });

window.addEventListener('mousemove', e => { bx(e.clientX + 13); by(e.clientY - 43); });

let hovering = false;
document.addEventListener('mouseover', e => {
  const target = e.target.closest('[data-bubble]');
  if (target && !hovering) {
    hovering = true;
    bubble.textContent = target.dataset.bubble || 'click';
    gsap.killTweensOf(bubble, 'scale,rotation');
    // Elastic spring in — the key is elastic.out easing
    gsap.to(bubble, { scale: 1, rotation: 0, duration: 1.7, delay: 0.1, ease: 'elastic.out(1, 0.4)' });
  } else if (!target && hovering) {
    hovering = false;
    gsap.to(bubble, { scale: 0, rotation: -30, duration: 0.3, ease: 'sine.inOut' });
  }
});
</script>
```

Tag elements: `<a data-bubble="view">`, `<div data-bubble="drag">`, etc.

---

## Pinned scrub timeline

The backbone of every scrollytelling section. The section is pinned (frozen) while a timeline scrubs in lockstep with scroll position.

```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: section,
    start: 'top top',
    end: '+=120%',     // bigger = slower/more deliberate
    scrub: 1,          // 0 = exact, 0.5–1 = pleasantly laggy
    pin: true,
    anticipatePin: 1,  // prevents visible jump as pin kicks in
    invalidateOnRefresh: true,
  },
});

tl.to('.thing-a', { scale: 0.8 }, 0);
tl.to('.thing-b', { opacity: 0 }, 0);
tl.from('.thing-c', { x: '-200%' }, '-=0.6');
```

---

## Split-text mask reveal

Splits headline into words/lines, masks overflow, animates each piece up out of its clipping box. The `mask` option on SplitText is what creates the "slide up out of a slot" premium feel.

```js
const split = SplitText.create(headlineEl, { type: 'words', mask: 'words' });
gsap.set(split.words, { opacity: 0, yPercent: 110 });
gsap.to(split.words, {
  yPercent: 0, opacity: 1,
  stagger: 0.05, ease: 'power2.inOut', duration: 1,
});
```

For multi-line, auto-resplit on resize:
```js
SplitText.create(headlineEl, {
  type: 'lines', mask: 'lines', autoSplit: true,
  onSplit(self) { return gsap.from(self.lines, { y: 120, duration: 0.8, ease: 'power3.out', stagger: 0.05 }); },
});
```

---

## Per-letter slide-in with BezierEasing (musabhassan.com)

Each letter is wrapped in an overflow-hidden mask and slides in from the right, with words offset slightly for a wave effect. Extracted from musabhassan.com, an Awwwards-winning portfolio.

```js
// Parse node's innerHTML and wrap every letter in a mask+block pair
function letterSlideIn(node, { duration = 600, delay = 35, initDelay = 0 } = {}) {
  const original = node.innerHTML;

  // Wrap each word in .a-word, each letter in .a-text-mask > .a-text-block
  function parseLetters(str) {
    let out = '', isTag = false, isWord = false;
    [...str].forEach((ch, i) => {
      if (ch === '<') { isTag = true; if (isWord) { isWord = false; out += '</div>'; } }
      if (str[i - 1] === '>' && ch !== '<') { isTag = false; if (!isWord) { isWord = true; out += '<div class="a-word">'; } }
      if (isTag) { out += ch; }
      else {
        if (ch === ' ' || i === 0) { isWord = !isWord; out += isWord ? '<div class="a-word">' : '</div><span class="a-spacer"> </span>'; }
        if (ch !== ' ') out += `<div class="a-text-mask"><div class="a-text-block">${ch}</div></div>`;
      }
    });
    return out;
  }

  node.innerHTML = parseLetters(node.innerHTML);
  const masks = node.querySelectorAll('.a-text-mask');
  const blocks = node.querySelectorAll('.a-text-block');
  const words = node.querySelectorAll('.a-word');

  // Set initial state
  masks.forEach(m => { m.style.display = 'inline-flex'; m.style.overflowX = 'clip'; m.style.overflowY = 'visible'; m.style.transform = 'translateX(80%)'; });
  blocks.forEach(b => { b.style.transform = 'translateX(150%)'; });
  words.forEach(w => { w.style.display = 'inline-block'; w.style.whiteSpace = 'nowrap'; });

  // Custom BezierEasing — the signature curve from musabhassan.com
  function bezier(p1x, p1y, p2x, p2y) {
    return t => {
      // Simplified cubic bezier approximation
      const cx = 3 * p1x, bx = 3 * (p2x - p1x) - cx, ax = 1 - cx - bx;
      const cy = 3 * p1y, by = 3 * (p2y - p1y) - cy, ay = 1 - cy - by;
      let t2 = t * t, t3 = t2 * t;
      return ay * t3 + by * t2 + cy * t;
    };
  }
  const ease = bezier(0.2, 0.58, 0.43, 1);

  let frame = 0;
  const totalFrames = Math.round(duration / 16.67);

  function tick() {
    frame++;
    const t = ease(Math.min(frame / totalFrames, 1));
    masks.forEach(m => { m.style.transform = `translateX(${80 - t * 80}%)`; });
    blocks.forEach(b => { b.style.transform = `translateX(${150 - t * 150}%)`; });
    if (frame < totalFrames) requestAnimationFrame(tick);
    else node.innerHTML = original; // restore original HTML
  }

  setTimeout(tick, initDelay);
}
```

Usage: `letterSlideIn(document.querySelector('h1'), { initDelay: 200 });`

---

## Text scramble reveal

Headline/counter text resolves from random glyphs into real characters — a "decoding" feel.

```js
function scrambleText(el, { duration = 1200, charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@#%&' } = {}) {
  const original = el.textContent;
  const totalFrames = Math.round(duration / 33);
  let frame = 0;
  const tick = () => {
    frame++;
    const revealCount = Math.floor((frame / totalFrames) * original.length);
    el.textContent = original.split('').map((ch, i) =>
      ch === ' ' || i < revealCount ? ch : charset[(Math.random() * charset.length) | 0]
    ).join('');
    if (frame < totalFrames) requestAnimationFrame(tick);
    else el.textContent = original;
  };
  tick();
}

document.querySelectorAll('[data-scramble]').forEach((el, i) => {
  setTimeout(() => scrambleText(el), i * 150);
});
```

---

## Blur-reveal on scroll (Kintaro)

Elements fade in from blurred+shifted state as they scroll into view. Cleaner and more distinctive than plain opacity fades. From Kintaro's Awwwards site.

```css
.blur-reveal {
  opacity: 0;
  filter: blur(15px);
  transform: translateY(30px);
  transition: opacity 0.9s cubic-bezier(0.25, 0.46, 0.45, 0.94),
              filter 0.9s cubic-bezier(0.25, 0.46, 0.45, 0.94),
              transform 0.9s cubic-bezier(0.25, 0.46, 0.45, 0.94);
  will-change: opacity, filter, transform;
}
.blur-reveal.in-view {
  opacity: 1; filter: blur(0px); transform: translateY(0);
}
```

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry, i) => {
    if (entry.isIntersecting) {
      setTimeout(() => entry.target.classList.add('in-view'), i * 80);
    }
  });
}, { threshold: 0.1, rootMargin: '-60px' });

document.querySelectorAll('.blur-reveal').forEach(el => observer.observe(el));
```

For staggered children: add `.blur-reveal` to each child, they'll auto-stagger from the IntersectionObserver callback index.

---

## WebGL noise-distortion image (musabhassan.com)

Images warp with simplex noise on scroll — the signature effect from musabhassan.com. Requires Three.js. Creates a curved, fabric-like scroll distortion with RGB-split at high velocity.

```html
<script type="importmap">
{ "imports": { "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js" } }
</script>
<script type="module">
import * as THREE from 'three';

// --- Simplex noise (Ian McEwan / Ashima Arts, MIT license) ---
const NOISE_GLSL = `
vec3 mod289v3(vec3 x){return x-floor(x*(1./289.))*289.;}
vec4 mod289v4(vec4 x){return x-floor(x*(1./289.))*289.;}
vec4 permute(vec4 x){return mod289v4(((x*34.)+10.)*x);}
vec4 taylorInvSqrt(vec4 r){return 1.79284291400159-.85373472095314*r;}
float snoise(vec3 v){
  const vec2 C=vec2(1./6.,1./3.);const vec4 D=vec4(0.,.5,1.,2.);
  vec3 i=floor(v+dot(v,C.yyy));vec3 x0=v-i+dot(i,C.xxx);
  vec3 g=step(x0.yzx,x0.xyz);vec3 l=1.-g;
  vec3 i1=min(g.xyz,l.zxy);vec3 i2=max(g.xyz,l.zxy);
  vec3 x1=x0-i1+C.xxx;vec3 x2=x0-i2+C.yyy;vec3 x3=x0-D.yyy;
  i=mod289v3(i);
  vec4 p=permute(permute(permute(i.z+vec4(0.,i1.z,i2.z,1.))+i.y+vec4(0.,i1.y,i2.y,1.))+i.x+vec4(0.,i1.x,i2.x,1.));
  float n_=.142857142857;vec3 ns=n_*D.wyz-D.xzx;
  vec4 j=p-49.*floor(p*ns.z*ns.z);vec4 x_=floor(j*ns.z);vec4 y_=floor(j-7.*x_);
  vec4 x=x_*ns.x+ns.yyyy;vec4 y=y_*ns.x+ns.yyyy;vec4 h=1.-abs(x)-abs(y);
  vec4 b0=vec4(x.xy,y.xy);vec4 b1=vec4(x.zw,y.zw);
  vec4 s0=floor(b0)*2.+1.;vec4 s1=floor(b1)*2.+1.;vec4 sh=-step(h,vec4(0.));
  vec4 a0=b0.xzyw+s0.xzyw*sh.xxyy;vec4 a1=b1.xzyw+s1.xzyw*sh.zzww;
  vec3 p0=vec3(a0.xy,h.x);vec3 p1=vec3(a0.zw,h.y);vec3 p2=vec3(a1.xy,h.z);vec3 p3=vec3(a1.zw,h.w);
  vec4 norm=taylorInvSqrt(vec4(dot(p0,p0),dot(p1,p1),dot(p2,p2),dot(p3,p3)));
  p0*=norm.x;p1*=norm.y;p2*=norm.z;p3*=norm.w;
  vec4 m=max(.5-vec4(dot(x0,x0),dot(x1,x1),dot(x2,x2),dot(x3,x3)),0.);m=m*m;
  return 105.*dot(m*m,vec4(dot(p0,x0),dot(p1,x1),dot(p2,x2),dot(p3,x3)));
}
`;

const vertexShader = NOISE_GLSL + `
uniform vec2 uOffset;
uniform float uTime;
varying vec2 vUv;
varying float vWave;
#define M_PI 3.1415926535897932384626433832795
vec3 deformationCurve(vec3 pos,vec2 uv,vec2 offset){
  pos.x+=sin(uv.y*M_PI)*offset.x;
  pos.y+=sin(uv.x*M_PI)*offset.y;
  return pos;
}
void main(){
  vUv=uv;
  vec3 newPos=deformationCurve(position,uv,uOffset);
  vec3 p=position;
  p.z+=snoise(vec3(p.x*2.5+uTime,p.y,p.z))*.05;
  vWave=p.z*.2;
  gl_Position=projectionMatrix*modelViewMatrix*vec4(newPos,1.);
}`;

const fragmentShader = `
uniform sampler2D uTexture;
uniform float uAlpha;
uniform vec2 uOffset;
varying vec2 vUv;
varying float vWave;
vec3 rgbShift(sampler2D tex,vec2 uv,vec2 offset){
  float r=texture2D(tex,uv+offset+vWave).r;
  vec2 gb=texture2D(tex,uv+vWave).gb;
  return vec3(r,gb);
}
void main(){
  vec3 color=rgbShift(uTexture,vUv,uOffset);
  gl_FragColor=vec4(color,uAlpha);
}`;

// Set up one distortion mesh per .distort-img element
document.querySelectorAll('.distort-img').forEach(imgEl => {
  const container = imgEl.parentElement;
  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  const W = container.offsetWidth, H = container.offsetHeight;
  renderer.setSize(W, H);
  renderer.domElement.style.cssText = 'position:absolute;inset:0;pointer-events:none;';
  container.style.position = 'relative';
  container.appendChild(renderer.domElement);
  imgEl.style.opacity = '0'; // hide the real img, show via WebGL

  const scene = new THREE.Scene();
  const camera = new THREE.OrthographicCamera(-0.5, 0.5, 0.5, -0.5, 0.1, 10);
  camera.position.z = 1;

  const texture = new THREE.TextureLoader().load(imgEl.src);
  const geo = new THREE.PlaneGeometry(1, 1, 32, 32);
  const mat = new THREE.ShaderMaterial({
    vertexShader, fragmentShader,
    uniforms: { uTexture: { value: texture }, uOffset: { value: new THREE.Vector2(0, 0) }, uAlpha: { value: 1 }, uTime: { value: 0 } },
    transparent: true,
  });
  scene.add(new THREE.Mesh(geo, mat));

  let lastY = 0, velocity = 0;
  function render(t) {
    mat.uniforms.uTime.value = t * 0.001;
    const curY = container.getBoundingClientRect().top;
    velocity += (curY - lastY) * 0.005; // scroll velocity → offset
    velocity *= 0.85; // damping
    lastY = curY;
    mat.uniforms.uOffset.value.set(velocity * 0.3, velocity);
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  }
  requestAnimationFrame(render);
});
</script>
```

HTML: wrap your image in a container with `position: relative`, add class `distort-img` to the `<img>`:
```html
<div style="position:relative;overflow:hidden;width:600px;height:400px;">
  <img class="distort-img" src="your-image.jpg" style="width:100%;height:100%;object-fit:cover;">
</div>
```

---

## Interactive particle field (Kintaro)

A canvas-based particle field that floats gently and scatters away from the mouse cursor. Pauses when off-screen. Theme-aware (dark/light). Extracted from Kintaro's Awwwards-winning portfolio.

```html
<canvas id="particleCanvas" style="position:absolute;inset:0;width:100%;height:100%;pointer-events:none;z-index:0;"></canvas>

<script>
(function () {
  const canvas = document.getElementById('particleCanvas');
  const ctx = canvas.getContext('2d');
  let particles = [], animId = 0, isVisible = false;
  let dpr = window.devicePixelRatio || 1;
  let mouse = { x: -1000, y: -1000, radius: 140 * dpr };

  class Particle {
    constructor(w, h) {
      this.x = Math.random() * w; this.y = Math.random() * h;
      this.baseVx = (Math.random() - 0.5) * 0.6 * dpr;
      this.baseVy = (Math.random() - 0.5) * 0.6 * dpr;
      this.vx = this.baseVx; this.vy = this.baseVy;
      this.r = (Math.random() * 1.5 + 0.5) * dpr;
      this.alpha = Math.random() * 0.5 + 0.15;
    }
    update(w, h) {
      if (this.x < 0 || this.x > w) { this.baseVx *= -1; this.vx *= -1; }
      if (this.y < 0 || this.y > h) { this.baseVy *= -1; this.vy *= -1; }
      const dx = mouse.x - this.x, dy = mouse.y - this.y;
      const dist = Math.sqrt(dx * dx + dy * dy);
      if (dist < mouse.radius) {
        const force = (mouse.radius - dist) / mouse.radius;
        this.vx += -(dx / dist) * force * 3 * dpr;
        this.vy += -(dy / dist) * force * 3 * dpr;
      }
      this.vx += (this.baseVx - this.vx) * 0.04;
      this.vy += (this.baseVy - this.vy) * 0.04;
      const speed = Math.sqrt(this.vx * this.vx + this.vy * this.vy);
      if (speed > 4 * dpr) { this.vx = (this.vx / speed) * 4 * dpr; this.vy = (this.vy / speed) * 4 * dpr; }
      this.x += this.vx; this.y += this.vy;
    }
    draw(isDark) {
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.r, 0, Math.PI * 2);
      ctx.fillStyle = isDark ? `rgba(255,255,255,${this.alpha})` : `rgba(0,0,0,${this.alpha})`;
      ctx.fill();
    }
  }

  function resize() {
    dpr = window.devicePixelRatio || 1;
    const rect = canvas.getBoundingClientRect();
    canvas.width = rect.width * dpr; canvas.height = rect.height * dpr;
    mouse.radius = 140 * dpr;
    const count = Math.max(20, Math.min(150, Math.floor((rect.width * rect.height) / 12000)));
    particles = Array.from({ length: count }, () => new Particle(canvas.width, canvas.height));
  }

  function animate() {
    if (!isVisible) { animId = 0; return; }
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const isDark = document.documentElement.classList.contains('dark') || document.body.dataset.theme === 'dark';
    particles.forEach(p => { p.update(canvas.width, canvas.height); p.draw(isDark); });
    animId = requestAnimationFrame(animate);
  }

  new IntersectionObserver(([e]) => {
    isVisible = e.isIntersecting;
    if (isVisible && !animId) animId = requestAnimationFrame(animate);
  }, { threshold: 0 }).observe(canvas);

  window.addEventListener('resize', resize);
  window.addEventListener('mousemove', e => {
    const r = canvas.getBoundingClientRect();
    mouse.x = (e.clientX - r.left) * dpr; mouse.y = (e.clientY - r.top) * dpr;
  });
  window.addEventListener('mouseleave', () => { mouse.x = -1000; mouse.y = -1000; });

  resize();
})();
</script>
```

Place the canvas inside any `position: relative; overflow: hidden` section — hero, contact section, footer.

---

## SVG progress ring preloader (Kintaro)

A logo-centered spinner with an animated SVG stroke circle that fills as assets load. Slides up on exit. Extracted from Kintaro's Awwwards portfolio.

```html
<div id="preloader" style="position:fixed;inset:0;z-index:99999;display:flex;align-items:center;justify-content:center;background:var(--bg);">
  <!-- Outer ring: dotted rotating ring -->
  <svg style="position:absolute;width:144px;height:144px;animation:spin 12s linear infinite;opacity:0.4;" viewBox="0 0 100 100">
    <circle cx="50" cy="50" r="48" fill="none" stroke="currentColor" stroke-width="0.5" stroke-dasharray="2 6"/>
  </svg>
  <!-- Progress ring: fills as assets load -->
  <svg style="position:absolute;width:144px;height:144px;transform:rotate(-90deg);" viewBox="0 0 100 100">
    <circle cx="50" cy="50" r="46" fill="none" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
    <circle id="progressRing" cx="50" cy="50" r="46" fill="none" stroke="currentColor" stroke-width="1.5"
      stroke-dasharray="289" stroke-dashoffset="289" style="transition:stroke-dashoffset 0.4s ease;"/>
  </svg>
  <!-- Logo / brand name in center -->
  <div style="position:relative;width:56px;height:56px;display:flex;align-items:center;justify-content:center;font-weight:700;font-size:1.1rem;">
    LOGO
  </div>
</div>
<style>
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
<script>
const preloader = document.getElementById('preloader');
const ring = document.getElementById('progressRing');
const circumference = 289;

function setProgress(fraction) {
  ring.style.strokeDashoffset = circumference * (1 - fraction);
}

function exitPreloader() {
  gsap.to(preloader, {
    y: '-100%', duration: 0.9, ease: 'power3.inOut',
    onComplete: () => { preloader.style.display = 'none'; document.body.style.overflow = ''; }
  });
}

// Real asset gating — race against 6s timeout
document.body.style.overflow = 'hidden';
const assets = [...document.querySelectorAll('img, video')];
let loaded = 0;
function tick() { loaded++; setProgress(loaded / assets.length); if (loaded >= assets.length) exitPreloader(); }

if (assets.length === 0) {
  setTimeout(exitPreloader, 1200);
} else {
  assets.forEach(el => {
    const done = () => tick();
    if (el.tagName === 'IMG') el.complete ? done() : el.addEventListener('load', done, { once: true });
    else el.readyState >= 3 ? done() : el.addEventListener('loadeddata', done, { once: true });
    el.addEventListener('error', done, { once: true });
  });
  setTimeout(exitPreloader, 6000); // failsafe
}
</script>
```

---

## SVG curved loader wipe (portfolio-main)

A full-viewport SVG overlay with a bezier-curved bottom edge. On load, the curve flattens and the panel slides up — creating a "wave lifts away" entrance. Extracted from portfolio-main Awwwards entry.

```html
<div id="loaderPanel" style="position:fixed;bottom:0;left:0;right:0;z-index:99998;pointer-events:none;">
  <svg id="loaderSvg" style="width:100%;display:block;" preserveAspectRatio="none">
    <path id="loaderPath" fill="var(--fg, #0a0a0a)"/>
  </svg>
</div>

<script>
(function () {
  const svg = document.getElementById('loaderSvg');
  const path = document.getElementById('loaderPath');
  const W = window.innerWidth, H = window.innerHeight;
  svg.setAttribute('viewBox', `0 0 ${W} ${H * 1.35}`);
  svg.style.height = `${H * 1.35}px`;

  // Curved bottom edge — Q bezier midpoint pulled down
  const curve = H + H * 0.3;
  const flat = H;
  path.setAttribute('d', `M0 0 L${W} 0 L${W} ${H} Q${W/2} ${curve} 0 ${H} Z`);

  // Animate curve → flat → slide up
  gsap.to({ t: 0 }, {
    t: 1, duration: 0.9, ease: 'power3.out',
    onUpdate() {
      const mid = curve + (flat - curve) * this.targets()[0].t;
      path.setAttribute('d', `M0 0 L${W} 0 L${W} ${H} Q${W/2} ${mid} 0 ${H} Z`);
    },
    onComplete() {
      gsap.to('#loaderPanel', { y: `-${H * 1.35}px`, duration: 0.7, ease: 'power3.inOut',
        onComplete: () => document.getElementById('loaderPanel').remove() });
    }
  });
})();
</script>
```

---

## 3D card flip

A card that rotates like a playing card: front shows an image, back reveals text on scroll.

```html
<div class="card-shell">
  <div class="card-inner">
    <div class="card-face front"><img src="…"></div>
    <div class="card-face back">…back content…</div>
  </div>
</div>
```

```css
.card-shell { perspective: 1000px; }
.card-inner { transform-style: preserve-3d; position: relative; }
.card-face { position: absolute; inset: 0; backface-visibility: hidden; }
.card-face.back { transform: rotateY(180deg); }
```

```js
const flipTl = gsap.timeline({ paused: true })
  .to('.card-inner', { rotationY: 180, duration: 0.8, ease: 'power2.inOut', force3D: true })
  .set('.card-face.front', { visibility: 'hidden' }, 0.4)
  .set('.card-face.back', { visibility: 'visible' }, 0.4);

let flipped = false;
ScrollTrigger.create({
  trigger: '.card-shell', start: 'top 60%',
  onUpdate(self) {
    if (self.progress > 0.5 && !flipped) { flipped = true; flipTl.play(); }
    if (self.progress <= 0.5 && flipped) { flipped = false; flipTl.reverse(); }
  }
});
```

---

## Clip-path mask morph

Hero video/image shrinks from full-screen into a smaller frame elsewhere on the page — the Sondr-style hero effect.

```js
function getInset() {
  const heroRect = heroEl.getBoundingClientRect();
  const targetRect = targetEl.getBoundingClientRect();
  return {
    top: targetRect.top - heroRect.top, left: targetRect.left - heroRect.left,
    right: heroRect.right - targetRect.right, bottom: heroRect.bottom - targetRect.bottom,
  };
}

gsap.timeline({ scrollTrigger: { trigger: heroEl, start: 'top top', end: '+=200%', scrub: 1, pin: true, invalidateOnRefresh: true } })
  .to(videoWrapperEl, {
    clipPath: () => { const i = getInset(); return `inset(${i.top}px ${i.right}px ${i.bottom}px ${i.left}px round 12px)`; },
    ease: 'power2.out', duration: 1,
  }, 0);
```

---

## Header blur-on-scroll

Nav goes glassy almost immediately as scroll begins — a short, separate ScrollTrigger.

```js
gsap.to('header', {
  backdropFilter: 'blur(30px)',
  backgroundColor: 'rgba(0,0,0,0.3)',
  scrollTrigger: { trigger: 'main', start: 'top top', end: '+=100', scrub: 1 },
});
```

---

## Staggered fly-in

Elements enter from opposite directions and settle, timed to overlap the previous animation's tail:

```js
tl.from('.col-left img', { x: '-200%', y: '150%', stagger: 0.15, ease: 'power3.out', duration: 1 }, '-=0.6');
tl.from('.col-right img', { x: '200%', y: '150%', stagger: 0.15, ease: 'power3.out', duration: 1 }, '-=0.8');
```

---

## Velocity marquee (infinite loop + scroll speed-up)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.13.0/Observer.min.js"></script>
```

```js
function horizontalLoop(items, { speed = 1, repeat = -1, paddingRight = 0 } = {}) {
  items = gsap.utils.toArray(items);
  const tl = gsap.timeline({ repeat, defaults: { ease: 'none' }, onReverseComplete: () => tl.totalTime(tl.rawTime() + tl.duration() * 100) });
  const widths = [], xPercents = [], pixelsPerSecond = speed * 100;
  let totalWidth;
  gsap.set(items, { xPercent: (i, el) => { widths[i] = parseFloat(gsap.getProperty(el, 'width', 'px')); return (xPercents[i] = (parseFloat(gsap.getProperty(el, 'x', 'px')) / widths[i]) * 100 + gsap.getProperty(el, 'xPercent')); } });
  gsap.set(items, { x: 0 });
  const last = items.length - 1;
  totalWidth = items[last].offsetLeft + (xPercents[last] / 100) * widths[last] - items[0].offsetLeft + items[last].offsetWidth * gsap.getProperty(items[last], 'scaleX') + paddingRight;
  items.forEach((item, i) => {
    const curX = (xPercents[i] / 100) * widths[i], distanceToLoop = item.offsetLeft + curX - items[0].offsetLeft + widths[i] * gsap.getProperty(item, 'scaleX');
    tl.to(item, { xPercent: ((curX - distanceToLoop) / widths[i]) * 100, duration: distanceToLoop / pixelsPerSecond }, 0)
      .fromTo(item, { xPercent: ((curX - distanceToLoop + totalWidth) / widths[i]) * 100 }, { xPercent: xPercents[i], duration: (curX - distanceToLoop + totalWidth - curX) / pixelsPerSecond, immediateRender: false }, distanceToLoop / pixelsPerSecond);
  });
  tl.progress(1, true).progress(0, true);
  return tl;
}

const marqueeTl = horizontalLoop('.marquee-item', { repeat: -1, paddingRight: 30 });
Observer.create({
  onChangeY(self) {
    const factor = (self.deltaY < 0 ? 1 : -1) * 2.5;
    gsap.timeline({ defaults: { ease: 'none' } })
      .to(marqueeTl, { timeScale: factor * 2.5, duration: 0.2, overwrite: true })
      .to(marqueeTl, { timeScale: factor / 2.5, duration: 1 }, '+=0.3');
  },
});
```

---

## Magnetic cursor preview

Project list rows where hover reveals a floating image that smoothly follows the cursor:

```js
const preview = document.querySelector('.preview-image');
const previewImg = preview.querySelector('img');
const moveX = gsap.quickTo(preview, 'x', { duration: 0.6, ease: 'power3.out' });
const moveY = gsap.quickTo(preview, 'y', { duration: 0.8, ease: 'power3.out' });

document.addEventListener('mousemove', e => { moveX(e.clientX + 24); moveY(e.clientY + 24); });

document.querySelectorAll('.project-row').forEach(row => {
  row.addEventListener('mouseenter', () => {
    previewImg.src = row.dataset.previewSrc;
    gsap.to(row.querySelector('.overlay'), { clipPath: 'polygon(0 0, 100% 0, 100% 100%, 0 100%)', duration: 0.4, ease: 'power2.out' });
    gsap.to(preview, { opacity: 1, scale: 1, duration: 0.3 });
  });
  row.addEventListener('mouseleave', () => {
    gsap.to(row.querySelector('.overlay'), { clipPath: 'polygon(0 100%, 100% 100%, 100% 100%, 0 100%)', duration: 0.3 });
    gsap.to(preview, { opacity: 0, scale: 0.95, duration: 0.3 });
  });
});
```

Guard with `window.matchMedia('(hover: hover)').matches` — don't ship this half-wired on touch devices.

---

## Magnetic button pull

Button content pulls toward the cursor within a trigger radius — a subtle, satisfying interaction that signals craft.

```js
document.querySelectorAll('.mag-btn').forEach(btn => {
  btn.addEventListener('mousemove', e => {
    const rect = btn.getBoundingClientRect();
    const cx = rect.left + rect.width / 2, cy = rect.top + rect.height / 2;
    const dx = e.clientX - cx, dy = e.clientY - cy;
    gsap.to(btn, { x: dx * 0.3, y: dy * 0.3, duration: 0.4, ease: 'power3.out' });
    gsap.to(btn.querySelector('.btn-label'), { x: dx * 0.15, y: dy * 0.15, duration: 0.4, ease: 'power3.out' });
  });
  btn.addEventListener('mouseleave', () => {
    gsap.to(btn, { x: 0, y: 0, duration: 0.7, ease: 'elastic.out(1, 0.4)' });
    gsap.to(btn.querySelector('.btn-label'), { x: 0, y: 0, duration: 0.7, ease: 'elastic.out(1, 0.4)' });
  });
});
```

---

## Slide-in nav overlay

Full-screen or half-screen nav panel that slides in with staggered link reveals:

```js
gsap.set('.nav-panel', { xPercent: 100 });
gsap.set(['.nav-panel a', '.nav-panel .contact-line'], { autoAlpha: 0, x: -20 });

const panelTl = gsap.timeline({ paused: true })
  .to('.nav-panel', { xPercent: 0, duration: 0.8, ease: 'power3.out' })
  .to('.nav-panel a', { autoAlpha: 1, x: 0, stagger: 0.08, duration: 0.5, ease: 'power2.out' }, '<');

const iconTl = gsap.timeline({ paused: true })
  .to('.burger span:first-child', { rotate: 45, y: 4, duration: 0.3, ease: 'power2.inOut' })
  .to('.burger span:last-child', { rotate: -45, y: -4, duration: 0.3, ease: 'power2.inOut' }, '<');

let open = false;
document.querySelector('.burger').addEventListener('click', () => {
  open ? (panelTl.reverse(), iconTl.reverse()) : (panelTl.play(), iconTl.play());
  open = !open;
});
```

---

## Sticky stacking cards

Cards each become `position: sticky` and visually stack as you scroll, like a deck being dealt down:

```css
.stack-card { position: sticky; }
```

```js
document.querySelectorAll('.stack-card').forEach((card, i) => {
  card.style.top = `calc(8vh + ${i * 2.5}rem)`;
  gsap.from(card, { y: 120, opacity: 0, duration: 1, ease: 'circ.out', scrollTrigger: { trigger: card, start: 'top 85%' } });
});
```

---

## Horizontal text drift

Big text rows gliding at different speeds/directions on scroll — gives capabilities sections a kinetic energy:

```js
document.querySelectorAll('[data-drift]').forEach(row => {
  gsap.to(row, { xPercent: parseFloat(row.dataset.drift), ease: 'none', scrollTrigger: { trigger: row, scrub: true } });
});
```

Alternate values: `data-drift="20"`, `data-drift="-30"`, `data-drift="100"`. Set `overflow-x: clip` on a parent.

---

## Hover mood-color shift

Background subtly retints to each project's color on hover — cheap, effective, atmospheric:

```css
html { transition: background-color 0.6s cubic-bezier(0.19, 1, 0.22, 1); }
```

```html
<div class="project-row" data-mood="#1a354e">…</div>
```

```js
document.querySelectorAll('[data-mood]').forEach(row => {
  row.addEventListener('mouseenter', () => document.documentElement.style.backgroundColor = row.dataset.mood);
  row.addEventListener('mouseleave', () => document.documentElement.style.backgroundColor = '');
});
```

---

## Asset-gated preloader (vanilla counter)

A loading screen that tracks real image/video load progress and only releases the page once ready:

```js
function gateOnAssetsLoaded(onProgress, onDone) {
  const assets = [...document.querySelectorAll('img, video')];
  if (!assets.length) return onDone();
  let loaded = 0;
  assets.forEach(el => {
    const mark = () => { loaded++; onProgress(loaded / assets.length); if (loaded === assets.length) onDone(); };
    if (el.tagName === 'IMG') el.complete ? mark() : el.addEventListener('load', mark, { once: true });
    else el.readyState >= 3 ? mark() : el.addEventListener('loadeddata', mark, { once: true });
    el.addEventListener('error', mark, { once: true });
  });
}

gateOnAssetsLoaded(
  fraction => document.querySelector('.loader-pct').textContent = Math.floor(fraction * 100),
  () => gsap.timeline()
    .to('.loader', { yPercent: -100, duration: 0.8, ease: 'power3.inOut' })
    .from('.hero-content', { opacity: 0, y: 40, duration: 1 }, '-=0.3')
);
setTimeout(() => document.querySelector('.loader')?.remove(), 6000); // failsafe
```

---

## General notes

- **Easing**: `power2.inOut`, `power2.out`, `power3.out`, `elastic.out(1, 0.4)` cover almost everything. The elastic easing is distinctive and worth using on cursor/button interactions.
- **`force3D: true`** on any transform-heavy tween keeps it GPU-composited.
- **Test reverse.** Scrubbed timelines run backward when scrolling up — always check that reversing looks intentional.
- **Touch guards.** Cursor, magnetic, and WebGL effects need `matchMedia('(hover: hover)')` or `window.innerWidth > 768` guards.
- **`will-change: transform`** on anything that animates; remove it after animation completes on non-looping elements.
- **Lenis + ScrollTrigger.** Always wire Lenis to ScrollTrigger via `lenis.on('scroll', ScrollTrigger.update)` or positions drift.
- **Canvas particles / WebGL.** Use IntersectionObserver to pause loops when off-screen — don't burn GPU on invisible sections.
- **3D hero.** See `references/3d-hero.md` — reach for it only when user explicitly wants WebGL 3D, not as a default "make it impressive" choice.
