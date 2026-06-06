---
name: threejs
description: "3D graphics, WebGL scenes, and real-time visuals with three.js for plain HTML/JS and React (react-three-fiber). Use when adding, building, designing, reviewing, fixing, or optimizing 3D content: hero backgrounds, particle systems / starfields, wireframe and solid geometry, GLTF/GLB model loading, PBR materials and lighting, orbit/scroll/pointer-driven cameras, post-processing (bloom/glow), instancing, shaders (GLSL/ShaderMaterial), and responsive resize/pixel-ratio handling. Covers the script-tag global (window.THREE) path, the modern ESM + importmap path with addons (OrbitControls, GLTFLoader, EffectComposer), and react-three-fiber (@react-three/fiber + drei). Includes performance (draw calls, instancing, dispose, devicePixelRatio cap), accessibility (prefers-reduced-motion, pause off-screen), and graceful WebGL fallbacks. Source: https://github.com/mrdoob/three.js"
---

# three.js — 3D / WebGL Intelligence

Production guidance for building real-time 3D with **three.js**, the standard WebGL
library. It runs anywhere there's a `<canvas>` — a single static HTML file or a full
React app via **react-three-fiber**.

- npm package: **`three`**. Versioned by **revision** (e.g. `r128`, `r160`+). Check the
  latest at https://github.com/mrdoob/three.js/releases.
- Script-tag global: `window.THREE` (older `*.min.js` builds; **no ES modules**).
- ESM entry: `import * as THREE from "three"` + addons from `three/addons/...`.
- React entry: `@react-three/fiber` (renderer) + `@react-three/drei` (helpers).
- Docs: https://threejs.org/docs · Examples: https://threejs.org/examples

## Pick the right setup first

| Project type | Use | Load from |
|---|---|---|
| **Plain HTML / static site, no build** (default for marketing sites) | **ESM + importmap** (preferred) or the **global script** | jsDelivr / unpkg CDN |
| React / Next.js / Astro-React | **react-three-fiber** + drei | npm |
| Vue / Svelte | Core ESM API directly (R3F is React-only) | npm / CDN |

> **Global `window.THREE` (`three.min.js`) vs ESM:** the classic single-file build exposes
> a global `THREE` but **does not include addons** (OrbitControls, GLTFLoader,
> EffectComposer) and is frozen at older revisions (latest non-module UMD build was ~r148).
> For anything beyond core meshes — model loading, controls, post-processing — use the
> **ESM + importmap** path below. Do **not** mix a global build with `import ... from "three"`.

## Install / load

### Static HTML — ESM + importmap (recommended)
```html
<script type="importmap">
{ "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
} }
</script>
<script type="module">
  import * as THREE from "three";
  import { OrbitControls } from "three/addons/controls/OrbitControls.js";
  import { GLTFLoader }   from "three/addons/loaders/GLTFLoader.js";
  // ...your scene
</script>
```
Pin the version (npm uses `0.160.0`-style numbers that mirror `r160`). The `three/addons/`
trailing slash matters — it maps the whole `examples/jsm/` tree.

### Static HTML — global script (core only, legacy)
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
  if (window.THREE) { /* THREE.Scene(), THREE.Mesh()… core only, no addons */ }
</script>
```

### Bundled / React
```bash
npm install three
npm install @react-three/fiber @react-three/drei   # React only
```

---

## Core API — the four things every scene needs

```js
// 1. Scene — the container for everything
const scene = new THREE.Scene();

// 2. Camera — PerspectiveCamera(fov, aspect, near, far)
const camera = new THREE.PerspectiveCamera(50, innerWidth / innerHeight, 0.1, 100);
camera.position.z = 6;

// 3. Renderer — bind to a canvas, set size + pixel ratio
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));   // cap at 2 — critical for perf

