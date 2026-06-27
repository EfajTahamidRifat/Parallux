# 3D / WebGL hero

A hero built around a real 3D object — a model that floats, rotates, and catches studio-style lighting — reads as a tier above flat image/video heroes. This is a bigger lift than everything else in this skill, so reach for it only when:

- The user explicitly asks for a "3D hero", "WebGL", "rotating model/product", or similar, **or**
- They've uploaded (or clearly have) a `.glb`/`.gltf` 3D model to use.

Don't default to this for a generic "make it more impressive" request — ask first (see the asset note at the bottom) rather than assuming everyone wants a 3D centerpiece.

There are two versions below: a single-file Three.js build (matches this skill's normal single-HTML-file deliverable) and a React Three Fiber version (for the Next.js integration path in `references/nextjs-integration.md`).

## Vanilla Three.js (single HTML file)

Three.js ships as ES modules now, so load it via an import map rather than a plain `<script src>` tag:

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
  }
}
</script>
<script type="module">
import * as THREE from "three";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import gsap from "https://cdn.jsdelivr.net/npm/gsap@3.13.0/index.js";

const canvasHost = document.querySelector(".hero-3d");

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(35, canvasHost.clientWidth / canvasHost.clientHeight, 0.1, 100);
camera.position.set(0, 0, 6);

const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(canvasHost.clientWidth, canvasHost.clientHeight);
canvasHost.appendChild(renderer.domElement);

// Studio-style lighting reads much better on a model than one harsh light.
scene.add(new THREE.AmbientLight(0xffffff, 0.6));
const key = new THREE.DirectionalLight(0xffffff, 1.2);
key.position.set(4, 5, 6);
scene.add(key);
const rim = new THREE.DirectionalLight(0x88aaff, 0.5);
rim.position.set(-5, -2, -4);
scene.add(rim);

let model;
new GLTFLoader().load("models/hero-model.glb", (gltf) => {
  model = gltf.scene;
  scene.add(model);

  // Entrance: drop + fade in, then settle into a slow idle spin.
  gsap.from(model.position, { y: 3, duration: 1.6, ease: "power3.out" });
  gsap.from(model.scale, { x: 0, y: 0, z: 0, duration: 1.2, ease: "back.out(1.4)" });

  gsap.to(model.rotation, {
    y: "+=" + Math.PI * 2,
    duration: 18,
    repeat: -1,
    ease: "none",
  });

  // Optional: tie a bit of extra rotation to scroll, layered on top of the idle spin.
  gsap.to(model.rotation, {
    x: 0.3,
    scrollTrigger: { trigger: ".hero-3d", start: "top top", end: "bottom top", scrub: 1 },
  });
});

function resize() {
  camera.aspect = canvasHost.clientWidth / canvasHost.clientHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(canvasHost.clientWidth, canvasHost.clientHeight);
}
window.addEventListener("resize", resize);

renderer.setAnimationLoop(() => renderer.render(scene, camera));
</script>
```

```css
.hero-3d { position: absolute; inset: 0; z-index: -1; }
.hero-3d canvas { display: block; }
```

Notes:
- `<script type="module">` runs after the DOM is parsed by default — no `DOMContentLoaded` wrapper needed, but `gsap.registerPlugin(ScrollTrigger)` still needs to happen if you're using ScrollTrigger here (load it the same way as `gsap` via the jsdelivr ESM path, or fall back to the regular CDN `<script>` tag for GSAP/ScrollTrigger and only use the import map for `three`).
- `renderer.setAnimationLoop` is Three.js's built-in render loop — don't also hand-roll a `requestAnimationFrame` loop alongside it.
- If the user has no 3D model yet and just wants the *idea* of a WebGL hero to look at, a procedural primitive (`new THREE.IcosahedronGeometry(1.5, 4)` with a simple `MeshStandardMaterial`) is a reasonable stand-in — it's generated geometry, not a hotlinked/borrowed asset, so it's fine to use as a placeholder while they source a real model. Say plainly that it's a placeholder shape.

## React Three Fiber (Next.js integration path)

When the project already uses React (see `references/nextjs-integration.md` for the general setup), prefer `@react-three/fiber` + `@react-three/drei` over hand-rolled Three.js — `Canvas` manages the renderer/render-loop lifecycle for you, and `drei`'s `Environment`/`Lightformer` give cheap, good-looking studio lighting without manually placing lights.

```bash
npm install three @react-three/fiber @react-three/drei @gsap/react
```

```tsx
"use client"
import { useRef } from "react"
import { useGLTF } from "@react-three/drei"
import { useGSAP } from "@gsap/react"
import gsap from "gsap"

export function Model(props) {
  const ref = useRef(null)
  const { scene } = useGLTF("/models/hero-model.glb")

  useGSAP(() => {
    gsap.from(ref.current.position, { y: 3, duration: 1.6, ease: "power3.out" })
    gsap.to(ref.current.rotation, { y: `+=${Math.PI * 2}`, duration: 18, repeat: -1, ease: "none" })
  }, [])

  return <primitive ref={ref} object={scene} {...props} />
}

useGLTF.preload("/models/hero-model.glb")
```

```tsx
"use client"
import { Canvas } from "@react-three/fiber"
import { Environment, Float, Lightformer } from "@react-three/drei"
import { Model } from "./Model"

const Hero3D = () => (
  <Canvas camera={{ position: [0, 0, 6], fov: 35 }}>
    <ambientLight intensity={0.5} />
    <Float speed={0.6}>
      <Model scale={1} />
    </Float>
    <Environment resolution={256}>
      <group rotation={[-Math.PI / 3, 4, 1]}>
        <Lightformer form="circle" intensity={2} position={[0, 5, -9]} scale={10} />
        <Lightformer form="circle" intensity={2} position={[0, 3, 1]} scale={10} />
        <Lightformer form="circle" intensity={2} position={[-5, -1, -1]} scale={10} />
      </group>
    </Environment>
  </Canvas>
)

export default Hero3D
```

`Float` (from drei) gives a free, cheap idle bob/sway without writing your own sine-wave animation loop — layer GSAP entrance/scroll-tied tweens on top of it as shown in `Model`, rather than fighting it for control of the same properties.

Gate the rest of the page behind real asset loading when a 3D model is involved — `.glb` files are heavier than images and a flash of an empty hero looks worse than a brief loader. Drei's `useProgress` hook tracks loading across everything `useGLTF`/`useTexture` etc. are fetching:

```tsx
import { useProgress } from "@react-three/drei"

const { progress } = useProgress() // 0–100, drives a preloader same as the vanilla pattern in references/patterns.md
```

## Asset note

3D heroes need an actual `.glb`/`.gltf` file — there's no reasonable placeholder for a specific branded model the way a solid-color block stands in for a photo. If the user wants this effect but doesn't have a model ready, say so plainly and offer the procedural-primitive fallback above, or suggest a quick model from a tool like [Spline](https://spline.design) or a marketplace like [Sketchfab](https://sketchfab.com) (for assets they have a license to use) as a starting point.
