# State Diagram — 52 Pickup Heap Sort Visualizer

## Animation State Machine

```
                    ┌──────────────────────┐
                    │                      │
        ┌───────────▶       IDLE           ◀──────────────┐
        │           │                      │              │
        │           └──┬───────────┬───────┘              │
        │              │           │                      │
        │         [Sort]      [Randomize]                 │
        │              │           │                      │
        │              │           ▼                      │
        │              │    ┌──────────────┐              │
        │              │    │  SHUFFLING   │──(300ms)──┐  │
        │              │    └──────────────┘           │  │
        │              │                        (back to IDLE)
        │              ▼                               │  │
        │           ┌──────────────────┐                │  │
        │           │                  │                │  │
   [Reset]         │   BUILDING HEAP  │──[Reset]───────┼──┤
        │           │                  │                │  │
        │           └────────┬─────────┘                │  │
        │                    │                          │  │
        │           (build complete)                    │  │
        │                    │                          │  │
        │                    ▼                          │  │
        │           ┌──────────────────┐                │  │
        │           │                  │                │  │
        ├───────────│    HEAPIFYING    │──[Reset]───────┼──┤
        │           │                  │                │  │
        │           └────────┬─────────┘                │  │
        │                    │                          │  │
        │            (heap property                     │  │
        │             restored)                         │  │
        │                    │                          │  │
        │                    ▼                          │  │
        │           ┌──────────────────┐                │  │
        │           │                  │                │  │
        ├───────────│   EXTRACTING     │──[Reset]───────┼──┤
        │           │                  │                │  │
        │           └────────┬─────────┘                │  │
        │                    │                          │  │
        │              (heap empty?)                    │  │
        │              /           \                    │  │
        │           NO              YES                 │  │
        │            │               │                  │  │
        │            ▼               ▼                  │  │
        │    ┌──────────────┐ ┌──────────────┐          │  │
        │    │  HEAPIFYING  │ │   COMPLETE   │──────────┼──┘
        │    │  (loop back) │ │              │          │
        │    └──────────────┘ └──────┬───────┘          │
        │                           │                  │
        │                   [Reset] or [Randomize]     │
        └───────────────────────────┘──────────────────┘
```

---

## State Descriptions

| State | Description | Active Visual Cues | Allowed Actions |
|---|---|---|---|
| **IDLE** | No animation running. Cards in initial or shuffled positions. | None | Randomize, Sort, Reset |
| **BUILDING HEAP** | `buildMaxHeap()` is running. Calling `heapify` bottom-up from last non-leaf node. | Blue highlight on current subtree root. Yellow glow on compared nodes. | Pause, Step, Reset |
| **HEAPIFYING** | `heapify()` is running on a specific subtree. Comparing parent with children, swapping if needed. | Yellow glow on compared pair. Green pulse on swapping pair. Blue highlight on subtree root. | Pause, Step, Reset |
| **EXTRACTING** | Swapping root with last element, then shrinking heap. | Root card lifts + flies to sorted area. Replacement card moves to root position. | Pause, Step, Reset |
| **COMPLETE** | All cards sorted. Heaps are empty, sorted areas are full. | All cards in sorted area. Victory animation (optional shimmer). | Reset, Randomize |

---

## Pause Sub-State

Pause can occur during any active state (BUILDING, HEAPIFYING, EXTRACTING):

```
┌──────────────┐   [Pause]   ┌──────────────┐
│    ACTIVE    │────────────▶│    PAUSED    │
│  (any state) │             │  (same state) │
│              │◀────────────│              │
└──────────────┘   [Play]    └──────────────┘
                              │
                         [Step] → advance one
                              operation, remain
                              PAUSED
```

---

## Shuffle State

```
┌──────────┐   [Randomize]   ┌──────────────┐           ┌──────────┐
│   IDLE   │────────────────▶│  SHUFFLING   │──────────▶│   IDLE   │
│          │                 │  (300ms anim) │           │ (new pos)│
└──────────┘                 └──────────────┘           └──────────┘

┌──────────┐   [Randomize]   ┌──────────────┐           ┌──────────┐
│ COMPLETE │────────────────▶│  SHUFFLING   │──────────▶│   IDLE   │
│          │                 │  (300ms anim) │           │ (new pos)│
└──────────┘                 └──────────────┘           └──────────┘
```

---

## Reset Behavior

Reset can be triggered from **any** state:

```
[Any State] ──[Reset]──▶ Cancel current animation
                         → Animate cards back to ordered positions (A→K per suit)
                         → Transition to IDLE
                         → Clear sorted areas
```

---

## Control Button Availability Matrix

| State | Reset | Randomize | Step | Play | Pause | Speed |
|---|---|---|---|---|---|---|
| IDLE | ✅ | ✅ | ❌ | ✅ (starts sort) | ❌ | ✅ |
| BUILDING | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| HEAPIFYING | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| EXTRACTING | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ |
| PAUSED | ✅ | ❌ | ✅ | ✅ | ❌ | ✅ |
| COMPLETE | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| SHUFFLING | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
