# Agent Collections: Implementation Roadmap

## Phase 1: types, constants, localStorage helpers

- **Files likely to touch**
  - Add `src/lib/agentCollections/constants.js`
  - Add `src/lib/agentCollections/storage.js`
- **What to add/change**
  - Define storage key and max limits.
  - Define the persisted state shape through JSDoc comments.
  - Implement safe load/save helpers with JSON parse protection.
  - Normalize corrupted state and dedupe agent IDs inside collections.
- **Risk level**: Low.
- **Validation steps**
  - Manually call helpers in browser devtools or temporary unit-like script if test tooling is added later.
  - Confirm no source UI imports these files yet if docs-first implementation is being staged.

## Phase 2: collection state/store/hook

- **Files likely to touch**
  - Add `src/lib/agentCollections/useAgentCollections.js`
- **What to add/change**
  - Model after `src/lib/useFavorites.js` with module-level listeners for cross-component sync.
  - Expose collection state and CRUD/add/remove actions.
  - Return structured action results for validation errors.
- **Risk level**: Medium.
- **Validation steps**
  - Mount hook in a temporary page/component during development.
  - Verify two components update after one action, matching Favorites behavior.
  - Verify localStorage persists after refresh.

## Phase 3: create/delete/rename collection UI

- **Files likely to touch**
  - Add `src/pages/CollectionsPage.jsx`
  - Add `src/components/collections/CreateCollectionForm.jsx`
  - Add `src/components/collections/CollectionCard.jsx`
  - Later update `src/App.jsx` route table.
- **What to add/change**
  - Create collection overview page.
  - Add create form with validation and max-10 disabled state.
  - Add delete and rename controls, ideally with accessible labels and non-destructive confirmation for delete.
- **Risk level**: Medium.
- **Validation steps**
  - Create, rename, delete collections.
  - Refresh and confirm persisted state.
  - Try empty and duplicate names.

## Phase 4: add/remove agents from collections

- **Files likely to touch**
  - Add `src/components/collections/CollectionAgentPicker.jsx`
  - Add or update `src/pages/CollectionDetailPage.jsx`
  - Potentially update `src/components/AgentCard.jsx` only if adding collection actions on cards.
- **What to add/change**
  - Load all agents through `loadAllAgents()` or `useAgents()`.
  - Show available agents with search/filter inside the picker.
  - Add/remove agent IDs via hook actions.
  - Prevent duplicates and enforce max 15 agents.
- **Risk level**: Medium.
- **Validation steps**
  - Add a known agent to a collection.
  - Confirm duplicate add is blocked.
  - Add 15 agents and confirm the 16th is blocked.
  - Remove and re-add an agent.

## Phase 5: `/collections` page

- **Files likely to touch**
  - `src/App.jsx`
  - `src/pages/CollectionsPage.jsx`
  - Possibly `src/lib/useDocumentTitle.js` consumers only through page calls.
- **What to add/change**
  - Add route under `MainLayout`.
  - Set document title to `Collections`.
  - Provide overview, empty state, and CTA to create the first collection.
- **Risk level**: Low.
- **Validation steps**
  - Navigate to `/collections` directly.
  - Navigate from sidebar link once Phase 7 is implemented.
  - Confirm battle routes remain full-screen and unaffected.

## Phase 6: collection detail page if routing supports it

- **Files likely to touch**
  - `src/App.jsx`
  - Add `src/pages/CollectionDetailPage.jsx`
  - Add/update collection components.
- **What to add/change**
  - Add `/collections/:collectionId` route.
  - Resolve collection by ID from hook state.
  - Resolve `agentIds` to agent objects from registry.
  - Redirect or show a not-found/empty state for missing collection IDs.
- **Risk level**: Medium.
- **Validation steps**
  - Open detail URL directly after refresh.
  - Delete a collection while on its detail page and verify graceful navigation.
  - Confirm stale/unknown agent IDs are filtered out.

## Phase 7: sidebar integration

- **Files likely to touch**
  - `src/components/Sidebar.jsx`
  - Possibly add `src/components/collections/CollectionSidebarSection.jsx`
- **What to add/change**
  - Add top-level “Collections” link/section below Suites and above agent categories.
  - Show collection links with agent counts.
  - Keep category search and active category logic unchanged.
  - Highlight `/collections` and `/collections/:collectionId` routes using `NavLink`.
- **Risk level**: Medium-high because `Sidebar.jsx` has dense UI and route-active behavior.
- **Validation steps**
  - Confirm agent category auto-expand still works on `/agent/:id`.
  - Confirm Suites link still works.
  - Confirm collection links close mobile sidebar via `onClose`.
  - Confirm sidebar still shows all agent categories.

## Phase 8: validation, empty states, edge cases

- **Files likely to touch**
  - Collection hook/storage files.
  - Collection pages/components.
- **What to add/change**
  - Polish error messages and disabled states.
  - Add empty collection state.
  - Add max-limit messages.
  - Handle malformed localStorage.
  - Decide and document duplicate-name behavior.
- **Risk level**: Medium.
- **Validation steps**
  - Manually edit localStorage to malformed JSON and reload.
  - Try maximum collections and maximum agents.
  - Try deleted/stale agent IDs.

## Phase 9: testing/manual QA

- **Files likely to touch**
  - No required source files unless adding tests.
  - Potentially add a test framework later; currently `package.json` has no test script.
- **What to add/change**
  - Run `npm run build`.
  - Run the app with `npm run dev` and execute the QA checklist.
  - If test tooling is added later, unit test storage normalization and hook actions first.
- **Risk level**: Low for manual QA; medium if introducing new test tooling.
- **Validation steps**
  - Complete `docs/agent-collections/05-qa-checklist.md`.
  - Confirm Favorites, Suites, Recent Agents, Workflows, and Battle routes are unaffected.