// 4. A mesh = geometry + material
const mesh = new THREE.Mesh(
  new THREE.IcosahedronGeometry(2, 1),
  new THREE.MeshStandardMaterial({ color: 0xd4af37, metalness: 0.8, roughness: 0.3 })
);
scene.add(mesh);

// Render loop (use the clock for frame-rate-independent motion)
const clock = new THREE.Clock();
renderer.setAnimationLoop(() => {           // preferred over requestAnimationFrame (XR-safe)
  mesh.rotation.y = clock.getElapsedTime() * 0.4;
  renderer.render(scene, camera);
});
```

### Resize handling (always include)
```js
addEventListener("resize", () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();          // required after changing aspect
  renderer.setSize(innerWidth, innerHeight);
});
```

## Geometry & materials

- **Geometry:** `BoxGeometry`, `SphereGeometry`, `IcosahedronGeometry`, `TorusGeometry`,
  `PlaneGeometry`, `BufferGeometry` (custom). Build particles with a `BufferGeometry` +
  a `position` `BufferAttribute` + `THREE.Points`.
- **Materials (cheap → expensive):**
  - `MeshBasicMaterial` — unlit, flat color. **No lights needed.** Great for wireframes,
    glows, additive particles, UI-ish 3D.
  - `MeshStandardMaterial` / `MeshPhysicalMaterial` — PBR (metalness/roughness). **Requires
    lights.** Realistic surfaces, the default for product/model work.
  - `LineBasicMaterial` + `WireframeGeometry` / `LineSegments` — wireframe looks.
  - `ShaderMaterial` / `RawShaderMaterial` — custom GLSL (vertex/fragment) for effects.
- **Wireframe + faint solid combo** (the look used in this repo's hero):
```js
const geo  = new THREE.IcosahedronGeometry(2.4, 1);
const wire = new THREE.LineSegments(new THREE.WireframeGeometry(geo),
  new THREE.LineBasicMaterial({ color: 0xf6d98a, transparent: true, opacity: 0.55 }));
const solid = new THREE.Mesh(geo,
  new THREE.MeshBasicMaterial({ color: 0xd4af37, transparent: true, opacity: 0.05, side: THREE.DoubleSide }));
```

## Lights (only for Standard/Physical/Lambert/Phong materials)

```js
scene.add(new THREE.AmbientLight(0xffffff, 0.4));        // global fill
const key = new THREE.DirectionalLight(0xffffff, 1.2);
key.position.set(3, 5, 2);
scene.add(key);
// also: PointLight, SpotLight, HemisphereLight, RectAreaLight
```
`MeshBasicMaterial` / `Points` / lines ignore lights — skip lighting if the whole scene is
unlit (faster, simpler).

## Particles / starfield

```js
const N = 1200, pos = new Float32Array(N * 3);
for (let i = 0; i < N * 3; i++) pos[i] = (Math.random() - 0.5) * 40;
const g = new THREE.BufferGeometry();
g.setAttribute("position", new THREE.BufferAttribute(pos, 3));
const stars = new THREE.Points(g, new THREE.PointsMaterial({
  color: 0xecdcb4, size: 0.05, transparent: true, opacity: 0.8,
  blending: THREE.AdditiveBlending, depthWrite: false,   // glow + correct overlap
}));
scene.add(stars);
```

## Loading models (GLTF/GLB — the standard format)

```js
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/addons/loaders/DRACOLoader.js";   // for compressed meshes

const draco = new DRACOLoader().setDecoderPath("https://www.gstatic.com/draco/v1/decoders/");
const loader = new GLTFLoader().setDRACOLoader(draco);
loader.load("/model.glb", (gltf) => scene.add(gltf.scene),
  undefined, (err) => console.error(err));
