# IDEAS — Naturally Arising Features

> Features that emerge organically from the existing design, architecture, and documentation.

---

## 🎓 Educational Enhancements

### 1. Complexity Counter
The step info bar already tracks "Step X of Y" — extend this to show a **live O(n log n) comparison/swap counter** alongside, so students can see time complexity play out in real-time.
> *Source: PROMPT.md §6 (algorithm phases) + COMPONENT_TREE.md (StepInfo component)*

### 2. Array View Toggle
The binary tree is backed by an array (`HeapState.nodes`). Add a **collapsible array view** beneath each tree that highlights the same indices being compared/swapped, showing the array ↔ tree duality.
> *Source: PROMPT.md §4 (heap stored as array), §9 (data structures)*

### 3. Heap Property Validator Badge
Show a small ✅/❌ badge on each heap panel indicating whether the current arrangement satisfies the max-heap property. Turns green after `buildMaxHeap` completes, goes red after shuffle.
> *Source: STATE_DIAGRAM.md (IDLE → BUILDING transition implies heap is invalid after shuffle)*

### 4. Algorithm Pseudocode Panel
A slide-out sidebar showing heap sort pseudocode with the **current line highlighted** as the animation runs — like a debugger stepping through code.
> *Source: PROMPT.md §6 (distinct phases), COMPONENT_TREE.md (useHeapSort yields StepInfo with type)*

---

## 🎮 Interaction & Playback

### 5. Step Backward (Undo)
The step generator produces a linear list of steps. Store a history stack to allow **stepping backward** through the sort — extremely useful for learning.
> *Source: COMPONENT_TREE.md (useAnimationController manages step consumption), STATE_DIAGRAM.md (Step advances one operation)*

### 6. Scrubber / Timeline Bar
Replace or augment the step counter with a **draggable timeline scrubber** showing progress through the sort. Users can jump to any point.
> *Source: COMPONENT_TREE.md (ProgressIndicator — step X of Y), pre-generated step array in useHeapSort*

### 7. Click-to-Swap (Manual Mode)
Let users **click two cards to manually swap** them in the tree, then validate whether the move maintains heap property. A hands-on learning mode.
> *Source: WIREFRAMES.md (card components are interactive), PROMPT.md §1 (educational tool)*

### 8. Keyboard Shortcuts
Controls already need keyboard accessibility (Tab + Enter). Add **hotkeys**: Space = Play/Pause, → = Step Forward, ← = Step Back, R = Reset, S = Shuffle.
> *Source: Create_homepage.md §Phase 7 (accessibility — keyboard navigation)*

---

## 📊 Visualizations

### 9. Comparison Heatmap
Track how many times each tree position is involved in a comparison. Render a **subtle heatmap overlay** on nodes — hotter = more comparisons. Reveals the algorithm's traversal pattern.
> *Source: ANIMATION_SPEC.md (comparison highlights already exist), PROMPT.md §6 (heapify walks the tree repeatedly)*

### 10. Sorted Area Fan / Cascade
Instead of a flat row, render the sorted area as a **cascading fan of cards** (like a poker hand spread). More visually striking for a portfolio piece.
> *Source: WIREFRAMES.md (Sorted Area is currently a flat row), Create_homepage.md (portfolio piece requirement)*

### 11. Edge Weight Labels
Show the **relationship direction** on tree edges (e.g., "parent > child" or "needs swap") during heapify. The edge infrastructure already exists in `TreeEdges`.
> *Source: COMPONENT_TREE.md (TreeEdges SVG), ANIMATION_SPEC.md (tree edge highlights turn blue during traversal)*

### 12. Mini-Map / Overview
For mobile's 1×4 vertical stack, add a **sticky mini-map** showing all four heaps' progress at a glance — which heaps are sorted, which are active.
> *Source: WIREFRAMES.md (mobile is a 1×4 stack — hard to see all heaps), STATE_DIAGRAM.md (heaps can run simultaneously)*

---

## ✨ Polish & Delight

### 13. Victory Animation
The state diagram mentions "optional shimmer" in the COMPLETE state. Implement a **full celebration**: cards cascade/rain, confetti particles, or a satisfying "deck fan" animation.
> *Source: STATE_DIAGRAM.md (COMPLETE state — "Victory animation (optional shimmer)")*

### 14. Card Flip Entrance
On first load, cards could start **face-down** and flip face-up with a staggered reveal animation — ties into the "52 Pickup" card-game theme.
> *Source: Project name "52 Pickup" (a card game where cards are scattered), ANIMATION_SPEC.md (shuffle has stagger effect)*

### 15. Suit Theme Colors
Each heap panel could have a **subtle background tint** matching its suit — warm red for Hearts, cool blue for Spades, green for Clubs, orange for Diamonds — instead of uniform dark panels.
> *Source: WIREFRAMES.md (four identical dark panels), PROMPT.md §5 (suit colors defined but only for card text)*

### 16. Dark / Light Theme Toggle
The design is dark-mode-first, but a **light theme** toggle would widen appeal. The CSS variables are already centralized in `globals.css`.
> *Source: Create_homepage.md (dark theme, globals.css), PROMPT.md §3 (CSS Modules or Tailwind)*

### 17. Sound Visualization
When sound is enabled, show a **tiny waveform or ripple** emanating from the card that triggered the sound — visual feedback for audio events.
> *Source: PROMPT.md §8 (5 distinct sound hooks), COMPONENT_TREE.md (useSound hook with per-event methods)*

---

## 🔀 Algorithm Variants

### 18. Algorithm Comparison Mode
The architecture cleanly separates algorithm logic (`lib/heap.ts`) from visualization. Swap in **other sorting algorithms** (quicksort, mergesort) using the same card UI and compare their behavior side-by-side.
> *Source: PROMPT.md §10 (heap.ts is isolated), COMPONENT_TREE.md (useHeapSort is a pluggable generator)*

### 19. Min-Heap Toggle
The current design uses max-heap. Add a toggle for **min-heap** (smallest at root) — a one-line change in the comparator, but visually demonstrates the difference.
> *Source: PROMPT.md §4 ("binary max-heaps"), Create_homepage.md (HeapType is max)*

### 20. Custom Deck Size
Let users choose a **subset of cards** (e.g., only 7 cards per suit) for simpler visualizations, or use all 13 for the full experience. The tree layout math already adapts to variable sizes.
> *Source: PROMPT.md §2 (13 cards per suit), COMPONENT_TREE.md (BinaryTree takes nodes array + heapSize)*

---

## 📱 Platform & Sharing

### 21. Shareable Snapshots
Generate a **URL with encoded state** (card positions + sort progress) so users can share a specific moment of the sort — "look at step 47 where this interesting swap happens."
> *Source: PROMPT.md §9 (AppState is fully serializable), deployment on Vercel*

### 22. Embed Mode
An `?embed=true` query param that hides the header and control bar, rendering just the heap grid — embeddable in blog posts, course slides, or documentation.
> *Source: PROMPT.md §1 (educational tool), Create_homepage.md (layout is modular)*

### 23. Export as GIF / Video
Record the sort animation and **export as a GIF** for use in presentations or README files. The step-based architecture makes frame capture straightforward.
> *Source: ANIMATION_SPEC.md (precise timing constants make frame timing deterministic)*

### 24. Narration Mode / Guided Tour
First-time visitors get a **guided walkthrough**: tooltips explain each panel, each button, and then run a demo sort with explanatory callouts. Builds on the step description system.
> *Source: COMPONENT_TREE.md (StepInfo already generates human-readable descriptions), PROMPT.md §1 (target: CS students)*
