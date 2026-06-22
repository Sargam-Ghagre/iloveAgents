# Agent Collections: Codebase Overview

## Tech stack

- **Vite + React 18** single-page app. Entry point is `src/main.jsx`, which mounts `App` inside `BrowserRouter` and `AgentsProvider`.
- **React Router v6** powers page routing through `Routes`, nested layout routes, `NavLink`, `Link`, `useNavigate`, `useParams`, and `Outlet`.
- **Tailwind CSS** is the styling system. Components use utility classes directly and share dark/light theme tokens from `src/index.css`/Tailwind config.
- **lucide-react** provides icons. Agent and suite data store icon names as strings that are resolved to Lucide components at render time.
- **Supabase client** exists for public workflows only through `src/lib/supabase.js` and `src/hooks/useWorkflows.js`. Agent Collections should not use Supabase in the initial implementation.

## Main folders

- `src/agents/`
  - `registry.js` discovers and normalizes all agent definition files.
  - `definitions/*.js` contains one default-exported agent config per agent.
  - `categories.js` contains category metadata, but the main UI mostly derives categories from loaded agent configs.
- `src/components/`
  - Reusable UI such as `Sidebar.jsx`, `Navbar.jsx`, `AgentCard.jsx`, `SuiteWizard.jsx`, `RecentRuns.jsx`, and runner/output components.
- `src/pages/`
  - Route-level pages including `HomePage.jsx`, `AgentPage.jsx`, `SuitesPage.jsx`, workflow pages, battle mode pages, and `NotFoundPage.jsx`.
- `src/lib/`
  - App hooks and utilities such as `useFavorites.js`, `useHistory.js`, `useAgents.jsx`, `useApiKey.js`, document title handling, and model helpers.
- `src/hooks/`
  - Feature hooks such as keyboard shortcuts and Supabase-backed workflows.
- `src/suites/`
  - Maintainer-curated suite definitions in `suitesData.js`.

## Routing structure

`src/App.jsx` defines two routing groups:

1. **Battle mode full-screen routes** without the normal navbar/sidebar:
   - `/battle`
   - `/battle/setup`
   - `/battle/arena`
   - `/battle/winner`
2. **Main app layout routes** rendered inside `MainLayout`, which includes `Navbar`, `Sidebar`, `CustomCursor`, and an `Outlet`:
   - `/`
   - `/agent/:id`
   - `/suites`
   - `/workflows`
   - `/workflows/build`
   - `/workflows/:id`
   - `/workflows/:id/run`
   - `*` for not found

Collections should be added under the main layout, most likely with `/collections` and, if needed, `/collections/:collectionId`.

## Agent registry and data flow

- `src/agents/registry.js` uses `import.meta.glob('./definitions/*.js', { eager: false })` for lazy loading and exports `loadAllAgents()`.
- `loadAllAgents()` caches a single promise in `cachedAgentsPromise`, imports every definition, maps default exports, and filters invalid/duplicate IDs through `normalizeAgents()`.
- The same file also exports a default eager `agents` array via `import.meta.glob(..., { eager: true })` for code that wants synchronous registry access.
- `src/lib/useAgents.jsx` wraps `loadAllAgents()` in a React context. It is mounted in `src/main.jsx`, but several existing components still call `loadAllAgents()` directly.
- `HomePage.jsx`, `Sidebar.jsx`, and `AgentPage.jsx` load agents independently with `loadAllAgents()` and store them in local state.
- Agent IDs are the stable references used across favorites, recent agents, suites, workflows, and proposed collections.

## Existing Favorites flow

- `src/lib/useFavorites.js` stores favorite agent IDs as a JSON array in localStorage under `ila_favorites`.
- The hook exposes `{ favorites, isFavorite, toggleFavorite }`.
- It uses a module-level `listeners` set and `notify()` function so multiple mounted components using the hook stay in sync after toggles.
- `toggleFavorite(agentId)` loads the latest localStorage value, adds the ID to the front or removes it, saves, updates local state, and notifies listeners.
- `src/components/AgentCard.jsx` renders a star button and calls `toggleFavorite(agent.id)` without navigating the card link.
- `src/pages/HomePage.jsx` resolves favorite IDs to loaded agent objects and renders a “Your Favorites” section only when favorites exist and the user is not searching/filtering.

## Existing Suites flow

- Suites are static maintainer-curated data in `src/suites/suitesData.js`.
- Each suite has `id`, `name`, `icon`, `description`, `color`, `agents`, and `quiz` fields.
- Suite `agents` are arrays of agent IDs. They do not store full agent objects.
- `src/pages/SuitesPage.jsx` imports `suites`, renders suite cards, and switches to `SuiteWizard` when a suite is selected.
- `src/components/SuiteWizard.jsx` uses the suite quiz to recommend agent IDs and links users to matching agents.
- The sidebar has a top-level `/suites` `NavLink`; suites are not mixed into category lists and are not user-editable.

## Sidebar/navigation structure

- `src/components/Sidebar.jsx` owns its search query, expanded category state, loaded agents, and active route detection.
- It loads all agents on mount through `loadAllAgents()`.
- It derives categories by reducing filtered agents by `agent.category`.
- The current active agent category is auto-expanded when the route starts with `/agent/`.
- It renders:
  1. Header with filtered agent count.
  2. Agent search input.
  3. A top-level “Suites” link.
  4. A divider.
  5. Collapsible category sections with agent links.
  6. Footer links.
- Collections should be a separate sidebar section, not nested under agent categories and not merged into Suites.

## Current persistence patterns

- Theme: `src/components/Navbar.jsx` uses `localStorage` key `ila_theme`.
- Favorites: `src/lib/useFavorites.js` uses `localStorage` key `ila_favorites` and manually synchronizes hook consumers with a module-level listener set.
- Recent agents: `src/pages/AgentPage.jsx` writes a JSON array to `recentAgents`; `src/pages/HomePage.jsx` reads it to show recently used agents.
- Run history: `src/lib/useHistory.js` stores up to 10 runs under `iloveAgents_history` with error handling for parse/save failures.
- API keys: `src/lib/useApiKey.js` uses `sessionStorage`, not localStorage.
- Workflows: `src/hooks/useWorkflows.js` use Supabase and are unrelated to local user collections.

## Assumptions and uncertainties

- There is no existing collections feature, route, hook, or localStorage key.
- The project is JavaScript-only; new collection types will likely be documented with JSDoc or colocated constants rather than TypeScript interfaces unless the project migrates.
- Because `useAgents.jsx` is available but not consistently used, implementation can either follow current direct `loadAllAgents()` patterns or gradually reuse the context in collection pages. Avoid broad refactors for Issue #569.
