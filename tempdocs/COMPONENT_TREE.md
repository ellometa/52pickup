# Component Tree — 52 Pickup Heap Sort Visualizer

## Hierarchy

```
App (layout.tsx)
└── Page (page.tsx)
    ├── Header
    │   ├── Title — "52 Pickup — Heap Sort Visualizer"
    │   └── SoundToggle — 🔇/🔊 mute button
    │
    ├── HeapGrid — CSS Grid container (2×2 on desktop, 1×4 on mobile)
    │   ├── HeapPanel (suit="H") — Hearts
    │   │   ├── SuitLabel — "♥ Hearts" + card count
    │   │   ├── BinaryTree
    │   │   │   ├── TreeEdges (SVG) — lines connecting parent→child
    │   │   │   └── TreeNode[] — positioned card slots
    │   │   │       └── PlayingCard — the card face
    │   │   │           ├── RankDisplay — "7", "K", "A"
    │   │   │           ├── SuitIcon — ♥ ♦ ♣ ♠
    │   │   │           └── AnimationOverlay — glow / pulse / dim
    │   │   └── SortedArea — row of extracted cards
    │   │       └── PlayingCard[] — sorted cards (dimmed style)
    │   │
    │   ├── HeapPanel (suit="D") — Diamonds
    │   ├── HeapPanel (suit="C") — Clubs
    │   └── HeapPanel (suit="S") — Spades
    │
    ├── StepInfo — text bar describing current operation
    │   ├── StepDescription — "Comparing H9 with H13..."
    │   └── ProgressIndicator — step X of Y
    │
    └── ControlBar
        ├── ActionButtons
        │   ├── ResetButton — ⟲
        │   ├── RandomizeButton — 🔀
        │   ├── StepButton — ⏮
        │   └── PlayPauseButton — ▶ / ⏸ (toggles)
        ├── SpeedSlider — 0.5× / 1× / 2× / 4×
        └── ModeToggle — Sequential vs Simultaneous
```

---

## Component Responsibilities

### Page-Level

| Component | Responsibility |
|---|---|
| `Page` | Top-level state provider. Holds `AppState` via `useReducer`. Orchestrates all child components. |
| `Header` | Static title + sound toggle. No internal state. |

### Heap Components

| Component | Props | Responsibility |
|---|---|---|
| `HeapGrid` | `heaps: HeapState[]` | CSS Grid layout for the four heap panels. Handles responsive breakpoints. |
| `HeapPanel` | `heap: HeapState`, `activeStep: StepInfo` | Container for one suit. Shows binary tree + sorted area. |
| `BinaryTree` | `nodes: Card[]`, `heapSize: number`, `activeStep: StepInfo \| null` | Renders heap array as positioned tree. Computes x,y for each node based on depth and index. |
| `TreeEdges` | `nodes: Card[]`, `positions: Position[]` | SVG overlay drawing parent–child connector lines. |
| `TreeNode` | `card: Card`, `position: Position`, `state: NodeState` | Positioned wrapper. Passes highlight state down to PlayingCard. |
| `PlayingCard` | `card: Card`, `state: 'normal' | 'comparing' | 'swapping' | 'sorted'` | The visual card. Framer Motion `motion.div` with `layout` prop for automatic position transitions. |
| `SortedArea` | `cards: Card[]` | Horizontal row of extracted cards. Cards animate in via `AnimatePresence`. |

### Control Components

| Component | Props | Responsibility |
|---|---|---|
| `ControlBar` | `animationState`, `onAction` | Container for all controls. Disables buttons based on current state. |
| `PlayPauseButton` | `isPlaying: boolean`, `onToggle` | Toggles between ▶ and ⏸ icons. |
| `SpeedSlider` | `speed: number`, `onChange` | Range input for animation speed multiplier. |
| `StepInfo` | `step: StepInfo \| null` | Displays human-readable description of current algorithm step. |
| `SoundToggle` | `enabled: boolean`, `onToggle` | Mute/unmute. Persists preference to localStorage. |

### Hooks

| Hook | Purpose |
|---|---|
| `useHeapSort(heaps)` | Generator-based heap sort. Yields `StepInfo` objects one at a time. Pure algorithm — no UI concerns. |
| `useAnimationController(steps, speed)` | Consumes steps from the generator. Manages play/pause/step. Dispatches state updates after each animation completes. |
| `useSound(enabled)` | Returns `{ playSwap, playCompare, playExtract, playComplete, playShuffle }`. No-ops when disabled. |

---

## State Flow

```
User clicks "Randomize"
  → Page dispatches SHUFFLE action
  → HeapGrid receives new heaps
  → BinaryTree recomputes positions
  → PlayingCard animates to new position (Framer Motion layout)

User clicks "Sort"
  → useHeapSort generates step list
  → useAnimationController starts consuming steps
  → Each step:
      1. Dispatch HIGHLIGHT action → PlayingCard glows
      2. Wait (speed-adjusted duration)
      3. Dispatch SWAP/EXTRACT action → PlayingCard animates
      4. useSound fires appropriate sound
      5. Dispatch CLEAR_HIGHLIGHT → next step

User clicks "Step"
  → useAnimationController advances exactly one step
  → Same flow as above, but single iteration
```
