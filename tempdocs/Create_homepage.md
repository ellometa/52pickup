# Create Homepage — 52 Pickup Heap Sort Visualizer

> **This is the index for 7 self-contained build prompts.** Feed each phase to an AI as a separate prompt to maintain fresh context.

---

## How to Use

1. Feed one phase file at a time to your AI
2. Include the referenced **spec files** in each prompt (or instruct the AI to read them)
3. **Verify the checkpoint** at the bottom of each phase before moving on
4. If a phase fails, retry that phase only — previous phases are unaffected

---

## Spec Files (shared context)

These files define the project requirements. The AI should read them before each phase:

| File | Purpose |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | Master spec — card system, tech stack, heap structure, controls, algorithm, types, architecture |
| [`WIREFRAMES.md`](./WIREFRAMES.md) | ASCII layouts for interface, heap panels, card states, responsive breakpoints |
| [`COMPONENT_TREE.md`](./COMPONENT_TREE.md) | Component hierarchy, props, hooks, state flow |
| [`STATE_DIAGRAM.md`](./STATE_DIAGRAM.md) | Animation state machine, pause logic, control availability matrix |
| [`ANIMATION_SPEC.md`](./ANIMATION_SPEC.md) | Timing, animation sequences, visual effects CSS, Framer Motion patterns |

---

## Build Phases

| # | File | What it builds | Estimated steps |
|---|---|---|---|
| 1 | [Phase_1_setup.md](./Phase_1_setup.md) | Next.js project init, folders, fonts, dark theme | 5 steps |
| 2 | [Phase_2_types.md](./Phase_2_types.md) | TypeScript types, constants, card utils, heap sort generator | 4 steps |
| 3 | [Phase_3_components.md](./Phase_3_components.md) | PlayingCard, BinaryTree, TreeEdges, HeapPanel, HeapGrid | 7 steps |
| 4 | [Phase_4_controls.md](./Phase_4_controls.md) | ControlBar, SpeedSlider, StepInfo, SoundToggle | 4 steps |
| 5 | [Phase_5_hooks.md](./Phase_5_hooks.md) | useHeapSort, useAnimationController, useSound, reducer | 4 steps |
| 6 | [Phase_6_assembly.md](./Phase_6_assembly.md) | page.tsx wiring, event handlers, sound callbacks | 4 steps |
| 7 | [Phase_7_polish.md](./Phase_7_polish.md) | Edge cases, responsive, accessibility, performance, SEO | 7 steps |

---

## Phase Dependencies

```
Phase 1 (Setup)
    ↓
Phase 2 (Types & Utils)
    ↓
Phase 3 (Components)  ←→  Phase 4 (Controls)   [can run in parallel]
    ↓                         ↓
    └──────────┬──────────────┘
               ↓
         Phase 5 (Hooks & Logic)
               ↓
         Phase 6 (Page Assembly)
               ↓
         Phase 7 (Polish)
```

> **Note:** Phases 3 and 4 are independent and can be built in either order or in parallel.

---

## Design Requirements Quick Reference

| Aspect | Value |
|---|---|
| **Theme** | Dark (#0A0A0A bg, #1A1A2E cards) |
| **Fonts** | Inter (UI), JetBrains Mono (labels) |
| **Animation** | Framer Motion, spring physics, snappy (200ms swaps) |
| **Heap type** | Max-heap per suit |
| **Cards** | 52 total, 13 per suit, ranks 1–13 (A, 2–10, J, Q, K) |
| **Key colors** | Yellow (#FFD700) compare, Green (#00DC64) swap, Blue (#3C82FF) heapify |
