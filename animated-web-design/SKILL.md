---
name: animated-web-design
description: >
  Build visually stunning, animation-rich websites with cinematic-quality motion design.
  Use this skill whenever the user wants to create a website, landing page, portfolio,
  product page, or any web UI that should look impressive and feel alive. Also use when
  the user mentions animations, transitions, scroll effects, parallax, carousels,
  hover effects, layered designs, or says things like "make it look premium",
  "I want something beautiful", "cinematic feel", "not generic", or "high-end design".
  This skill prevents generic AI-generated website aesthetics and produces distinctive,
  hand-crafted-feeling interfaces with real motion design craft.
---

# Animated Web Design

You build websites that feel alive. Every section breathes, layers shift, elements reveal themselves with purpose. The goal is never "add some animations" — it's to create a cohesive visual experience where motion serves the design and the design serves the content.

## Stack

- **Next.js** (App Router, React 19, TypeScript)
- **Tailwind CSS v4** with custom design tokens
- **CSS animations first** — JS only when CSS can't do it (scroll-triggered state changes, intersection observers)
- **No animation libraries** unless the user requests one. Raw CSS animations are faster, lighter, and give you full control.

## The Anti-Slop Principles

Generic AI websites share telltale signs: centered hero with gradient background, predictable card grids, stock-photo circles, generic blue-purple palette, and zero animation beyond basic fade-ins. Your job is to avoid all of this.

### 1. Every site needs a spatial concept

Before writing any code, decide on the site's **spatial metaphor**. This isn't a theme — it's how elements relate to each other in space and time.

Examples of spatial concepts:
- **Orbital** — elements arranged in circles, rotating layers, radial navigation (like a star chart or clock face)
- **Geological** — deep layered sections that feel like you're descending through strata, parallax depth
- **Theatrical** — elements enter and exit like actors on a stage, curtain-reveal transitions
- **Kinetic typography** — text IS the visual, words move and transform to create the experience
- **Liquid** — flowing transitions, morphing shapes, smooth state changes
- **Mechanical** — gears, interlocking parts, satisfying snap-into-place animations
- **Atmospheric** — particles, fog, gradients that shift subtly, ambient motion

Pick one that fits the content. A luxury brand might use theatrical. A tech product might use mechanical. A creative portfolio might use liquid. Don't mix metaphors randomly.

### 2. Layered compositions over flat layouts

Flat layouts feel dead. Depth makes pages feel rich. Build depth through:

- **Z-index stacking** — overlapping elements at different layers
- **Multiple background layers** — base texture + gradient overlay + decorative elements
- **Foreground/background separation** — hero content in front, decorative elements behind
- **Scale hierarchy** — one massive hero element surrounded by smaller satellite elements
- **Opacity layering** — semi-transparent overlapping shapes create visual complexity

### 3. Motion with purpose

Every animation must answer: **what does this motion communicate?**

- Rotation = ongoing process, cycles, time
- Breathing/pulsing = life, energy, heartbeat
- Parallax = depth, immersion
- Slide-in = arrival, presentation
- Fade = appearance/disappearance, transition between states
- Scale = emphasis, attention, importance
- Float = weightlessness, calm, dreamy

Don't animate something just because you can. Animate it because the motion says something.

## Animation Toolkit

### CSS Keyframe Patterns

Define these in `globals.css` and reference them throughout:

```css
/* Continuous rotation — for decorative elements, loading states, orbital layouts */
@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Breathing pulse — for living elements, CTAs, focal points */
@keyframes breath {
  0%, 100% { transform: scale(0.97); }
  50% { transform: scale(1.03); }
}

/* Gentle float — for cards, icons, ambient elements */
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

/* Infinite horizontal scroll — for carousels, marquees, logo bars */
@keyframes scroll-left {
  from { transform: translateX(0); }
  to { transform: translateX(-33.333%); }
}

/* Shimmer effect — for loading, luxury feel, highlight */
@keyframes shimmer {
  from { background-position: -200% center; }
  to { background-position: 200% center; }
}

/* Subtle parallax drift — for background layers */
@keyframes drift {
  0%, 100% { transform: translate(0, 0); }
  25% { transform: translate(5px, -3px); }
  50% { transform: translate(-3px, 5px); }
  75% { transform: translate(3px, 2px); }
}
```

