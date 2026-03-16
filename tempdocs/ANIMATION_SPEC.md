# Animation Spec — 52 Pickup Heap Sort Visualizer

## Design Philosophy

> **Snappy and satisfying.** Animations should feel instant — never sluggish. Use short durations, punchy easing, and spring physics to make every card movement feel responsive and alive.

---

## Timing Constants

All durations are **base values** at 1× speed. Divide by the speed multiplier: `actualDuration = baseDuration / speed`. Slower speeds (0.5×) double durations; faster speeds (4×) quarter them.

| Animation | Base Duration | Easing | Notes |
|---|---|---|---|
| **Card swap** | 200ms | `spring { stiffness: 500, damping: 30 }` | Two cards exchange positions simultaneously |
| **Card compare highlight** | 150ms in, hold 100ms, 150ms out | `easeInOut` | Yellow glow appears and fades |
| **Card extraction** | 300ms | `spring { stiffness: 400, damping: 25 }` | Card lifts, arcs, lands in sorted area |
| **Shuffle** | 300ms | `spring { stiffness: 300, damping: 28 }` | All 52 cards fly to new positions simultaneously |
| **Reset** | 250ms | `easeOut` | All cards return to ordered positions |
| **Highlight on** | 100ms | `easeIn` | Glow/border appears |
| **Highlight off** | 150ms | `easeOut` | Glow/border fades |
| **Sorted card appear** | 200ms | `spring { stiffness: 600, damping: 35 }` | Card pops into sorted area with slight overshoot |

---

## Animation Sequences

### Swap Animation (Card A ↔ Card B)

```
Timeline:
0ms      100ms     200ms     300ms     400ms     500ms
│         │         │         │         │         │
├─ Highlight A & B (yellow glow) ──────┤         │
│         ├─ Wait 100ms ──────┤        │         │
│         │         ├─ Swap positions (200ms spring) ──┤
│         │         │         │        ├─ Green pulse (100ms) ─┤
│         │         │         │         │  ├─ Clear highlights ─┤
│         │         │         │         │         │
```

**Framer Motion implementation outline:**
```
1. Set state: cards A & B → 'comparing'
   → PlayingCard renders yellow boxShadow via animate prop
2. After 200ms delay:
   → Dispatch SWAP action (array indices swap)
   → PlayingCard uses `layout` prop → Framer Motion auto-animates position change
   → Set state: cards A & B → 'swapping'
   → PlayingCard renders green border pulse
3. After swap animation completes (200ms spring):
   → Set state: cards A & B → 'normal'
   → Trigger onSwap sound
```

---

### Extraction Animation (Root → Sorted Area)

```
Timeline:
0ms      100ms     200ms     350ms     500ms     700ms
│         │         │         │         │         │
├─ Highlight root (blue) ────┤         │         │
│         │    ├─ Root lifts up 20px (scale 1.1) ┤
│         │         │    ├─ Root flies to sorted area (300ms arc) ──┤
│         │         │         │    ├─ Last node moves to root (200ms)─┤
│         │         │         │         │    ├─ Begin heapify on new root ─▶
```

**Card extraction motion path:**
```
     ┌────┐
     │root│ ← starts here
     └──┬─┘
        │ ↑ lift 20px (100ms)
        │
        └──────────────────────────┐
                                   │  arc motion (300ms)
                                   │  use Framer Motion
                                   │  with custom keyframes
                                   ▼
                              ┌────┐
                     sorted → │card│ ← lands here with overshoot spring
                              └────┘
```

---

### Shuffle Animation

```
All 52 cards receive new positions simultaneously.
Each card has a random delay between 0ms and 100ms (stagger effect).

Card motion:
  → position: spring { stiffness: 300, damping: 28 }
  → rotation: random(-15°, 15°) during flight, 0° on land
  → scale: 0.9 during flight, 1.0 on land

Total perceived duration: ~400ms (300ms base + 100ms stagger)

Sound: card riffle sound plays at 0ms
```

