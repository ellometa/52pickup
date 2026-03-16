# 52 Pickup — Heap Sort Visualizer

> A portfolio-grade, educational website that visualizes heap sort using a standard 52-card playing deck, rendered as an interactive binary tree.

---

## Prompt

You are a senior software architect and frontend engineer. Your task is to **fully design and then implement** a heap sort visualization website using playing cards.

---

### 1. Project Overview

| Aspect | Detail |
|---|---|
| **Purpose** | Portfolio piece + live educational tool |
| **Core concept** | Visualize heap sort using a 52-card deck, displayed as a binary tree |
| **Feel** | Snappy, responsive, premium — animations must feel instant and satisfying |
| **Target users** | CS students, hiring managers, anyone learning data structures |

---

### 2. Card System

Each card has a **suit** and a **rank** (1–13):

| Rank | Label | Display |
|------|-------|---------|
| 1 | A (Ace) | `H1`, `D1`, `C1`, `S1` |
| 2–10 | Number | `H2`…`H10`, etc. |
| 11 | J (Jack) | `H11`, `D11`, `C11`, `S11` |
| 12 | Q (Queen) | `H12`, `D12`, `C12`, `S12` |
| 13 | K (King) | `H13`, `D13`, `C13`, `S13` |

**Suits:**
- **H** = Hearts ♥
- **D** = Diamonds ♦
- **C** = Clubs ♣
- **S** = Spades ♠

Each suit has exactly **13 cards**. The full deck is **52 cards**.

Card **sort value** is determined by rank (1–13). Suit is used only for grouping into the four heaps.

---

### 3. Tech Stack

| Layer | Choice |
|---|---|
| **Framework** | Next.js (App Router) |
| **Language** | TypeScript |
| **Styling** | Tailwind CSS |
| **Animation** | See §3.1 — choose one primary library |
| **State** | React Context + `useReducer` (or Zustand if state grows complex) |
| **Sound** | Web Audio API — sound files provided later by user |
| **Deployment** | Vercel |

#### 3.1 Animation Library Recommendation

Choose **one** primary animation library. Here is the comparison:

##### Framer Motion (recommended ✅)

| | |
|---|---|
| **Pros** | Declarative React-native API (`animate`, `layout`, `AnimatePresence`). Built-in gesture support. Automatic layout animations when nodes move in the tree. Excellent Next.js compatibility. Easiest to learn. 3.5M+ weekly downloads. Spring physics built-in. |
| **Cons** | Less granular control for complex sequenced timelines. Heavier bundle than alternatives (~30KB). Not ideal for Canvas/WebGL. |
| **Best for** | UI-centric animations, layout transitions, card movements — exactly this project. |
| **AI competence** | ⭐⭐⭐⭐⭐ — Most training data, most examples, highest success rate in generated code. |

##### GSAP (GreenSock)

| | |
|---|---|
| **Pros** | Unmatched timeline control. Battle-tested performance. Can animate anything (DOM, SVG, Canvas). Rich plugin ecosystem (ScrollTrigger, MorphSVG). Pixel-perfect precision. |
| **Cons** | Imperative API — less "React-y". Steeper learning curve. Requires `useRef` + manual cleanup. Licensing considerations for commercial plugins. |
| **Best for** | Complex choreographed sequences, scroll-driven animations, SVG morphing. |
| **AI competence** | ⭐⭐⭐⭐ — Well-known but imperative style leads to more integration bugs in generated code. |

##### react-spring

| | |
|---|---|
| **Pros** | Physics-based by default — natural springy feel. Hooks-first API (`useSpring`, `useTrail`). Lightweight. No re-render overhead. |
| **Cons** | Steeper learning curve (physics model vs duration model). Smaller community. Less intuitive for sequenced, step-by-step animations. |
| **Best for** | Organic, springy UI interactions. Drag-and-drop. |
| **AI competence** | ⭐⭐⭐ — Fewer training examples, less reliable generated code. |

> **Verdict:** Use **Framer Motion** as the primary animation library. Its declarative `layout` animations are perfect for binary tree node repositioning, and its `AnimatePresence` handles card insertion/removal elegantly. If timeline sequencing becomes complex, GSAP can be added as a secondary library for specific sequences.

---

### 4. Heap Structure

- The visualization shows **four independent binary max-heaps**, one per suit.
- Each heap contains **13 cards** from its suit.
- The heap is rendered as a **binary tree** (not a flat array).
- Tree nodes show the **card face** (e.g., `H7`, `S13`).
- Parent–child relationships are drawn with **connecting lines/edges**.

