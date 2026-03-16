# Phase 2 — Types & Utilities

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 2 of 7.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | §2 Card System, §9 Data Structures (types + reducer actions) |
| [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) | Timing Constants table |
| [`STATE_DIAGRAM.md`](./STATE_DIAGRAM.md) | All states listed |

---

## Already Exists (from Phase 1)

```
src/
├── app/
│   ├── layout.tsx    ← dark theme, Inter + JetBrains Mono fonts, meta tags
│   ├── page.tsx      ← default Next.js page (will be replaced in Phase 6)
│   └── globals.css   ← dark base colors
├── components/       ← empty
├── hooks/            ← empty
├── lib/              ← empty
└── types/            ← empty
```

---

## Steps

### 1. Create `src/types/index.ts`

Define all TypeScript interfaces exactly as specified in [`PROMPT.md` §9](./PROMPT.md). Key types:

```typescript
type Suit = 'H' | 'D' | 'C' | 'S';

interface Card {
  suit: Suit;
  rank: number;        // 1–13
  label: string;       // e.g., "H7", "S13"
  displayName: string; // e.g., "7♥", "K♠"
}

type NodeState = 'normal' | 'comparing' | 'swapping' | 'sorted';
type AnimationState = 'idle' | 'shuffling' | 'building' | 'heapifying' | 'extracting' | 'complete';
type Position = { x: number; y: number; level: number; index: number };

interface HeapState { ... }
interface AppState { ... }  // includes isPaused: boolean
interface StepInfo { ... }
type AppAction = ...        // 13-variant discriminated union
```

Copy the full definitions from PROMPT.md §9. Do not improvise.

---

### 2. Create `src/lib/constants.ts`

```typescript
// Suit metadata
export const SUIT_SYMBOLS: Record<Suit, string> = { H: '♥', D: '♦', C: '♣', S: '♠' };
export const SUIT_COLORS: Record<Suit, string> = { H: '#E53E3E', D: '#E53E3E', C: '#E0E0E0', S: '#E0E0E0' };
export const SUIT_NAMES: Record<Suit, string> = { H: 'Hearts', D: 'Diamonds', C: 'Clubs', S: 'Spades' };

// Animation timing (base values at 1× speed, divide by speed multiplier)
export const TIMING = {
  swap: 200,
  compareHighlight: 150,  // in + out
  compareHold: 100,
  extraction: 300,
  shuffle: 300,
  reset: 250,
  highlightOn: 100,
  highlightOff: 150,
  sortedAppear: 200,
} as const;

// Speed options
export const SPEED_OPTIONS = [0.5, 1, 2, 4] as const;

// Tree layout
export const TREE = {
  nodeWidth: 60,
  nodeHeight: 84,
  levelGap: 80,    // vertical distance between levels
  siblingGap: 20,  // minimum horizontal gap between sibling nodes
} as const;

// Visual effects
export const GLOW = {
  compare: '0 0 12px 4px rgba(255, 200, 0, 0.6)',
  swap: '0 0 16px 6px rgba(0, 220, 100, 0.6)',
  heapifyRoot: '0 0 10px 3px rgba(60, 130, 255, 0.5)',
  none: 'none',
} as const;

// Colors
export const COLORS = {
  bg: '#0A0A0A',
  cardBg: '#1A1A2E',
  text: '#E0E0E0',
  muted: '#888888',
  compareAccent: '#FFD700',
  swapAccent: '#00DC64',
  heapifyAccent: '#3C82FF',
} as const;
```

---

### 3. Create `src/lib/cards.ts`

Implement these utility functions:

- **`getDisplayRank(rank: number): string`** — Maps: 1→"A", 2–10→string number, 11→"J", 12→"Q", 13→"K"
- **`createCard(suit: Suit, rank: number): Card`** — Builds label (`"H7"`) and displayName (`"7♥"`) using `SUIT_SYMBOLS`
- **`createDeck(): Card[]`** — Returns 52 cards (4 suits × 13 ranks)
- **`groupBySuit(cards: Card[]): Record<Suit, Card[]>`** — Groups cards by suit
- **`shuffleArray<T>(arr: T[]): T[]`** — Fisher-Yates shuffle (returns new array, does not mutate)
- **`createInitialHeaps(): HeapState[]`** — Creates 4 HeapStates with ordered cards (A→K per suit), `heapSize: 13`, empty `sortedCards`

---

### 4. Create `src/lib/heap.ts`

Implement heap sort as **generator functions** that yield `StepInfo` objects:

```typescript
function* heapify(heap: Card[], n: number, i: number, heapIndex: number): Generator<StepInfo>
```
- Compare parent at `i` with children at `2i+1` and `2i+2`
- Yield `{ type: 'compare', ... }` for each comparison
- If swap needed: yield `{ type: 'swap', ... }`, perform the swap in the array, recurse

```typescript
function* buildMaxHeap(heap: Card[], heapIndex: number): Generator<StepInfo>
```
- Iterate from `Math.floor(n/2) - 1` down to 0
- Yield all steps from `heapify()` for each node

```typescript
function* heapSort(heap: Card[], heapIndex: number): Generator<StepInfo>
```
- Call `buildMaxHeap()`
- For each extraction: yield `{ type: 'swap', ... }` (root ↔ last), then yield all from `heapify()` on reduced heap
- Yield `{ type: 'extract', ... }` after each root swap

**Important:** Each `StepInfo.description` must be human-readable, e.g.:
- `"Comparing H9 with H13 — H13 is larger"`
- `"Swapping H5 and H12"`
- `"Extracting H13 to sorted area"`

---

## ✅ Done when

- [ ] `src/types/index.ts` exports all types (`Suit`, `Card`, `HeapState`, `AppState`, `StepInfo`, `AppAction`, `NodeState`, `AnimationState`, `Position`)
- [ ] `src/lib/constants.ts` exports `SUIT_SYMBOLS`, `SUIT_COLORS`, `TIMING`, `SPEED_OPTIONS`, `TREE`, `GLOW`, `COLORS`
- [ ] `src/lib/cards.ts` — `createDeck()` returns 52 cards, `shuffleArray()` works, `createInitialHeaps()` returns 4 heaps of 13
- [ ] `src/lib/heap.ts` — `heapSort()` generator yields correct steps (test by collecting all steps into an array and verifying the heap is sorted after applying all swaps)
- [ ] `npm run dev` still compiles without errors
- [ ] No unused type warnings

**Quick verification:** Add a temporary `console.log` in `page.tsx` that runs:
```typescript
const heaps = createInitialHeaps();
const steps = [...heapSort([...heaps[0].nodes], 0)];
console.log(`Total steps for Hearts: ${steps.length}`);
```
Expected: a reasonable number of steps (roughly 50–100 for 13 elements).

---

**Next phase:** [Phase 3 — Card & Tree Components](./Phase_3_components.md)