---

### Heapify Walk Animation

```
When heapify walks down the tree:

1. Highlight current node (blue)  ──────── 100ms
2. Highlight children (yellow)    ──────── 100ms  
3. Compare (hold)                 ──────── 100ms at 1× speed
4. If swap needed:
   a. Swap animation             ──────── 200ms
   b. Move highlight to swapped child ── 100ms
   c. Repeat from step 2 at new position
5. If no swap needed:
   a. Clear all highlights        ──────── 150ms
   b. Done
```

---

## Visual Effects

### Card Glow (Comparison)

```css
/* Yellow comparison glow */
box-shadow: 0 0 12px 4px rgba(255, 200, 0, 0.6);
border: 2px solid rgba(255, 200, 0, 0.8);

/* Animate in 100ms, animate out 150ms */
```

### Card Pulse (Swap)

```css
/* Green swap pulse */
box-shadow: 0 0 16px 6px rgba(0, 220, 100, 0.6);
border: 2px solid rgba(0, 220, 100, 0.8);

/* Single pulse: scale 1.0 → 1.05 → 1.0 over 150ms */
```

### Subtree Root Highlight

```css
/* Blue heapify focus */
box-shadow: 0 0 10px 3px rgba(60, 130, 255, 0.5);
border: 2px solid rgba(60, 130, 255, 0.7);
```

### Sorted/Dimmed Card

```css
/* Gray out extracted cards */
opacity: 0.5;
filter: grayscale(40%);
transform: scale(0.85);

/* Transition: 200ms ease-out */
```

### Tree Edge Highlight

```css
/* When heapify traverses an edge: */
stroke: rgba(60, 130, 255, 0.8);
stroke-width: 3px;

/* Default edge: */
stroke: rgba(255, 255, 255, 0.2);
stroke-width: 1.5px;
```

---

## Speed Multiplier Effect

| Speed | Swap | Compare Hold | Extraction | Description |
|---|---|---|---|---|
| 0.5× | 400ms | 200ms | 600ms | Slow — for studying each step |
| 1× | 200ms | 100ms | 300ms | Normal — balanced pace |
| 2× | 100ms | 50ms | 150ms | Fast — quick overview |
| 4× | 50ms | 25ms | 75ms | Turbo — near-instant |

Formula: `actualDuration = baseDuration / speedMultiplier`

---

## Framer Motion Key Props

```tsx
// Every PlayingCard uses layoutId for shared layout animations across containers:
// (cards move between BinaryTree and SortedArea — different parents — so layoutId is needed)
<motion.div
  layoutId={card.label}  // e.g., "H7" — unique across the app
  transition={{
    layout: {
      type: "spring",
      stiffness: 500,
      damping: 30,
    }
  }}
  animate={{
    boxShadow: getGlow(cardState),
    scale: getScale(cardState),
    opacity: getOpacity(cardState),
  }}
/>

// AnimatePresence for cards entering/exiting sorted area:
<AnimatePresence>
  {sortedCards.map(card => (
    <motion.div
      key={card.label}
      initial={{ scale: 0, opacity: 0 }}
      animate={{ scale: 0.85, opacity: 0.5 }}
      transition={{ type: "spring", stiffness: 600, damping: 35 }}
    />
  ))}
</AnimatePresence>
```

---

## Performance Considerations

| Concern | Mitigation |
|---|---|
| **60fps target** | Use `transform` and `opacity` only — no layout-triggering properties. Framer Motion handles this via GPU-accelerated transforms. |
| **52 simultaneous animations (shuffle)** | Stagger by 0–100ms random delay. Use `will-change: transform` on card elements. |
| **Re-renders during animation** | Use `React.memo` on `PlayingCard`. Isolate animation state from app state where possible. |
| **SVG edge re-draws** | Edges are simple `<line>` elements — lightweight. Batch position updates. |
| **Mobile performance** | Reduce shadow blur radius on mobile. Use `prefers-reduced-motion` media query to simplify animations. |
