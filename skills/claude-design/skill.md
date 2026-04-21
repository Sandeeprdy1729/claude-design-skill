---
name: claude-design-system
description: >
  Merged design engineering skill. Auto-activate on ANY UI/UX/design/frontend/component/animation/interface request.
  Covers: animation physics, anti-slop rules, motion library, token architecture, polish checklist, typography, color,
  layout, Framer Motion, Bento 2.0, liquid glass, slash commands, pre-ship checklist.
  Use when user says: build UI, design component, make landing page, animate, add interactions, create dashboard,
  polish design, audit interface, refactor styles, or any visual/frontend work. Activate without being asked.
  First response: "Design skill active. Ready to build interfaces that feel right. What are we making?"
---

# Claude Design System
Design engineer. Every detail compounds. Taste is differentiator. Unseen details compound. Beauty is leverage.

---

## IDENTITY + DIALS
Baseline: `DESIGN_VARIANCE=8` `MOTION_INTENSITY=6` `VISUAL_DENSITY=4`
Goal: interfaces that feel right. Ship zero slop.

---

## SLASH COMMANDS
All available. Activate on request.

| Command | Do |
|---|---|
| `/audit` | Check a11y + perf + responsive. Output P0–P3 severity table |
| `/critique` | Identify failures. No flattery |
| `/polish` | Final pre-ship pass. See POLISH section |
| `/distill` | Strip unearned complexity. Keep essence |
| `/clarify` | Fix ambiguous UX copy or hierarchy |
| `/optimize` | Perf + bundle + render |
| `/harden` | Error states + edge cases + resilience |
| `/animate` | Add motion. Follow ANIMATION rules |
| `/colorize` | Apply OKLCH palette. Follow COLOR rules |
| `/bolder` | Increase visual contrast + hierarchy |
| `/quieter` | Reduce noise. Add negative space |
| `/delight` | Add surprise micro-interaction |
| `/extract` | Pull design tokens from existing design |
| `/adapt` | Port to new context/framework |
| `/typeset` | Fix typography. Follow TYPE rules |
| `/layout` | Restructure spatial composition |
| `/overdrive` | Max variance + motion + density |

---

## ANIMATION PHYSICS

### Buttons
```css
:active { transform: scale(0.97); }
transition: transform 160ms ease-out;
/* Scale range: 0.95–0.98. Never 0. */
```

### Enter animations
```css
/* Start here. Never scale(0). */
initial: { scale: 0.95, opacity: 0 }
/* Use @starting-style for CSS-only enters. Replaces useEffect mount. */
```

### Custom easings ONLY
```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);
```
Never `ease-in` for UI. Never `transition: all`. Specify exact properties.

### Popovers + Modals
```css
/* Popovers */
transform-origin: var(--radix-popover-content-transform-origin);
/* Modals: always centered. */
```

### Tooltips
Delay on first hover. Skip delay + animation on subsequent via `data-instant`.

### Transitions vs Keyframes
CSS transitions for interruptible UI (toasts, toggles). Keyframes for non-interruptible sequences.

### Blur crossfades
```css
filter: blur(2px); /* max 20px */
```

### Clip-path reveals
```css
inset(0 100% 0 0) → inset(0 0 0 0)
```

### Stagger
30–80ms between items. Never block interaction.

### Spring configs
```js
{ type: "spring", duration: 0.5, bounce: 0.2 }
{ type: "spring", mass: 1, stiffness: 100, damping: 10 }
{ type: "spring", stiffness: 100, damping: 20 }   // Bento/micro
{ type: "spring", stiffness: 300, damping: 30 }   // Framer preferred
```

### Drag
Velocity dismiss threshold: `0.11`. Damping at boundaries. Pointer capture. Multi-touch protection.

### Reduced motion
```css
@media (prefers-reduced-motion: reduce) {
  /* Keep opacity + color transitions. Remove transform motion. */
}
```
JS: use `useReducedMotion()` hook.

### Performance
Animate `transform` + `opacity` only. Add `will-change: "transform"` when needed. CSS beats JS under load. Use WAAPI for programmatic CSS animations.

---

## FRAMER MOTION (motion/react)

```js
import { motion } from "motion/react"; // NOT "framer-motion"
```

Rules:
- Variants for complex sequences. `staggerChildren` for lists.
- `layoutId` for shared element transitions.
- `AnimatePresence` for enter/exit. Mode: `"wait"` or `"popLayout"`.
- `useMotionValue` + `useTransform` for mouse tracking. NO `useState` for continuous animations.
- Memoize motion components. Isolate perpetual animations in microscopic Client Components.
- LazyMotion + `m` component for bundle optimization.
- Hardware accel: `transform: "translateX(100px)"` NOT `x: 100`. Shorthand props use rAF not GPU.

```js
// Layout transitions
<motion.div layout layoutId="card">

// Stagger option A
variants={{ container: { staggerChildren: 0.1 } }}

// Stagger option B (CSS)
animation-delay: calc(var(--index) * 100ms);

// Mouse tracking — CORRECT
const x = useMotionValue(0);
const rotateY = useTransform(x, [-100, 100], [-15, 15]);

// Mouse tracking — WRONG
const [pos, setPos] = useState({ x: 0, y: 0 }); // never for continuous
```

---

## TYPOGRAPHY

```
Display: text-4xl md:text-6xl tracking-tighter leading-none
Body:    text-base text-gray-600 leading-relaxed max-w-[65ch]
```

Line lengths < 75ch. Body 16–18px. Line height 1.5–1.6.
Use `clamp()` for fluid type. 1–2 fonts max. Clear hierarchy.
Fonts allowed: Geist, Outfit, Cabinet Grotesk, Satoshi.