### Timing and Easing Reference

Easing is the difference between "moves" and "feels alive":

| Purpose | Duration | Easing | Why |
|---------|----------|--------|-----|
| Hover feedback | 200-300ms | `ease-out` | Instant response, smooth settle |
| Entrance animation | 500-800ms | `cubic-bezier(0.16, 1, 0.3, 1)` | Quick start, gentle deceleration |
| Exit animation | 200-400ms | `ease-in` | Gather speed, vanish |
| Continuous ambient | 3-8s | `ease-in-out` | Natural, organic feel |
| Slow rotation | 60-180s | `linear` | Constant, mechanical, celestial |
| Bounce/spring | 500-700ms | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Overshoot then settle |
| Elegant reveal | 800-1200ms | `cubic-bezier(0.25, 0.46, 0.45, 0.94)` | Smooth, sophisticated |

**Performance rules:**
- Only animate `transform` and `opacity` — they're GPU-composited
- Avoid animating `width`, `height`, `margin`, `padding`, `top`, `left` — they trigger layout reflow
- Use `will-change: transform` sparingly and only on elements that actually animate
- Prefer CSS `animation` over JS `requestAnimationFrame` for continuous motion

### Scroll-Triggered Animations

Use IntersectionObserver for revealing elements on scroll. Create a reusable hook:

```tsx
// src/hooks/useScrollReveal.ts
"use client";
import { useEffect, useRef, useState } from "react";

export function useScrollReveal(threshold = 0.15) {
  const ref = useRef<HTMLDivElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const observer = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) setIsVisible(true); },
      { threshold }
    );
    observer.observe(el);
    return () => observer.disconnect();
  }, [threshold]);

  return { ref, isVisible };
}
```

Usage pattern:
```tsx
function FeatureSection() {
  const { ref, isVisible } = useScrollReveal();
  return (
    <div
      ref={ref}
      className={`transition-all duration-700 ease-out ${
        isVisible ? "opacity-100 translate-y-0" : "opacity-0 translate-y-8"
      }`}
    >
      {/* content */}
    </div>
  );
}
```

**Stagger children** by adding incremental delay:
```tsx
{items.map((item, i) => (
  <div
    key={item.id}
    className="transition-all duration-700 ease-out"
    style={{
      transitionDelay: isVisible ? `${i * 100}ms` : "0ms",
      opacity: isVisible ? 1 : 0,
      transform: isVisible ? "translateY(0)" : "translateY(20px)",
    }}
  >
    {item.content}
  </div>
))}
```

### Circular / Orbital Layouts

Position elements in a circle using the rotate-wrapper pattern:

```tsx
// Each item is placed by rotating a zero-size wrapper at center,
// then the item is offset by the radius and counter-rotated to stay upright
{items.map((item, i) => {
  const angle = (360 / items.length) * i - 90; // start from top
  return (
    <div
      key={item.id}
      className="absolute top-1/2 left-1/2"
      style={{ width: 0, height: 0, transform: `rotate(${angle}deg)` }}
    >
      <div
        className="absolute"
        style={{
          top: `-${radius}px`,
          transform: `rotate(${-angle}deg) translate(-50%, -50%)`,
        }}
      >
        {item.content}
      </div>
    </div>
  );
})}
```

### Infinite Carousels

Triple the content array, animate leftward by -33.333%, pause on hover:

```tsx
const tripled = [...items, ...items, ...items];

<div className="overflow-hidden">
  <div className="flex w-max gap-4 animate-[scroll-left_30s_linear_infinite] hover:[animation-play-state:paused]">
    {tripled.map((item, i) => (
      <div key={`${item.id}-${i}`} className="shrink-0">
        {/* card content */}
      </div>
    ))}
  </div>
</div>
```

### Parallax Layering

Create depth by moving layers at different speeds on scroll:

```tsx
// src/hooks/useParallax.ts
"use client";
import { useEffect, useState } from "react";

export function useParallax(speed = 0.3) {
  const [offset, setOffset] = useState(0);
  useEffect(() => {
    const onScroll = () => setOffset(window.scrollY * speed);
    window.addEventListener("scroll", onScroll, { passive: true });
    return () => window.removeEventListener("scroll", onScroll);
  }, [speed]);
  return offset;
}
```

