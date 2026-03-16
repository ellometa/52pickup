# Phase 3 — Card & Tree Components

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 3 of 7.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`WIREFRAMES.md`](./WIREFRAMES.md) | Heap Panel Detail, Single Card Component |
| [`COMPONENT_TREE.md`](./COMPONENT_TREE.md) | PlayingCard, BinaryTree, TreeEdges, TreeNode, HeapPanel, SortedArea, HeapGrid |
| [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) | Visual Effects section (glow CSS), Framer Motion Key Props |

---

## Already Exists (from Phases 1–2)

```
src/types/index.ts      ← Suit, Card, HeapState, AppState, StepInfo, NodeState, Position
src/lib/constants.ts     ← SUIT_SYMBOLS, SUIT_COLORS, TIMING, TREE, GLOW, COLORS
src/lib/cards.ts         ← createDeck(), shuffleArray(), createInitialHeaps(), getDisplayRank()
src/lib/heap.ts          ← heapify(), buildMaxHeap(), heapSort() generators
src/app/globals.css      ← dark theme base, font imports
src/app/layout.tsx       ← Inter + JetBrains Mono, dark bg, meta tags
```

**Key types you'll use:**
```typescript
type Suit = 'H' | 'D' | 'C' | 'S';
interface Card { suit: Suit; rank: number; label: string; displayName: string; }
type NodeState = 'normal' | 'comparing' | 'swapping' | 'sorted';
type Position = { x: number; y: number; level: number; index: number };
```

---

## Steps

### 1. Create `src/components/PlayingCard.tsx`

A single playing card rendered as a Framer Motion `motion.div`.

**Requirements:**
- **Props:** `card: Card`, `state: NodeState`
- Use `layoutId={card.label}` (e.g., `"H7"`) for shared layout animations
- **Do NOT use both `layout` and `layoutId`** — use `layoutId` only (cards move between tree and sorted area, which are different parent containers)
- Card size: ~60×84px (from `TREE.nodeWidth` / `TREE.nodeHeight`)
- **Card face layout:**
  - Rank in top-left corner (small)
  - Suit symbol centered (large)
  - Rank in bottom-right corner (small, rotated 180°)
- **Colors:** Red (`#E53E3E`) for Hearts/Diamonds, light (`#E0E0E0`) for Clubs/Spades — from `SUIT_COLORS`
- **Card background:** `#1A1A2E` with subtle border `1px solid rgba(255,255,255,0.1)`
- **Font:** JetBrains Mono for rank/suit text

**Visual states** (from `ANIMATION_SPEC.md`):
```
'normal'    → no glow, scale 1.0, opacity 1.0
'comparing' → yellow glow (box-shadow: 0 0 12px 4px rgba(255, 200, 0, 0.6)), yellow border
'swapping'  → green glow (box-shadow: 0 0 16px 6px rgba(0, 220, 100, 0.6)), scale pulse 1.05
'sorted'    → opacity 0.5, grayscale(40%), scale 0.85
```

**Spring transition:** `{ type: "spring", stiffness: 500, damping: 30 }`

**Wrap with `React.memo`** for performance.

---

### 2. Create `src/components/TreeEdges.tsx`

SVG overlay that draws parent–child connector lines.

**Requirements:**
- **Props:** `positions: Position[]`, `heapSize: number`, `activeIndices: number[]`
- For each node at index `i` (0 to `heapSize - 1`):
  - If child `2i+1 < heapSize`: draw line from `positions[i]` to `positions[2i+1]`
  - If child `2i+2 < heapSize`: draw line from `positions[i]` to `positions[2i+2]`
- Lines connect from bottom-center of parent to top-center of child
- **Default edge style:** `stroke: rgba(255, 255, 255, 0.2)`, `stroke-width: 1.5px`
- **Active edge style** (when index is in `activeIndices`): `stroke: rgba(60, 130, 255, 0.8)`, `stroke-width: 3px`
- SVG should be positioned absolutely behind the tree nodes

---

### 3. Create `src/components/TreeNode.tsx`

Positioned wrapper that places a `PlayingCard` at the correct tree position.