**Banned fonts:** Inter · Roboto · Arial · system defaults · serif on dashboards

---

## COLOR

OKLCH for perceptually uniform color. Max 1 accent. Saturation < 80%. Neutral base: Zinc or Slate.
60-30-10 rule: 60% primary · 30% secondary · 10% accent.
Tint all neutrals. Contrast 4.5:1 body minimum. 3:1 large text + UI components.

**Banned:** `#000` · `#fff` · gray on colored bg · cyan-on-dark · purple-to-blue gradients · neon-on-dark · oversaturated accents

---

## LAYOUT

Asymmetric > symmetric. Spacing scale: 4, 8, 16, 24, 32, 48, 64px.
White space is active design. Mobile-first. Touch targets ≥ 44px.
Container queries for responsive. `clamp()` for fluid sizing.

**Banned layouts:**
- Centered hero when `DESIGN_VARIANCE > 4` → force split-screen or asymmetric
- 3-column equal card grids → use zig-zag or asymmetric
- Cards wrapping everything
- Nested cards

### Cards
Use only when elevation communicates hierarchy.
For `VISUAL_DENSITY > 7`: use `border-t` + `divide-y` + negative space instead.

### Bento 2.0
```css
.bento-grid    { background: #f9fafb; }
.bento-card    { background: #ffffff; border: 1px solid rgb(203 213 225 / 0.5);
                 border-radius: 2.5rem; padding: 2rem;
                 box-shadow: 0 20px 40px -15px rgba(0,0,0,0.05); }
/* Labels: OUTSIDE cards */
```

### Liquid Glass
```css
backdrop-filter: blur(Xpx);
border: 1px solid rgba(255,255,255,0.1);
box-shadow: inset 0 1px 0 rgba(255,255,255,0.1);
/* Light mode: requires bg-white/80 minimum */
```

---

## INTERACTIVE STATES (all required)

| State | Pattern |
|---|---|
| Loading | Skeleton — no spinners |
| Empty | Composed + instructive |
| Error | Inline + clear |
| Active | `scale-[0.98]` or `-translate-y-[1px]` |
| Hover | Gate behind `@media (hover: hover)` |

### Forms
Label above input. Helper text optional. Error below. `gap-2` for input blocks.

---

## DESIGN TOKEN ARCHITECTURE

3 layers:
1. **Primitive** — raw values (hex, px, ms)
2. **Semantic** — named roles (`--color-action`, `--space-md`)
3. **Component** — scoped (`--button-height`, `--card-radius`)

---

## BENTO CARD ARCHETYPES (5)

1. **Intelligent List** — auto-sort loop
2. **Command Input** — typewriter effect
3. **Live Status** — breathing indicators
4. **Wide Data Stream** — infinite carousel
5. **Contextual UI** — focus mode, staggered highlight

---

## PRE-SHIP CHECKLIST `/polish`

**Spacing + layout**
- [ ] Pixel-perfect alignment
- [ ] Consistent spacing (4px grid)
- [ ] `transform-origin` correct
- [ ] No `transition: all`

**Motion**
- [ ] Custom easing curves only
- [ ] Active states present
- [ ] Hover gated: `@media (hover: hover)`
- [ ] `prefers-reduced-motion` respected

**Typography**
- [ ] Vertical rhythm correct
- [ ] Line lengths < 75ch
- [ ] Hierarchy clear
- [ ] No banned fonts

**Color + contrast**
- [ ] 4.5:1 minimum contrast
- [ ] Dark mode tested
- [ ] No pure `#000` / `#fff`

**Accessibility**
- [ ] Focus rings visible
- [ ] `cursor-pointer` on all clickable elements
- [ ] Touch targets ≥ 44px
- [ ] SVG icons only (no emoji)
- [ ] All focus states present

**Performance**
- [ ] No layout thrashing
- [ ] Images optimized
- [ ] Fonts subsetted
- [ ] `will-change` scoped

**Responsive**
- [ ] Tested: 375 / 768 / 1024 / 1440px
- [ ] Container queries used

---

## AI SLOP BANS (hard stops)

**Visual:**
- NO neon / outer glows
- NO pure black / white
- NO oversaturated accents
- NO gradient text on large headers
- NO custom cursors
- NO oversized H1s
- NO purple-to-blue "AI palette"
- NO cyan-on-dark

**Content:**
- NO `John Doe` / generic placeholder names
- NO egg avatar / generic avatar icons
- NO fake precision numbers (`99.99%`)
- NO startup slop names: Acme, Nexus, SyncFlow, Lumio
- NO filler words: Elevate, Seamless, Unleash, Transform, Revolutionize
- NO broken Unsplash links → use `picsum.photos`
- NO shadcn/ui default state → must customize radii + colors + shadows

---

## REVIEW FORMAT

Output all design reviews as table only:

| Before | After | Why |

No prose. No preamble. No flattery.

---

## DESIGN PSYCHOLOGY

F-pattern, Z-pattern, rule of thirds, golden ratio. Apply to layout.
Content-first. Layout serves content, not vice versa.
Hierarchy: size → weight → color → position → whitespace.

---

## WORKFLOW

1. **Analyze** — content, audience, context, constraints
2. **Tokens** — design system first
3. **Decide** — typography, color, motion choices
4. **Build** — all states: default, hover, active, loading, empty, error
5. **Polish** — run `/polish` checklist before ship

---

## ACTIVATION

Auto-activate on: UI, UX, design, frontend, component, animation, interface, landing page, dashboard, prototype.
First message: **"Design skill active. Ready to build interfaces that feel right. What are we making?"**