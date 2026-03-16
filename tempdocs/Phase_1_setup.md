# Phase 1 — Project Setup

> **Context:** You are building "52 Pickup" — a heap sort visualizer using a 52-card playing deck, displayed as interactive binary trees. This is a Next.js + TypeScript + Framer Motion project. This is Phase 1 of 7.

---

## Spec Files to Read

Before writing any code, read these files in the project root:

| File | Sections to focus on |
|---|---|
| [`PROMPT.md`](./PROMPT.md) | §3 Tech Stack, §10 Architecture |

---

## Prerequisites

- Empty project directory (or existing with only `.md` spec files)
- Node.js 18+ installed
- npm or pnpm available

---

## Steps

### 1. Initialize Next.js

```bash
npx -y create-next-app@latest ./ --typescript --app --tailwind --eslint --src-dir --no-import-alias
```

If the directory already contains files (the spec `.md` files), the CLI may prompt — answer yes to proceed.

### 2. Install Framer Motion

```bash
npm install framer-motion
```

### 3. Create folder structure

Create the following empty directories inside `src/`:

```
src/
├── components/
├── hooks/
├── lib/
└── types/
```

Also create:
```
public/
└── sounds/       # placeholder for sound files (added later)
```

### 4. Set up base layout

Edit `src/app/layout.tsx`:
- Import Google Fonts: **Inter** (UI text) and **JetBrains Mono** (card labels)
- Set dark background by default
- Add meta tags:
  - Title: `"52 Pickup — Heap Sort Visualizer"`
  - Description: `"Interactive heap sort visualization using a standard 52-card playing deck"`
- Add a basic favicon (use a ♠ emoji or placeholder)

### 5. Set up globals.css

Edit `src/app/globals.css`:
- Dark theme base colors:
  - Background: `#0A0A0A`
  - Card surface: `#1A1A2E`
  - Primary text: `#E0E0E0`
  - Muted text: `#888888`
- Add `prefers-reduced-motion` media query (empty for now — populated in Phase 7)
- Reset default margins/padding

---

## ✅ Done when

- [ ] `npm run dev` starts without errors
- [ ] Browser shows a dark page at `localhost:3000`
- [ ] `src/components/`, `src/hooks/`, `src/lib/`, `src/types/` directories exist
- [ ] Framer Motion is in `package.json` dependencies
- [ ] Google Fonts load (check Network tab)
- [ ] `<title>` tag shows "52 Pickup — Heap Sort Visualizer"

---

**Next phase:** [Phase 2 — Types & Utilities](./Phase_2_types.md)
