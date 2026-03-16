# Phase 5 — Hooks & Logic

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 5 of 7. This is the most complex phase — it wires the algorithm to the UI.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | §5 User Controls, §6 Algorithm Visualization, §8 Sound Effects, §9 Data Structures (AppAction union) |
| [`STATE_DIAGRAM.md`](./STATE_DIAGRAM.md) | Full state machine, Pause sub-state, Shuffle flow, Reset behavior |
| [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) | Timing Constants, Swap/Extraction/Shuffle/Heapify sequences, Speed Multiplier Effect |
| [`COMPONENT_TREE.md`](./COMPONENT_TREE.md) | Hooks section, State Flow |

---

## Already Exists (from Phases 1–4)

```
src/types/index.ts        ← All types, AnimationState, AppAction (13 variants)
src/lib/constants.ts      ← TIMING, SPEED_OPTIONS, GLOW, COLORS
src/lib/cards.ts          ← createInitialHeaps(), shuffleArray()
src/lib/heap.ts           ← heapify(), buildMaxHeap(), heapSort() generators → yield StepInfo
src/components/
├── PlayingCard.tsx
├── BinaryTree.tsx
├── HeapPanel.tsx
├── HeapGrid.tsx
├── ControlBar.tsx        ← buttons with enable/disable logic
├── StepInfo.tsx          ← step description display
└── SoundToggle.tsx       ← mute toggle
```

**Key types you'll use:**
```typescript
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

## Steps

### 1. Create `src/hooks/useHeapSort.ts`

Algorithm controller that pre-generates all sort steps.

**Requirements:**
- **Input:** `heaps: HeapState[]`
- **Behavior:**
  1. When called, deep-clone the heap arrays (don't mutate state)
  2. Run `heapSort()` generator for each heap, collecting all yielded `StepInfo` into a flat array
  3. For simultaneous mode: interleave steps from all 4 heaps
  4. For sequential mode: concatenate steps (all Hearts, then Diamonds, etc.)
- **Returns:**
  ```typescript
  {
    steps: StepInfo[];
    totalSteps: number;
  }
  ```
- Steps must be **pre-generated** (not lazy) so we can show "Step X of Y" counter
- The generator modifies cloned arrays; the original heap state is updated via dispatch only

---

### 2. Create `src/hooks/useAnimationController.ts`

Playback engine that consumes steps and drives the UI.

**Requirements:**
- **Input:** `steps: StepInfo[]`, `speed: number`, `dispatch: (action: AppAction) => void`
- **Returns:**
  ```typescript
  {
    play: () => void;
    pause: () => void;
    step: () => Promise<void>;   // advance exactly one step
    reset: () => void;
    currentStepIndex: number;
    isPlaying: boolean;
  }
  ```

**Step execution sequence** (for each step):
```
1. dispatch({ type: 'HIGHLIGHT', heapIndex, indices })
2. await delay(TIMING.compareHold / speed)
3. if step.type === 'swap':
     dispatch({ type: 'SWAP', heapIndex, i, j })
     await delay(TIMING.swap / speed)       ← wait for Framer Motion spring to settle
4. if step.type === 'extract':
     dispatch({ type: 'EXTRACT', heapIndex })
     await delay(TIMING.extraction / speed)
5. dispatch({ type: 'CLEAR_HIGHLIGHT' })
6. trigger sound callback
7. advance currentStepIndex
```

**Play mode:** Auto-advances through steps using a loop with delays. Uses `useRef` to store a cancellation flag.

**Pause:** Sets the cancellation flag. Loop exits. `currentStepIndex` stays where it is.

**Step:** Executes exactly one step, then stops (remains paused).

**Reset:**
1. Set cancellation flag
2. `dispatch({ type: 'RESET' })`
3. Reset `currentStepIndex` to 0

**Tab visibility:** Listen to `document.visibilitychange`. When `document.hidden` is true, auto-pause. Do **not** auto-resume on tab return.

**Important:** Use `await` pattern with `setTimeout` wrapped in a Promise — do not use `setInterval`. This ensures each step completes before the next begins.

```typescript
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));
```

---

### 3. Create `src/hooks/useSound.ts`

Web Audio API wrapper with pluggable sound files.

**Requirements:**
- **Input:** `enabled: boolean`
- **Returns:**
  ```typescript
  {
    playSwap: () => void;
    playCompare: () => void;
    playExtract: () => void;
    playComplete: () => void;
    playShuffle: () => void;
  }
  ```
- All functions are **no-ops** when:
  - `enabled` is false
  - Sound files haven't been loaded yet
  - AudioContext is not available (SSR safety)
- Use lazy initialization: create `AudioContext` on first user interaction (browser autoplay policy)
- Load sound files from `/sounds/swap.mp3`, `/sounds/compare.mp3`, etc.
  - **These files don't exist yet** — the functions should gracefully handle missing files
- Use `AudioContext.createBufferSource()` for low-latency playback
- Cache decoded `AudioBuffer` objects to avoid re-decoding

---

### 4. Create the reducer `src/lib/reducer.ts`

The central state reducer for `useReducer`.

**Requirements:**
```typescript
export function heapReducer(state: AppState, action: AppAction): AppState
```

Handle each action:

| Action | State changes |
|---|---|
| `SHUFFLE` | Shuffle each heap's nodes (Fisher-Yates), reset `heapSize` to 13, clear `sortedCards`, set `animationState: 'shuffling'` → after animation, set `'idle'` |
| `SORT` | Set `animationState: 'building'`, `isPaused: false` |
| `STEP` | No state change — handled by animation controller |
| `PLAY` | Set `isPaused: false` |
| `PAUSE` | Set `isPaused: true` |
| `RESET` | Return `initialState` (ordered heaps, idle, no step) |
| `SET_SPEED` | Update `speed` |
| `HIGHLIGHT` | Set `currentStep` with the given indices, update highlighted nodes |
| `SWAP` | Swap `nodes[i]` and `nodes[j]` in the specified heap |
| `EXTRACT` | Move last active node to `sortedCards`, decrement `heapSize` |
| `CLEAR_HIGHLIGHT` | Set `currentStep: null` |
| `SET_ANIMATION_STATE` | Update `animationState` |
| `TOGGLE_SOUND` | Toggle `soundEnabled` |

**Critical:** The reducer must return **new objects** (immutable updates). Never mutate `state.heaps` directly.

---

## ✅ Done when

- [ ] `useHeapSort()` returns an array of steps when given initial heaps
- [ ] `useAnimationController()` — calling `step()` dispatches HIGHLIGHT → delay → SWAP → CLEAR_HIGHLIGHT in sequence
- [ ] `play()` auto-advances through steps; `pause()` stops it
- [ ] Speed changes are reflected immediately in delay durations
- [ ] `reset()` cancels playback and returns to initial state
- [ ] `useSound()` returns functions without crashing (even with no audio files)
- [ ] `heapReducer` correctly handles all 13 action types
- [ ] Tab hidden → auto-pauses (test by switching tabs during play)

**Quick verification:** In `page.tsx`, wire up `useReducer(heapReducer, initialState)` + `useHeapSort` + `useAnimationController`. Click "Sort" and confirm cards visually swap positions in the tree. If they move, Phase 5 is working.

---

**Next phase:** [Phase 6 — Page Assembly](./Phase_6_assembly.md)
