# CLAUDE.md

## Project Structure

Excalidraw is a **monorepo** with a clear separation between the core library and the application:

| Directory | Purpose |
|-----------|---------|
| `packages/excalidraw/` | Core React component library (`@excalidraw/excalidraw`) — the editor |
| `packages/common/` | Shared constants, utils, types (`@excalidraw/common`) |
| `packages/element/` | Element manipulation — CRUD, arrows, binding, collision (`@excalidraw/element`) |
| `packages/math/` | Geometry primitives — points, vectors, curves, polygons (`@excalidraw/math`) |
| `packages/utils/` | Export helpers, shape utils (`@excalidraw/utils`) |
| `excalidraw-app/` | Web application (excalidraw.com) consuming the library |
| `dev-docs/` | Public Docusaurus API documentation |
| `docs/` | Our working notes, modification log, and lessons learned |
| `examples/` | Integration examples (NextJS, browser script) |

## Key Files

- **`packages/excalidraw/components/App.tsx`** (~12,800 lines) — the core editor class component. Event handling, rendering orchestration, tool logic. Most modifications touch this file.
- **`packages/excalidraw/index.tsx`** — library entry point, exports `<Excalidraw>` component and hooks
- **`packages/excalidraw/types.ts`** — `AppState`, `AppProps`, `ExcalidrawImperativeAPI` type definitions
- **`packages/common/src/constants.ts`** — all shared constants (keys, colors, thresholds, tool types)
- **`excalidraw-app/App.tsx`** — web app shell (collab, firebase, theming)

## Development Workflow

1. **Package Development**: Work in `packages/*` for editor features
2. **App Development**: Work in `excalidraw-app/` for app-specific features
3. **Testing**: Always run `yarn test:update` before committing
4. **Type Safety**: Use `yarn test:typecheck` to verify TypeScript

## Development Commands

```bash
yarn start              # Dev server (excalidraw-app on localhost)
yarn test:typecheck     # TypeScript type checking
yarn test:update        # Run all tests with snapshot updates
yarn test:app           # Run tests in watch mode
yarn fix                # Auto-fix formatting and linting issues
yarn build:packages     # Build all packages (order: common -> math -> element -> excalidraw)
```

## Architecture Notes

### Package System

- Uses Yarn workspaces for monorepo management
- Internal packages use path aliases (see `vitest.config.mts`)
- Build system uses esbuild for packages, Vite for the app
- TypeScript throughout with strict configuration

### State Management

- **Jotai** for editor-level state (`editor-jotai.ts`, `app-jotai.ts`)
- `App.tsx` class component holds bulk editor state internally
- `appState.ts` defines the `AppState` type — scroll, zoom, selected tool, theme, etc.

### Rendering

- **roughjs** for the hand-drawn sketch aesthetic
- `packages/excalidraw/renderer/` — canvas rendering pipeline
- `packages/excalidraw/scene/` — element z-ordering and scene graph

## Local Environment

Excalidraw runs permanently at **http://excalidraw.local** (port 80) via nginx + launchd.

```bash
excalidraw status   # Check current mode (prod/dev)
excalidraw dev      # Switch to Vite HMR (active development)
excalidraw prod     # Rebuild and switch to static production build
excalidraw stop     # Stop all services
```

- **Prod mode** (default): nginx serves `excalidraw-app/build/` statically. Starts on boot via launchd.
- **Dev mode**: nginx proxies to Vite on port 5173. Instant HMR at the same URL.
- **yarn install**: Use `--ignore-optional` to skip ~97 unused platform binaries. `@rollup/rollup-darwin-arm64` is an explicit devDependency to compensate.

## Our Working Notes

See `docs/README.md` for the full modification log, architecture deep-dives, and lessons learned.
