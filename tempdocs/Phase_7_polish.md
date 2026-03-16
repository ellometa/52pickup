# Phase 7 — Polish

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 7 of 7. The app is functionally complete — this phase refines it to portfolio quality.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | §12 Edge Cases |
| [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) | Performance Considerations |
| [`WIREFRAMES.md`](./WIREFRAMES.md) | Responsive Breakpoints |

---

## Already Exists (from Phases 1–6)

The full app is functional:
- 52 cards rendering in 4 binary trees
- Randomize, Sort, Step, Play/Pause, Reset all work
- Visual cues (compare glow, swap pulse, sorted dim) appear
- Speed slider adjusts timing
- Sound architecture ready (no audio files yet)

---

## Steps

### 1. Edge Cases

Handle everything from [`PROMPT.md` §12](./PROMPT.md):

| Case | Implementation |
|---|---|
| **Rapid button clicks** | Already debounced in ControlBar (Phase 4). Verify all buttons have 200ms debounce. Add `pointer-events: none` on disabled buttons. |
| **Mid-sort reset** | Cancel the animation controller's async loop (set cancellation ref). Dispatch RESET. Framer Motion `layoutId` will animate cards back to ordered positions. Verify no orphaned timeouts. |
| **Animation sync** | Verify each step `await`s completion before the next. No step should overlap. Test at 4× speed — should still be sequential, just faster. |
| **Browser tab hidden** | Already handled in `useAnimationController` (Phase 5). Verify: start sort → switch tabs → return → animation is paused. |
| **Window resize** | Framer Motion `layoutId` handles repositioning. But the tree position calculation in `BinaryTree.tsx` needs to re-run on resize. Add a `resize` listener with debounce that triggers re-render. Or use a `ResizeObserver` on the tree container. |

---

### 2. Responsive Design

Follow [`WIREFRAMES.md`](./WIREFRAMES.md) responsive breakpoints:

| Breakpoint | Layout |
|---|---|
| ≥1200px | 2×2 grid, full card size (60×84px), all controls visible |
| 768–1199px | 2×2 grid, slightly smaller cards (~50×70px), controls may wrap |
| <768px | 1×4 vertical stack, compact cards (~44×62px), controls stacked vertically |

**Additional mobile considerations:**
- ControlBar: stack buttons vertically or use icon-only mode on small screens
- StepInfo: truncate long descriptions with ellipsis on mobile
- Tree: reduce `levelGap` and `siblingGap` on smaller screens
- Ensure touch targets are ≥44×44px for all buttons

---

### 3. Accessibility

| Feature | Implementation |
|---|---|
| **Keyboard navigation** | All buttons focusable via Tab. Enter/Space activates. Speed slider controlled with Arrow keys. |
| **ARIA labels** | Every button gets `aria-label`. Cards get `aria-label` with full name (e.g., "7 of Hearts"). StepInfo region: `role="status"`, `aria-live="polite"`. |
| **Reduced motion** | Detect `prefers-reduced-motion: reduce`. When active: disable all spring animations, show instant position changes, skip glow effects, keep the algorithm logic the same. |
| **Color contrast** | Verify yellow, green, blue glows are visible on dark background (WCAG AA). Card text on card background (#E0E0E0 on #1A1A2E) is ~10:1 ratio ✅. |
| **Focus indicators** | Custom focus ring on buttons: `outline: 2px solid #3C82FF`, `outline-offset: 2px`. |

---

### 4. Performance Optimization

Follow [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) performance section:

| Optimization | Implementation |
|---|---|
| **React.memo** | Already on PlayingCard (Phase 3). Also memo HeapPanel and BinaryTree. |
| **GPU acceleration** | Verify that only `transform` and `opacity` are animated. No `width`, `height`, `top`, `left` animations (these trigger layout). |
| **will-change** | Add `will-change: transform` on card elements during shuffle animation. Remove after animation completes (to avoid memory overhead). |
| **Stagger** | During shuffle, add random 0–100ms delay per card for a natural feel. |
| **Avoid re-renders** | Tree edge SVG should only re-render when `heapSize` or positions change. Use `React.memo` with custom comparison. |

---

### 5. Visual Polish

| Area | Detail |
|---|---|
| **Card shadows** | Subtle drop shadow on cards: `box-shadow: 0 2px 8px rgba(0,0,0,0.3)` |
| **Hover effects** | Cards: slight scale (1.02) on hover. Buttons: lighten background on hover. |
| **Suit label** | Each HeapPanel suit label uses suit color. Small font, uppercase. |
| **Tree edges** | Smooth anti-aliased SVG lines. Consider subtle animation when edges connect. |
| **"Complete" state** | When all heaps are sorted: subtle shimmer/glow animation on sorted areas. Optional confetti/celebration micro-animation. |
| **Loading state** | Show skeleton cards briefly while initial state calculates (if needed). |
| **Empty sorted area** | Show faint dashed border outline where sorted cards will appear. |

---

### 6. SEO & Meta

| Tag | Value |
|---|---|
| `<title>` | `"52 Pickup — Heap Sort Visualizer"` |
| `<meta description>` | `"Interactive visualization of heap sort using a 52-card playing deck. Watch cards swap, compare, and sort in real-time binary tree animations."` |
| `og:title` | Same as title |
| `og:description` | Same as meta description |
| `og:type` | `"website"` |
| `og:image` | Screenshot of the app (generate later) |
| `<link rel="icon">` | Favicon — ♠ or a card icon |

---

### 7. Final Code Cleanup

- Remove all `console.log` statements
- Remove any hardcoded test props
- Ensure all components have proper TypeScript types (no `any`)
- Ensure all exports are clean (named exports preferred)
- Add brief JSDoc comments on hooks and complex functions
- Verify `npm run build` succeeds with no warnings

---

## ✅ Done when

- [ ] Edge cases all handled (rapid clicks, mid-sort reset, tab hidden, resize)
- [ ] Responsive: 2×2 on desktop, stacks on mobile — cards resize properly
- [ ] Keyboard: Tab through all controls, Enter activates, Arrow keys on speed slider
- [ ] Screen reader: ARIA labels on all interactive elements, step description announced
- [ ] `prefers-reduced-motion`: animations disabled, positions update instantly
- [ ] Performance: 60fps during shuffle of all 52 cards (check with Chrome DevTools Performance tab)
- [ ] `npm run build` succeeds with zero errors and zero warnings
- [ ] App looks **premium** — dark, clean, modern. Not a student project — a portfolio piece.
- [ ] Meta tags and favicon present

---

**🎉 Project complete!** The heap sort visualizer is ready for deployment and portfolio presentation.
