# Phase 6 — Page Assembly

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 6 of 7. All components and hooks exist — this phase wires everything together.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`WIREFRAMES.md`](./WIREFRAMES.md) | Main Interface Layout (full page wireframe) |
| [`COMPONENT_TREE.md`](./COMPONENT_TREE.md) | Full hierarchy, State Flow section |

---

## Already Exists (from Phases 1–5)

```
src/types/index.ts
src/lib/constants.ts
src/lib/cards.ts          ← createInitialHeaps()
src/lib/heap.ts           ← heapSort() generator
src/lib/reducer.ts        ← heapReducer, handles all 13 AppAction types

src/components/
├── PlayingCard.tsx
├── TreeEdges.tsx
├── TreeNode.tsx
├── BinaryTree.tsx
├── SortedArea.tsx
├── HeapPanel.tsx
├── HeapGrid.tsx
├── ControlBar.tsx
├── StepInfo.tsx
└── SoundToggle.tsx

src/hooks/
├── useHeapSort.ts         ← pre-generates steps from heaps
├── useAnimationController.ts ← play/pause/step/reset engine
└── useSound.ts            ← Web Audio wrapper (no-op without files)
```

---

## Steps

### 1. Build `src/app/page.tsx`

This is the main page — a **client component** that orchestrates everything.

```typescript
"use client";
```

**State setup:**
```typescript
const initialState: AppState = {
  heaps: createInitialHeaps() as [HeapState, HeapState, HeapState, HeapState],
  animationState: 'idle',
  isPaused: false,
  currentStep: null,
  currentStepIndex: 0,
  totalSteps: 0,
  speed: 1,
  soundEnabled: true,
};

const [state, dispatch] = useReducer(heapReducer, initialState);
```

**Hook wiring:**
```typescript
const { steps, totalSteps } = useHeapSort(state.heaps);
const { play, pause, step, reset, currentStepIndex, isPlaying } = useAnimationController(steps, state.speed, dispatch);
const { playSwap, playCompare, playExtract, playComplete, playShuffle } = useSound(state.soundEnabled);
```

**Event handlers:**
```typescript
const handleRandomize = () => {
  dispatch({ type: 'SHUFFLE' });
  playShuffle();
};

const handleSort = () => {
  dispatch({ type: 'SORT' });
  play();
};

const handleStep = () => {
  step();
};

const handlePlay = () => {
  dispatch({ type: 'PLAY' });
  play();
};

const handlePause = () => {
  dispatch({ type: 'PAUSE' });
  pause();
};

const handleReset = () => {
  dispatch({ type: 'RESET' });
  reset();
};

const handleSpeedChange = (speed: number) => {
  dispatch({ type: 'SET_SPEED', speed });
};

const handleSoundToggle = () => {
  dispatch({ type: 'TOGGLE_SOUND' });
};
```

**Render layout** (per [`WIREFRAMES.md`](./WIREFRAMES.md) main interface):

```
┌─────────────────────────────────┐
│  Header                         │
│  ├── Title                      │
│  └── SoundToggle                │
├─────────────────────────────────┤
│  HeapGrid                       │
│  ├── HeapPanel (Hearts)         │
│  ├── HeapPanel (Diamonds)       │
│  ├── HeapPanel (Clubs)          │
│  └── HeapPanel (Spades)         │
├─────────────────────────────────┤
│  StepInfo                       │
├─────────────────────────────────┤
│  ControlBar                     │
└─────────────────────────────────┘
```

**Page styling:**
- Full viewport height: `min-h-screen`
- Dark background: `bg-[#0A0A0A]`
- Flex column layout
- HeapGrid takes remaining space (`flex-grow`)
- StepInfo and ControlBar at bottom (fixed or sticky if needed)
- Max width container: ~1400px, centered
- Padding: 16–24px

---

### 2. Create the Header

Either inline in `page.tsx` or as a `Header.tsx` component:

- Title: **"52 Pickup"** (large) + **"Heap Sort Visualizer"** (subtitle, muted)
- `SoundToggle` positioned on the right side
- Horizontal flex with `justify-between`
- Subtle bottom border or shadow to separate from content

---

### 3. Wire sound callbacks to animation controller

The animation controller should trigger sounds at the right moments. Either:
- Pass sound functions to `useAnimationController` as callbacks
- Or call sounds from the event handlers based on dispatched actions

Sound triggers:
| Event | Sound |
|---|---|
| Swap completes | `playSwap()` |
| Compare highlight | `playCompare()` |
| Card extracted | `playExtract()` |
| Sort complete (all heaps done) | `playComplete()` |
| Shuffle triggered | `playShuffle()` |

---

### 4. Handle `shuffling` animation state

When user clicks Randomize:
1. Dispatch `SHUFFLE` → reducer shuffles card arrays, sets `animationState: 'shuffling'`
2. Framer Motion `layoutId` on PlayingCard animates cards to new positions (~300ms spring)
3. After 400ms (300ms + buffer), dispatch `SET_ANIMATION_STATE` → `'idle'`

Use a `useEffect` or `setTimeout` in the handler:
```typescript
const handleRandomize = () => {
  dispatch({ type: 'SHUFFLE' });
  playShuffle();
  setTimeout(() => {
    dispatch({ type: 'SET_ANIMATION_STATE', state: 'idle' });
  }, 400);
};
```

---

## ✅ Done when

- [ ] Page loads showing all 52 cards in 4 binary trees (ordered A→K per suit)
- [ ] Clicking **Randomize** shuffles cards with fly-in animation, then returns to idle
- [ ] Clicking **Sort** starts the heap sort — cards swap positions in the tree with highlights
- [ ] Clicking **Pause** freezes the animation mid-sort
- [ ] Clicking **Step** advances exactly one operation while paused
- [ ] Clicking **Play** resumes from paused state
- [ ] Clicking **Reset** returns everything to ordered initial state
- [ ] Speed slider changes animation speed (verify 4× is noticeably faster than 1×)
- [ ] StepInfo bar shows current operation description and step counter
- [ ] SoundToggle persists to localStorage
- [ ] Control buttons correctly enable/disable per state
- [ ] No console errors

**This is the integration test point.** If all of the above works, the app is functionally complete. Phase 7 is polish only.

---

**Next phase:** [Phase 7 — Polish](./Phase_7_polish.md)