**Requirements:**
- **Props:** `card: Card`, `position: Position`, `state: NodeState`
- Uses `absolute` positioning with `left` and `top` from `position.x` and `position.y`
- Renders `<PlayingCard card={card} state={state} />`

---

### 4. Create `src/components/BinaryTree.tsx`

Renders a heap array as a positioned binary tree.

**Requirements:**
- **Props:** `nodes: Card[]`, `heapSize: number`, `activeStep: StepInfo | null`
- **Position calculation:**
  ```
  For node at array index i (0-based):
    level = Math.floor(Math.log2(i + 1))
    positionInLevel = i - (Math.pow(2, level) - 1)
    nodesInLevel = Math.min(Math.pow(2, level), heapSize - (Math.pow(2, level) - 1))

    x = centerX + (positionInLevel - (nodesInLevel - 1) / 2) * horizontalSpacing(level)
    y = level * TREE.levelGap
  ```
  Where `horizontalSpacing` decreases per level for a compact tree
- Only render nodes where `index < heapSize` as active
- Determine `NodeState` for each card based on `activeStep`:
  - If `activeStep` and card's index is in `activeStep.indices` → `'comparing'` or `'swapping'` based on `activeStep.type`
  - Otherwise → `'normal'`
- Render `<TreeEdges>` behind `<TreeNode>` elements
- Container has `position: relative` with enough width/height to hold the tree

---

### 5. Create `src/components/SortedArea.tsx`

Horizontal row of extracted/sorted cards.

**Requirements:**
- **Props:** `cards: Card[]`
- Wraps cards in `<AnimatePresence>` from Framer Motion
- Each card enters with `initial={{ scale: 0, opacity: 0 }}`, `animate={{ scale: 0.85, opacity: 0.5 }}`
- Spring: `{ type: "spring", stiffness: 600, damping: 35 }`
- Cards are rendered with `state='sorted'` (dimmed style)
- Horizontal flex layout with small gap

---

### 6. Create `src/components/HeapPanel.tsx`

One suit's complete area: label + tree + sorted cards.

**Requirements:**
- **Props:** `heap: HeapState`, `activeStep: StepInfo | null`
- Renders:
  - **SuitLabel:** `"♥ Hearts"` with suit-colored icon + card count (e.g., `"11 in heap"`)
  - **BinaryTree:** active heap nodes
  - **SortedArea:** extracted cards
- Dark card-like container with subtle border, rounded corners
- Layout matches [`WIREFRAMES.md`](./WIREFRAMES.md) Heap Panel Detail

---

### 7. Create `src/components/HeapGrid.tsx`

Responsive grid container for 4 heap panels.

**Requirements:**
- **Props:** `heaps: HeapState[]`, `activeStep: StepInfo | null`
- CSS Grid layout:
  - Desktop (≥1200px): `grid-template-columns: 1fr 1fr` (2×2)
  - Tablet (768–1199px): `grid-template-columns: 1fr 1fr` (2×2, smaller)
  - Mobile (<768px): `grid-template-columns: 1fr` (1×4 vertical stack)
- Renders `<HeapPanel>` for each suit in order: Hearts, Diamonds, Clubs, Spades
- Gap between panels: ~16px

---

## ✅ Done when

- [ ] Import `HeapGrid` in `page.tsx` with test data: `const heaps = createInitialHeaps()`
- [ ] All 52 cards render correctly (4 suits × 13 ranks visible)
- [ ] Binary tree layout shows correct parent–child structure (root at top, children below)
- [ ] Tree edges (SVG lines) connect parent to children
- [ ] Cards show rank in corners + suit symbol centered
- [ ] Hearts/Diamonds are red, Clubs/Spades are light
- [ ] Responsive: 2×2 on desktop, stacks on mobile (test with browser resize)
- [ ] Dark theme looks clean — cards have subtle borders, no visual bugs

**Quick verification:** Temporarily hardcode `state='comparing'` on one card and `state='swapping'` on another to confirm glows render correctly. Then revert.

---

**Next phase:** [Phase 4 — Controls & Info](./Phase_4_controls.md)
