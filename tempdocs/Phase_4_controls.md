# Phase 4 — Controls & Info

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck displayed as binary trees. Next.js + TypeScript + Framer Motion. This is Phase 4 of 7.

---

## Spec Files to Read

| File | Sections to focus on |
|---|---|
| [`WIREFRAMES.md`](./WIREFRAMES.md) | Control Bar states (IDLE, PLAYING, PAUSED, COMPLETE) |
| [`COMPONENT_TREE.md`](./COMPONENT_TREE.md) | ControlBar, PlayPauseButton, SpeedSlider, StepInfo, SoundToggle |
| [`STATE_DIAGRAM.md`](./STATE_DIAGRAM.md) | Control Button Availability Matrix |

---

## Already Exists (from Phases 1–3)

```
src/types/index.ts       ← All types including AnimationState, AppAction
src/lib/constants.ts     ← SPEED_OPTIONS, COLORS
src/components/
├── PlayingCard.tsx       ← card rendering with visual states
├── TreeEdges.tsx         ← SVG connector lines
├── TreeNode.tsx          ← positioned card wrapper
├── BinaryTree.tsx        ← tree layout with position math
├── SortedArea.tsx        ← extracted cards row
├── HeapPanel.tsx         ← suit area (tree + sorted)
└── HeapGrid.tsx          ← 2×2 responsive grid
```

**Key types you'll use:**
```typescript
type AnimationState = 'idle' | 'shuffling' | 'building' | 'heapifying' | 'extracting' | 'complete';
interface StepInfo { type: 'compare' | 'swap' | 'extract'; description: string; ... }
```

---

## Steps

### 1. Create `src/components/ControlBar.tsx`

Container for all user controls.

**Requirements:**
- **Props:**
  ```typescript
  {
    animationState: AnimationState;
    isPaused: boolean;
    speed: number;
    onReset: () => void;
    onRandomize: () => void;
    onStep: () => void;
    onPlay: () => void;
    onPause: () => void;
    onSpeedChange: (speed: number) => void;
  }
  ```
- Renders in order: ResetButton, RandomizeButton, StepButton, PlayPauseButton, SpeedSlider
- **Button enable/disable logic** — follow this matrix exactly:

| State | Reset | Randomize | Step | Play | Pause |
|---|---|---|---|---|---|
| idle | ✅ | ✅ | ❌ | ✅ | ❌ |
| shuffling | ❌ | ❌ | ❌ | ❌ | ❌ |
| building (playing) | ✅ | ❌ | ❌ | ❌ | ✅ |
| building (paused) | ✅ | ❌ | ✅ | ✅ | ❌ |
| heapifying (playing) | ✅ | ❌ | ❌ | ❌ | ✅ |
| heapifying (paused) | ✅ | ❌ | ✅ | ✅ | ❌ |
| extracting (playing) | ✅ | ❌ | ❌ | ❌ | ✅ |
| extracting (paused) | ✅ | ❌ | ✅ | ✅ | ❌ |
| complete | ✅ | ✅ | ❌ | ❌ | ❌ |

- `PlayPauseButton` toggles between ▶ (play) and ⏸ (pause) based on `isPaused`
- Add debounce (200ms) on all button clicks to prevent rapid-fire
- Buttons use icon + text label: `⟲ Reset`, `🔀 Randomize`, `⏮ Step`, `▶ Play / ⏸ Pause`
- Disabled buttons: `opacity: 0.3`, `cursor: not-allowed`, `pointer-events: none`

**Styling:**
- Dark container with subtle top border
- Buttons: dark surface (#1A1A2E), rounded, hover brightens
- Horizontal flex layout with gap, centered

---

### 2. Create `SpeedSlider` (inside ControlBar or separate)

**Requirements:**
- Range input or 4 discrete stops: `[0.5, 1, 2, 4]`
- Display current speed label: `"0.5×"`, `"1×"`, `"2×"`, `"4×"`
- Default: `1×`
- Can be changed during any state (even while playing)
- Styled to match dark theme — custom track and thumb

---

### 3. Create `src/components/StepInfo.tsx`

Displays the current algorithm operation.

**Requirements:**
- **Props:** `step: StepInfo | null`, `currentIndex: number`, `totalSteps: number`
- When `step` is null: show `"Ready — click Sort or Randomize"` (muted text)
- When step exists: show `step.description` (e.g., `"Comparing H9 with H13 — H13 is larger"`)
- Show step counter: `"Step 24 of 156"`
- Use `AnimatePresence` from Framer Motion for smooth text fade transitions between steps
- Dark container between HeapGrid and ControlBar

---

### 4. Create `src/components/SoundToggle.tsx`

Mute/unmute button positioned in the header.

**Requirements:**
- **Props:** `enabled: boolean`, `onToggle: () => void`
- Shows `🔊` when enabled, `🔇` when muted
- Persists preference to `localStorage` (key: `"52pickup-sound"`)
- On mount, read from `localStorage` and dispatch toggle if needed
- Small, unobtrusive — positioned in header top-right (per wireframe)
- Hover tooltip: `"Toggle sound"`

---

## ✅ Done when

- [ ] `ControlBar` renders all 4 buttons + speed slider in `page.tsx`
- [ ] Buttons correctly enable/disable based on hardcoded `animationState` props (test by changing props)
- [ ] PlayPause button toggles icon between ▶ and ⏸
- [ ] Speed slider cycles through 0.5×, 1×, 2×, 4×
- [ ] `StepInfo` shows description text and step counter
- [ ] `StepInfo` shows "Ready" message when step is null
- [ ] `SoundToggle` toggles icon and persists to localStorage
- [ ] All controls look clean on dark theme — no visual glitches
- [ ] Disabled buttons are clearly dimmed

---

**Next phase:** [Phase 5 — Hooks & Logic](./Phase_5_hooks.md)