```
Prefer **`.glb`** (binary, single file). Compress with `gltf-transform` / Draco/Meshopt.
Models with PBR materials need lights and usually an environment map (`RoomEnvironment` or
an HDRI via `RGBELoader`) for realistic reflections.

## Controls & camera motion

```js
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;        // inertia — call controls.update() each frame
controls.autoRotate = true; controls.autoRotateSpeed = 0.6;
```
For background heroes you usually **don't** want OrbitControls (it grabs pointer/scroll).
Instead, ease the camera toward pointer/scroll yourself (lerp), as the repo hero does:
```js
camera.position.x += (targetX - camera.position.x) * 0.04;   // smooth follow
camera.lookAt(0, 0, 0);
```

## Post-processing (bloom / glow)

```js
import { EffectComposer } from "three/addons/postprocessing/EffectComposer.js";
import { RenderPass }     from "three/addons/postprocessing/RenderPass.js";
import { UnrealBloomPass } from "three/addons/postprocessing/UnrealBloomPass.js";

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));
composer.addPass(new UnrealBloomPass(new THREE.Vector2(innerWidth, innerHeight), 0.8, 0.4, 0.85));
// in the loop: composer.render()  (instead of renderer.render)
// on resize: composer.setSize(innerWidth, innerHeight)
```
Bloom is GPU-heavy — gate it behind a perf/device check; it's overkill on mobile.

---

## react-three-fiber (`@react-three/fiber`)

Declarative three.js. Every JSX tag maps to a three.js class (`<mesh>` → `THREE.Mesh`).

```jsx
import { Canvas, useFrame } from "@react-three/fiber";
import { OrbitControls, Stars, Environment } from "@react-three/drei";
import { useRef } from "react";

function Core() {
  const ref = useRef();
  useFrame((state, delta) => { ref.current.rotation.y += delta * 0.4; });  // render loop
  return (
    <mesh ref={ref}>
      <icosahedronGeometry args={[2, 1]} />
      <meshStandardMaterial color="#d4af37" metalness={0.8} roughness={0.3} />
    </mesh>
  );
}

