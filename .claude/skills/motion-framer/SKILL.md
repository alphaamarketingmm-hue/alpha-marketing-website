---
name: motion-framer
description: "Animation and motion design with Motion (formerly Framer Motion) for plain HTML/JS and React. Use when adding, building, designing, reviewing, fixing, or optimizing animations: transitions, scroll reveals (fade/slide in on scroll), parallax, scroll-linked progress, staggered lists, hover/tap micro-interactions, gestures, drag, page/element enter-exit animations, layout/shared-element animations, spring physics, and orchestrated entrance sequences for landing pages and marketing sites. Covers the vanilla-JS API (animate, scroll, inView, stagger, spring via ESM/CDN, no build step) and the React API (motion components, AnimatePresence, variants, useScroll, useTransform, useReducedMotion, MotionConfig). Includes performance (transform/opacity, GPU compositing) and accessibility (prefers-reduced-motion) best practices."
---

# Motion (Framer Motion) — Animation Intelligence

Production guidance for building animations with **Motion** — the library formerly
published as **Framer Motion**. Motion ships one package that powers **both vanilla
JavaScript and React**, so the same mental model applies whether the project has a
build step or is a single static HTML file.

- npm package: **`motion`** (current major: **v12**). Legacy `framer-motion` still works.
- Vanilla JS entry: `import { animate, scroll, inView, stagger, spring } from "motion"`
- React entry: `import { motion, AnimatePresence } from "motion/react"`
- Docs: https://motion.dev · Source: https://github.com/motiondivision/motion

## Pick the right API first

| Project type | Use | Import from |
|---|---|---|
| **Plain HTML / static site / no build step** (this is the default for marketing sites) | **Vanilla API** | `"motion"` via ESM CDN |
| React / Next.js / Remix / Astro-React | **React API** | `"motion/react"` |
| Vue / Svelte | Vanilla API (`animate`, `inView`, `scroll`) — the React components are React-only | `"motion"` |

> **For a static HTML page** (e.g. a hand-written `index.html`): use the **vanilla API**.
> `motion.div` / `<motion.*>` components and all `use*` hooks are **React-only** and will
> not work without React. Reach for `animate()`, `inView()`, `scroll()`, and `stagger()`.

Motion complements CSS — it does not replace it. Keep simple hover/focus states and
looping decorative effects in CSS (`transition`, `@keyframes`). Use Motion for things
CSS does poorly: **scroll-triggered reveals, scroll-linked/scrubbed effects, staggered
sequences, spring physics, gesture-driven motion, and layout (FLIP) animations.**

## Install

