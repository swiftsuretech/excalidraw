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
| 2026-03-23 | Local dev environment setup | `docs/nginx/`, `docs/excalidraw`, `docs/com.excalidraw.local.plist` | Excalidraw served at `excalidraw.local` via nginx + launchd |
| 2026-03-23 | Added `@rollup/rollup-darwin-arm64` | `package.json`, `yarn.lock` | Required for Vite builds on macOS ARM when using `--ignore-optional` |

## Local Development Setup

Excalidraw runs permanently at **http://excalidraw.local** (port 80).

```bash
excalidraw status   # Check current mode
excalidraw dev      # Switch to dev mode (Vite HMR)
excalidraw prod     # Rebuild and switch to production mode
excalidraw stop     # Stop all services
```

See `docs/superpowers/specs/2026-03-23-local-excalidraw-service-design.md` for full architecture.

## Lessons Learned

### yarn install with `--ignore-optional` breaks rollup native binaries

`yarn install --ignore-optional` skips `@rollup/rollup-darwin-arm64`, which Vite needs for builds. Fix: explicitly add it as a devDependency with `yarn add -W --dev @rollup/rollup-darwin-arm64`.

### nginx worker process permissions on macOS

When nginx runs as root (for port 80), the worker process drops to user `nobody`, which can't read files in your home directory. Fix: add `user dwhitehouse staff;` to `nginx.conf`.

### yarn 1 fetches ALL platform binaries regardless of OS

The lockfile contains ~97 platform-specific packages (esbuild, rollup, @next/swc) for every OS/arch. Yarn 1 downloads all of them sequentially. On slow connections this takes 30+ minutes. Using `--ignore-optional` helps but requires manually adding back the ones you actually need.