export default function Hero() {
  return (
    <Canvas camera={{ position: [0, 0, 6], fov: 50 }} dpr={[1, 2]}>
      <ambientLight intensity={0.4} />
      <directionalLight position={[3, 5, 2]} intensity={1.2} />
      <Core />
      <Stars />
      <Environment preset="studio" />
      <OrbitControls enableDamping autoRotate />
    </Canvas>
  );
}
```
- `args={[...]}` are the constructor arguments. Nest geometry/material as children of the mesh.
- **drei** has ready-made helpers: `OrbitControls`, `Stars`, `Environment`, `useGLTF`,
  `Float`, `MeshTransmissionMaterial`, `Html`, `Text3D`, `Bloom` (via `@react-three/postprocessing`).
- R3F disposes objects automatically on unmount — a big reason to prefer it in React.

---

## Performance (non-negotiable for 60fps)

- **Cap pixel ratio:** `renderer.setPixelRatio(Math.min(devicePixelRatio, 2))`. Retina at
  3× quadruples fragment work for no visible gain.
- **Minimize draw calls.** Each mesh ≈ one draw call. For many identical objects use
  **`InstancedMesh`** (thousands of copies in one call) instead of a loop of meshes.
- **Reuse geometries and materials** — create once, share across meshes; don't allocate in
  the render loop.
- **Dispose what you remove:** `geometry.dispose()`, `material.dispose()`, `texture.dispose()`,
  `renderer.dispose()`. WebGL resources are **not** garbage-collected. (R3F handles this.)
- **Pause when off-screen / tab hidden:** drive the loop only while the canvas intersects the
  viewport (IntersectionObserver) and on `document.visibilitychange`. The repo hero does this.
- Prefer `renderer.setAnimationLoop(fn)` over a manual `requestAnimationFrame` loop (handles
  WebXR and pauses with the page).
- Keep particle counts and post-processing modest on mobile; branch on
  `matchMedia("(max-width: 768px)")` or `navigator.hardwareConcurrency`.

## Accessibility & resilience

- **Respect `prefers-reduced-motion`:** stop/freeze auto-rotation and camera drift; render a
  single static frame instead of an animated loop.
  ```js
  const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
  if (reduce) renderer.render(scene, camera);        // one frame, no loop
  else renderer.setAnimationLoop(tick);
  ```
- **3D is decoration** for a marketing site: it must sit *behind* content
  (`pointer-events:none`, low z-index) and never gate information or interaction.
- **Graceful WebGL fallback:** guard for missing context; keep a CSS background so the page
  looks correct if WebGL is unavailable or the script fails.
  ```js
  if (!renderer.getContext()) { /* leave the CSS fallback in place */ }
  ```
- Don't trap scroll: only attach OrbitControls/scroll-zoom when the 3D is the primary content.

## Anti-patterns

- Mixing a global `three.min.js` build with `import ... from "three"` (two copies of THREE).
- Creating geometries/materials/vectors **inside** the render loop (GC churn, leaks).
- Forgetting `camera.updateProjectionMatrix()` after changing `aspect`/`fov`.
- Never calling `.dispose()` when tearing down a scene (memory leak in SPAs).
- Uncapped `devicePixelRatio` → mobile meltdown.
- Running the loop while the canvas is off-screen or the tab is hidden (wasted battery).
- Using PBR materials (`MeshStandardMaterial`) with no lights → everything renders black.
- A foreground 3D canvas without `pointer-events:none` swallowing clicks on real UI.

## Project note (this repo)

`index.html` is a **static, single-file** marketing site (no build step) that **already uses
three.js**: it lazy-loads the **global `three.min.js` (r128)** build from cdnjs and renders an
animated hero into `<canvas id="three3d">` — a wireframe `IcosahedronGeometry` core, three
`TorusGeometry` rings, orbiting `SphereGeometry` nodes, and an additive-blended `Points`
starfield, all in brand black & gold (`#d4af37` / `#f6d98a` / `#ecdcb4`). It already does the
right things: caps `setPixelRatio` at 2, pauses via `IntersectionObserver` when the hero
leaves the viewport, eases the camera toward the pointer, and fades the canvas in with the
`.on` class.

Guidance when changing it:
- **Stay on the existing global-`THREE` pattern** for small tweaks (it's already loaded and
  working). Only switch to the **ESM + importmap** path if you need an addon the global build
  lacks — `GLTFLoader` (load a 3D logo/product), `OrbitControls`, or `EffectComposer`/
  `UnrealBloomPass` for real bloom. Don't load both builds at once.
- Keep the canvas decorative: behind content, `pointer-events:none`, brand palette.
- Add a `prefers-reduced-motion` guard (render one frame instead of the loop) — the current
  scene animates unconditionally; that's the first thing to harden.
- Keep particle counts and any post-processing light on mobile.

## Quick reference

```js
// STATIC HTML — ESM + importmap (preferred for addons)
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(50, innerWidth/innerHeight, 0.1, 100); camera.position.z = 6;
const renderer = new THREE.WebGLRenderer({ canvas, antialias:true, alpha:true });
renderer.setSize(innerWidth, innerHeight); renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
scene.add(new THREE.Mesh(new THREE.IcosahedronGeometry(2,1), new THREE.MeshBasicMaterial({color:0xd4af37})));
const clock = new THREE.Clock();
renderer.setAnimationLoop(() => { renderer.render(scene, camera); });
addEventListener("resize", () => { camera.aspect = innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });
```
```jsx
// REACT — react-three-fiber
import { Canvas, useFrame } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
<Canvas camera={{ position:[0,0,6], fov:50 }} dpr={[1,2]}>
  <ambientLight intensity={0.4} /><directionalLight position={[3,5,2]} />
  <mesh><icosahedronGeometry args={[2,1]} /><meshStandardMaterial color="#d4af37" /></mesh>
  <OrbitControls enableDamping />
</Canvas>
```
</content>
</invoke>
