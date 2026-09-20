# Animation Hooks — Copy-Paste Ready

## useScrollReveal

Triggers once when element enters viewport. Good for entrance animations.

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

## useParallax

Returns a scroll-driven offset for parallax layers.

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

## useMouseParallax

Shifts elements based on mouse position for subtle depth effect on hover.

```tsx
// src/hooks/useMouseParallax.ts
"use client";
import { useEffect, useState } from "react";

export function useMouseParallax(intensity = 10) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const onMove = (e: MouseEvent) => {
      const x = (e.clientX / window.innerWidth - 0.5) * intensity;
      const y = (e.clientY / window.innerHeight - 0.5) * intensity;
      setPos({ x, y });
    };
    window.addEventListener("mousemove", onMove, { passive: true });
    return () => window.removeEventListener("mousemove", onMove);
  }, [intensity]);
  return pos;
}
```

Usage:
```tsx
const { x, y } = useMouseParallax(20);
<div style={{ transform: `translate(${x}px, ${y}px)` }}>
  {/* moves gently with cursor */}
</div>
```

## useScrollProgress

Returns 0-1 progress of an element through the viewport.

```tsx
// src/hooks/useScrollProgress.ts
"use client";
import { useEffect, useRef, useState } from "react";

export function useScrollProgress() {
  const ref = useRef<HTMLDivElement>(null);
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    const onScroll = () => {
      const rect = el.getBoundingClientRect();
      const start = window.innerHeight;
      const end = -rect.height;
      const p = Math.max(0, Math.min(1, (start - rect.top) / (start - end)));
      setProgress(p);
    };
    window.addEventListener("scroll", onScroll, { passive: true });
    onScroll();
    return () => window.removeEventListener("scroll", onScroll);
  }, []);

  return { ref, progress };
}
```

Good for scroll-driven color transitions, progress bars, morphing shapes.

## Stagger utility

Generates staggered delays for a list of elements:

```tsx
export function staggerDelay(index: number, baseMs = 100): string {
  return `${index * baseMs}ms`;
}
```