Tree layout rules:
- Root at the top center of its heap area.
- Left child: bottom-left. Right child: bottom-right.
- Maximum depth for 13 nodes = 4 levels (root at level 0).
- Nodes at the same level are evenly spaced horizontally.

---

### 5. User Controls

Two primary buttons:

| Button | Behavior |
|---|---|
| **Randomize** | Randomly reorders the 13 cards within each suit's heap. Cards stay in their own suit. Cards fly to new positions. Heap property is **not** maintained after shuffle. |
| **Sort** | Runs heap sort on all four heaps simultaneously (or sequentially — user's choice via toggle). Animates the full process: build heap → heapify → extract. |

Additional controls:

| Control | Behavior |
|---|---|
| **Step** | Advance one operation at a time (one swap / one comparison). |
| **Play / Pause** | Auto-play the sort at a configurable speed. |
| **Speed slider** | Controls animation speed (0.5×, 1×, 2×, 4×). |
| **Reset** | Returns cards to initial ordered state (A–K per suit). |

---

### 6. Algorithm Visualization

The sort proceeds in clearly distinct phases:

#### Phase 1: Build Max-Heap
- Starting from the last non-leaf node, call `heapify` on each node upward.
- **Animation:** Highlight the current node being heapified. Show comparisons (glow/pulse on compared nodes). Animate swaps by smoothly moving cards between tree positions.

#### Phase 2: Heap Sort (Extract Max)
- Swap root with last unsorted element.
- Shrink the heap boundary by one.
- Call `heapify` on the new root.
- **Animation:** Root card lifts out and flies to the "sorted" area. The replacement card moves to root. Heapify animation plays on the subtree.

#### Visual cues:
| Cue | Meaning |
|---|---|
| **Yellow glow** | Currently being compared |
| **Green pulse** | Swap happening |
| **Gray/dim** | Already sorted / extracted |
| **Blue highlight** | Current heapify subtree root |

---

### 7. UI/UX Design

Refer to the companion files for detailed specifications:

- **[`WIREFRAMES.md`](./WIREFRAMES.md)** — ASCII wireframes for the main interface layout
- **[`COMPONENT_TREE.md`](./COMPONENT_TREE.md)** — Full component hierarchy
- **[`STATE_DIAGRAM.md`](./STATE_DIAGRAM.md)** — Animation state machine (idle → building → heapifying → extracting)
- **[`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md)** — Detailed animation behavior, timing, and easing

---

### 8. Sound Effects

- Sound effect files will be provided later by the user.
- The architecture should support pluggable sound hooks:
  - `onSwap` — card swap sound
  - `onCompare` — subtle tick/click
  - `onExtract` — satisfying "pop" or "whoosh"
  - `onComplete` — completion chime
  - `onShuffle` — card shuffle/riffle sound
- Sound should be toggleable (mute button).
- Use the **Web Audio API** for low-latency playback.

---

### 9. Data Structures

```typescript
type Suit = 'H' | 'D' | 'C' | 'S';

interface Card {
  suit: Suit;
  rank: number;       // 1–13
  label: string;      // e.g., "H7", "S13"
  displayName: string; // e.g., "7♥", "K♠"
}

interface HeapState {
  suit: Suit;
  nodes: Card[];          // array representation of binary heap
  sortedCards: Card[];     // cards already extracted
  heapSize: number;        // current active heap size
}

type AnimationState = 'idle' | 'shuffling' | 'building' | 'heapifying' | 'extracting' | 'complete';

interface AppState {
  heaps: [HeapState, HeapState, HeapState, HeapState];
  animationState: AnimationState;
  isPaused: boolean;       // overlay flag — pauses any active state
  currentStep: StepInfo | null;
  currentStepIndex: number;
  totalSteps: number;
  speed: number;           // multiplier: 0.5, 1, 2, 4
  soundEnabled: boolean;
}

interface StepInfo {
  type: 'compare' | 'swap' | 'extract';
  heapIndex: number;
  indices: [number, number];
  description: string;     // human-readable, e.g., "Comparing H7 with H12"
}

// Reducer action types
type AppAction =
  | { type: 'SHUFFLE' }
  | { type: 'SORT' }
  | { type: 'STEP' }
  | { type: 'PLAY' }
  | { type: 'PAUSE' }
  | { type: 'RESET' }
  | { type: 'SET_SPEED'; speed: number }
  | { type: 'HIGHLIGHT'; heapIndex: number; indices: number[] }
  | { type: 'SWAP'; heapIndex: number; i: number; j: number }
  | { type: 'EXTRACT'; heapIndex: number }
  | { type: 'CLEAR_HIGHLIGHT' }
  | { type: 'SET_ANIMATION_STATE'; state: AnimationState }
  | { type: 'TOGGLE_SOUND' };
```

---

### 10. Architecture

```
src/
├── app/
│   ├── layout.tsx            # root layout
│   ├── page.tsx              # main visualizer page
│   └── globals.css
├── components/
│   ├── PlayingCard.tsx        # single card component
│   ├── BinaryTree.tsx         # renders a heap as a binary tree
│   ├── TreeEdges.tsx          # SVG parent–child connector lines
│   ├── TreeNode.tsx           # positioned card slot wrapper
│   ├── HeapGrid.tsx           # 2×2 / 1×4 responsive grid container
│   ├── HeapPanel.tsx          # one suit's heap area (tree + sorted cards)
│   ├── SortedArea.tsx         # row of extracted cards
│   ├── ControlBar.tsx         # play/pause/step/reset/shuffle + speed
│   ├── StepInfo.tsx           # shows current operation description
│   └── SoundToggle.tsx        # mute/unmute button
├── hooks/
│   ├── useHeapSort.ts         # heap sort algorithm + step generator
│   ├── useAnimationController.ts  # manages animation playback
│   └── useSound.ts            # Web Audio API wrapper
├── lib/
│   ├── cards.ts               # card creation, deck utilities
│   ├── heap.ts                # heap operations (heapify, buildHeap, sort)
│   └── constants.ts           # colors, timing, layout constants
├── types/
│   └── index.ts               # TypeScript interfaces
└── public/
    └── sounds/                # sound effect files (added later)
```

---

### 11. Milestones

| Stage | Deliverable | Scope |
|-------|-------------|-------|
| **1** | Static card grid | Render 52 cards, grouped by suit. No tree layout yet. |
| **2** | Binary tree layout | Position cards in a binary tree structure per suit. Draw edges. |
| **3** | Randomize | Shuffle button randomizes card positions with fly-in animation. |
| **4** | Heap sort logic | Implement heap sort algorithm as a step generator (no animation). |
| **5** | Animated sort | Connect step generator to animations. Cards move between tree nodes. |
| **6** | Controls | Play/pause/step/reset/speed slider. |
| **7** | Visual cues | Comparison highlights, swap glows, sorted dimming. |
| **8** | Sound | Integrate sound hooks with Web Audio API. |
| **9** | Polish | Responsive layout, dark mode, micro-interactions, edge cases. |

---

### 12. Edge Cases

| Case | Handling |
|---|---|
| **Equal values** | Not applicable — each card in a suit has a unique rank (1–13). |
| **Rapid button clicks** | Debounce controls. Disable sort while animation is in progress. |
| **Mid-sort reset** | Cancel current animation, return all cards to initial positions smoothly. |
| **Animation sync** | Use a step queue. Each animation must `await` completion before the next step begins. |
| **Browser tab hidden** | Pause animations when `document.hidden` is true (use `visibilitychange` event). |
| **Window resize** | Recalculate tree positions. Use Framer Motion `layout` for automatic repositioning. |

---

### 13. Estimated Complexity

| Metric | Estimate |
|---|---|
| **Difficulty** | Intermediate–Advanced |
| **Development time** | 3–5 days for a polished version |
| **Lines of code** | ~2,000–3,000 |
| **Key challenge** | Synchronizing step-by-step algorithm execution with snappy animations |

---

### 14. Implementation Plan

1. `npx -y create-next-app@latest ./ --typescript --app --tailwind --eslint --src-dir --no-import-alias`
2. Define types in `types/index.ts`
3. Build `cards.ts` utility — create deck, shuffle, label formatting
4. Build `heap.ts` — heapify, buildMaxHeap, heapSort as generator functions
5. Create `PlayingCard` component with Framer Motion
6. Create `BinaryTree` component — position nodes using recursive layout math
7. Create `HeapPanel` — combine tree + sorted area per suit
8. Build `ControlBar` with all buttons + speed slider
9. Implement `useHeapSort` hook — yields steps one at a time
10. Implement `useAnimationController` — consumes steps, triggers Framer Motion
11. Wire up `useSound` — placeholder API, ready for sound files
12. Add visual cues (highlights, glows, dimming)
13. Polish: responsive design, accessibility, dark mode
14. Deploy to Vercel

---

> **Do not write full code yet.** First, review the companion specification files ([WIREFRAMES.md](./WIREFRAMES.md), [COMPONENT_TREE.md](./COMPONENT_TREE.md), [STATE_DIAGRAM.md](./STATE_DIAGRAM.md), [ANIMATION_SPEC.md](./ANIMATION_SPEC.md)) and confirm the design before proceeding to implementation.