### Static HTML (no build) — recommended for this project
```html
<script type="module">
  import { animate, scroll, inView, stagger } from "https://cdn.jsdelivr.net/npm/motion@12.40.0/+esm";
  // ...your code
</script>
```
Pin the version (replace `12.40.0` with the latest from https://motion.dev). `esm.sh`
works too: `import { animate } from "https://esm.sh/motion@12.40.0"`.

### Bundled / React
```bash
npm install motion
```
```js
import { animate } from "motion";            // vanilla
import { motion } from "motion/react";       // React
```

---

## Vanilla JS API

### `animate(target, keyframes, options)`
`target` is a CSS selector string, an `Element`, or a list of elements. Returns a
playback control that is awaitable (`await animate(...)`) and exposes
`.stop() .pause() .play() .complete() .time .speed`.
```js
// Transform shortcuts (x, y, scale, rotate) animate independently of each other
animate("#hero", { opacity: 1, y: 0 }, { duration: 0.6, ease: "easeOut" });

// Keyframe arrays + looping
animate(".pulse", { scale: [1, 1.08, 1] }, { duration: 1.6, repeat: Infinity, ease: "easeInOut" });

// Spring physics
animate("#cta", { scale: 1.05 }, { type: "spring", stiffness: 300, damping: 18 });
```
Prefer transform shortcuts (`x`, `y`, `scale`, `rotate`) over animating `left/top/width`.

### `inView(target, onEnter, options)` — scroll reveals
Built on IntersectionObserver (work happens off the main thread). `onEnter` runs when an
element enters the viewport; return a function to run when it leaves. Returns a `stop()`.
```js
inView(".reveal", (element) => {
  animate(element, { opacity: 1, y: 0 }, { duration: 0.6, ease: "easeOut" });
  // return () => animate(element, { opacity: 0, y: 40 }); // re-hide on exit (optional)
}, { amount: 0.3 });   // amount: 0–1 | "some" | "all"; also `margin: "0px 0px -100px 0px"`
```
Pair with a CSS starting state so content is correct before JS runs:
```css
.reveal { opacity: 0; transform: translateY(40px); }
```

### `scroll(onScrollOrAnimation, options)` — scroll-linked / scrubbed
Uses the native ScrollTimeline where possible (hardware-accelerated). Progress is `0→1`.
```js
// Reading progress bar (whole page). #bar needs: transform-origin: left;
scroll(animate("#bar", { scaleX: [0, 1] }, { ease: "linear" }));

// Element-linked parallax: track one element across the viewport
scroll(
  animate("#hero-img", { y: [-80, 80] }, { ease: "linear" }),
  { target: document.querySelector("#hero-img"), offset: ["start end", "end start"] }
);

// Raw progress callback
scroll((progress) => { /* progress is 0..1 */ });
```
`offset` pairs describe [target edge, viewport edge]: `"start end"` = target's start meets
viewport's end. Common: `["start end", "end start"]` (full pass-through).

### `stagger(each, options)` — sequence many elements
```js
animate(
  ".card",
  { opacity: 1, y: 0 },
  { delay: stagger(0.08, { start: 0.1, from: "first" }) }  // from: "first"|"last"|"center"|index
);
```

---

## React API (`motion/react`)

```jsx
import { motion, AnimatePresence, useScroll, useTransform, useReducedMotion } from "motion/react";
```

### Animatable components & core props
```jsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0 }}
  transition={{ duration: 0.4, ease: "easeOut" }}
/>
```

### Gestures (micro-interactions)
```jsx
<motion.button whileHover={{ scale: 1.05 }} whileTap={{ scale: 0.95 }} />
<motion.div drag dragConstraints={{ left: 0, right: 200 }} />
```

### Scroll reveal — `whileInView`
```jsx
<motion.section
  initial={{ opacity: 0, y: 50 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, amount: 0.3 }}
  transition={{ duration: 0.6, ease: "easeOut" }}
/>
```

### Variants + stagger (orchestration)
```jsx
const container = { hidden: {}, show: { transition: { staggerChildren: 0.1, delayChildren: 0.2 } } };
const item = { hidden: { opacity: 0, y: 20 }, show: { opacity: 1, y: 0 } };

<motion.ul variants={container} initial="hidden" whileInView="show" viewport={{ once: true }}>
  {items.map((t) => <motion.li key={t} variants={item}>{t}</motion.li>)}
</motion.ul>
```

### Enter/exit — `AnimatePresence`
```jsx
<AnimatePresence>
  {open && (
    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} />
  )}
</AnimatePresence>
```

### Layout & shared element
```jsx
<motion.div layout />                 {/* animates size/position changes automatically (FLIP) */}
<motion.div layoutId="card-1" />      {/* same layoutId across components = shared-element transition */}
```

### Scroll-linked hooks
```jsx
const { scrollYProgress } = useScroll();
const width = useTransform(scrollYProgress, [0, 1], ["0%", "100%"]);
<motion.div style={{ width }} />       {/* top-of-page reading progress bar */}
```

---

## Transition options (both APIs)

```js
{
  type: "tween",          // "tween" (default) | "spring" | "inertia"
  duration: 0.4,          // seconds (tween)
  ease: "easeOut",        // "linear" | "easeIn/Out/InOut" | "circIn"… | [0.17,0.67,0.83,0.67]
  delay: 0,
  repeat: 0,              // number | Infinity
  repeatType: "loop",     // "loop" | "reverse" | "mirror"
  // spring only:
  stiffness: 300, damping: 20, mass: 1, bounce: 0.2,
  // React orchestration (variants):
  staggerChildren: 0.1, delayChildren: 0.2, when: "beforeChildren"
}
```
Feel guide: **micro-interactions 0.15–0.25s**, **UI transitions 0.2–0.4s**, **entrances
0.5–0.8s**. Springs feel more natural than tweens for interactive/gesture motion. For a
premium/luxury brand, favor longer easing, restraint, and `ease: "easeOut"` entrances.

---

## Performance (non-negotiable for 60fps)

- **Animate only `transform` (x/y/scale/rotate) and `opacity`.** These run on the GPU
  compositor and skip layout + paint.
- **Never animate `width`, `height`, `top`, `left`, `margin`, `padding`** — they trigger
  layout reflow and jank. Use `scale`, transforms, or React's `layout` prop instead.
- `filter` (blur) and `box-shadow` are animatable but GPU-expensive — use sparingly.
- Motion sets `will-change` automatically during an animation; don't leave it on permanently.
- `inView` and `scroll` use IntersectionObserver / ScrollTimeline — prefer them over manual
  `scroll` event listeners.

## Accessibility — respect `prefers-reduced-motion`

Always provide a reduced-motion path. Motion is decoration; content must work without it.

Vanilla:
```js
const reduce = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
animate("#hero", { opacity: 1, y: 0 }, { duration: reduce ? 0 : 0.6 });
```
React:
```jsx
const reduce = useReducedMotion();
// or wrap the app: <MotionConfig reducedMotion="user"> disables transform/layout
//   animations for users who prefer reduced motion while keeping opacity fades.
```
Rules: never convey essential information through motion alone; avoid large parallax,
auto-playing loops, and flashing for reduced-motion users; keep a visible focus state.

## Anti-patterns

- Using `<motion.*>` components or `use*` hooks in a non-React (plain HTML) project — they won't run.
- Animating layout properties (`width/height/top/left`) instead of transforms.
- No `prefers-reduced-motion` fallback.
- Over-animating: everything moving at once competes for attention. Motion should *guide* the eye.
- Sluggish durations (>0.6s) on hover/feedback; or instant (0s) entrances that feel cheap.
- Forgetting `transform-origin` on scaleX progress bars / scaleY reveals.
- Hiding content with `opacity:0` and never revealing it if JS fails — ensure graceful no-JS state.

## Project note (this repo)

`index.html` is a **static HTML/CSS** marketing site (no build step) that already uses CSS
`@keyframes`/`transition`. Use the **vanilla ESM-CDN** path above. Best fits here:
scroll reveals (`inView`), a scroll progress bar / parallax hero (`scroll`), and staggered
section entrances (`animate` + `stagger`) — layered on top of the existing CSS, gated behind
`prefers-reduced-motion`.

## Quick reference

```js
// VANILLA (static HTML)
import { animate, inView, scroll, stagger } from "https://cdn.jsdelivr.net/npm/motion@12.40.0/+esm";
animate("#el", { opacity: 1, y: 0 }, { duration: 0.5 });           // tween
animate("#el", { scale: 1.1 }, { type: "spring", stiffness: 300 }); // spring
inView(".reveal", (el) => animate(el, { opacity: 1, y: 0 }), { amount: 0.3 }); // scroll reveal
scroll(animate("#bar", { scaleX: [0, 1] }));                        // scroll progress
animate(".item", { opacity: 1 }, { delay: stagger(0.08) });        // stagger
```
```jsx
// REACT
import { motion, AnimatePresence } from "motion/react";
<motion.div initial={{opacity:0,y:20}} whileInView={{opacity:1,y:0}} viewport={{once:true}} />
<motion.button whileHover={{scale:1.05}} whileTap={{scale:0.95}} />
<AnimatePresence>{open && <motion.div exit={{opacity:0}} />}</AnimatePresence>
```
