# Excalidraw Modifications — Working Notes

This directory records our modifications to the Excalidraw codebase, lessons learned, and architectural notes gathered during development.

## Repository Architecture

### Monorepo Layout

```
excalidraw/
  packages/
    excalidraw/       Core React component library (@excalidraw/excalidraw)
    common/           Shared constants, utils, types (@excalidraw/common)
    element/          Element manipulation — create, update, transform (@excalidraw/element)
    math/             Geometry primitives — points, vectors, curves (@excalidraw/math)
    utils/            Export helpers, shape utils (@excalidraw/utils)
  excalidraw-app/     Web application (excalidraw.com) consuming the library
  dev-docs/           Public Docusaurus API docs (not our notes)
  examples/           Integration examples (NextJS, browser script)
  docs/               OUR working notes (this directory)
```

### Key Files

| File | Lines | Role |
|------|-------|------|
| `packages/excalidraw/components/App.tsx` | ~12,800 | Core editor — event handling, rendering orchestration, tool logic |
| `packages/excalidraw/index.tsx` | ~430 | Library entry point — exports `<Excalidraw>` component and hooks |
| `excalidraw-app/App.tsx` | ~1,000 | Web app shell — collab, firebase, theming |
| `excalidraw-app/index.tsx` | 17 | App bootstrap — ReactDOM render |
| `packages/common/src/constants.ts` | — | All shared constants (keys, colors, thresholds) |
| `packages/element/src/` | — | Element CRUD, arrows, binding, collision, drag, crop |
| `packages/math/src/` | — | Pure math: point, vector, curve, ellipse, polygon, segment |

### Package Dependencies

```
excalidraw-app
  └── @excalidraw/excalidraw
        ├── @excalidraw/common
        ├── @excalidraw/element
        │     ├── @excalidraw/common
        │     └── @excalidraw/math
        └── @excalidraw/math
```

### Build & Test

```bash
yarn start              # Dev server (excalidraw-app)
yarn test:typecheck     # TypeScript type checking
yarn test:update        # Run all tests with snapshot updates
yarn fix                # Auto-fix formatting and linting
yarn build:packages     # Build all packages (common -> math -> element -> excalidraw)
```

### State Management

- **Jotai** for editor-level state (`editor-jotai.ts`, `app-jotai.ts`)
- **App.tsx class component** holds the bulk of editor state internally
- **`appState.ts`** defines the `AppState` type — scroll position, selected tool, zoom, theme, etc.

### Rendering Pipeline

- **roughjs** for the hand-drawn aesthetic
- `packages/excalidraw/renderer/` handles canvas rendering
- `packages/excalidraw/scene/` manages element z-ordering and scene graph

## Modification Log

Record each modification below as we make changes.

| Date | Change | Files | Notes |
|------|--------|-------|-------|
| — | — | — | — |

## Lessons Learned

Gotchas, surprises, and non-obvious findings go here as we encounter them.

(None yet — we're just getting started.)