Apply different speeds to different layers:
```tsx
function HeroSection() {
  const bgOffset = useParallax(0.2);
  const midOffset = useParallax(0.5);
  return (
    <div className="relative h-screen overflow-hidden">
      <div style={{ transform: `translateY(${bgOffset}px)` }}>
        {/* slow background layer */}
      </div>
      <div style={{ transform: `translateY(${midOffset}px)` }}>
        {/* medium foreground layer */}
      </div>
      <div>{/* static content layer */}</div>
    </div>
  );
}
```

### Container Query Scaling

For components that need to scale proportionally (like the astrolabe we built), use container queries:

```tsx
<section style={{ containerType: "inline-size" }}>
  {/* Children can use cqi units — 1cqi = 1% of container inline size */}
  <div style={{ width: "50cqi", height: "50cqi" }}>
    {/* scales with parent */}
  </div>
</section>
```

This is better than viewport units for components that might be embedded at different sizes.

## Design Token Strategy

Every project needs a purposeful color palette. Define tokens in `globals.css`:

```css
:root {
  /* Don't use generic names like --primary. Name them for what they ARE. */
  --color-surface: /* the main background */;
  --color-text: /* primary text */;
  --color-accent: /* the attention-grabbing color */;
  --color-accent-muted: /* softer version for backgrounds */;
  --color-glow: /* for highlights, hover states */;
}
```

### Color Mood Reference

| Mood | Palette Direction | Animation Vibe |
|------|-------------------|----------------|
| Luxury | Gold/cream on deep navy/black | Slow, elegant, breathing |
| Tech/Modern | Electric blue/cyan on dark | Sharp, precise, glitch-inspired |
| Natural/Organic | Earth tones, greens | Flowing, growth, unfurling |
| Playful | Bold primaries, high saturation | Bouncy, springy, fast |
| Minimal/Clean | Near-monochromes, one accent | Subtle, precise, restrained |
| Dark/Dramatic | Deep purples, crimsons on black | Theatrical, reveal-based |

## Responsive Animation Strategy

Animations must adapt to viewport — not just disappear on mobile.

**Rules:**
1. **Never remove animations on mobile** — simplify them instead
2. **Scale down, don't switch off** — a rotating element should still rotate, just smaller
3. **Reduce duration on mobile** — smaller screens mean less distance to travel
4. **Use `aspect-ratio` for complex layouts** — they scale naturally
5. **`max-width` + `width: 100%`** — constrained but fluid
6. **Replace absolute pixel values with percentages or container units**

```css
/* Desktop: full parallax experience */
@media (min-width: 1024px) {
  .hero-layer { animation-duration: 180s; }
}
/* Mobile: same animation, faster cycle */
@media (max-width: 1023px) {
  .hero-layer { animation-duration: 60s; }
}
```

## Build Sequence

When creating an animated website, follow this order:

1. **Spatial concept** — decide the metaphor and mood
2. **Color tokens** — define the palette in globals.css
3. **Typography** — load fonts, set the type scale
4. **Background composition** — the base layer that sets the atmosphere
5. **Hero section** — the first thing users see, most animation-rich
6. **Section-by-section** — build top to bottom, each with its reveal
7. **Micro-interactions** — hover states, click feedback, focus rings
8. **Responsive pass** — verify every animation works at 390px, 768px, 1440px
9. **Performance audit** — check for layout thrashing, excessive repaints

## What NOT to Do

These produce the "AI website" look everyone recognizes and nobody wants:

- **Gradient blobs floating behind cards** — overused to death since 2022
- **All sections centered with the same layout** — vary your composition
- **Fade-up as the only animation** — if everything fades up, nothing stands out
- **Using animation libraries for simple transitions** — Framer Motion for a hover opacity change is overhead for nothing
- **Rainbow or gradient text on everything** — one gradient text element per page maximum
- **Identical card grids** — if you have cards, vary their sizes or stagger them
- **Dark mode with neon accents and nothing else** — add texture, depth, real imagery
- **Animations that play once and die** — ambient motion keeps the page alive
- **Ignoring timing** — all animations at 300ms ease feels robotic; vary your timing
