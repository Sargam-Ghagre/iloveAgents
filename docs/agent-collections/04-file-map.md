# Agent Collections: Practical File Map

## Existing files relevant to this feature

- `src/App.jsx`
  - Central route table and main layout. Add `/collections` and optional `/collections/:collectionId` under `MainLayout`.
- `src/main.jsx`
  - Mounts `BrowserRouter` and `AgentsProvider`. Useful context for route and agent data availability.
- `src/components/Sidebar.jsx`
  - Primary sidebar navigation. Collections must appear here in a distinct “Collections” section, separate from Suites and agent categories.
- `src/components/AgentCard.jsx`
  - Displays agent cards and currently owns the favorite star interaction. Optional future place for an “add to collection” control, but changes here affect many pages.
- `src/pages/HomePage.jsx`
  - Shows available agents, favorites, recently used agents, search/filter, and workflow CTA. Useful reference for rendering agent grids and resolving favorite IDs to agent objects.
- `src/pages/AgentPage.jsx`
  - Resolves `/agent/:id` to an agent and records `recentAgents`. Optional future place for collection actions for the current agent.
- `src/lib/useFavorites.js`
  - Best existing pattern for a localStorage-backed, cross-component-synced user preference hook.
- `src/lib/useHistory.js`
  - Good reference for localStorage parse/save error handling and max-item persistence.
- `src/lib/useAgents.jsx`
  - Existing context for loaded agents. Collection pages can use this rather than reloading all agents, but current app code also directly calls `loadAllAgents()`.
- `src/agents/registry.js`
  - Source of truth for agent IDs and agent metadata. Collections should store IDs only and resolve IDs through this registry when rendering.
- `src/suites/suitesData.js`
  - Static maintainer-curated suite definitions. Useful comparison point, but collections must not be added here.
- `src/pages/SuitesPage.jsx`
  - Route page for curated suites. Useful reference for card layout and icons; do not mix collection state into this page.
- `src/components/SuiteWizard.jsx`
  - Suite-specific recommendation UI. Not needed for collections unless a future “collection wizard” is designed separately.
- `src/hooks/useWorkflows.js`
  - Supabase-backed workflow service. Relevant only as a future sync pattern; do not use for initial local collections.
- `src/lib/supabase.js`
  - Supabase client setup. Keep unused for Phase 1 collections.
- `src/lib/useDocumentTitle.js`
  - Use in new collection pages to set document titles.

## New files recommended

- `src/lib/agentCollections/constants.js`
  - Storage key, max collections, max agents per collection, optional name length.
- `src/lib/agentCollections/storage.js`
  - LocalStorage adapter, normalization, migrations, safe parse/save.
- `src/lib/agentCollections/useAgentCollections.js`
  - Public React hook/store for collection state and actions.
- `src/pages/CollectionsPage.jsx`
  - Collection overview and create/manage UI.
- `src/pages/CollectionDetailPage.jsx`
  - Per-collection page for agent list and add/remove controls if detail routing is implemented.
- `src/components/collections/CreateCollectionForm.jsx`
  - Focused create form with inline validation.
- `src/components/collections/CollectionCard.jsx`
  - Overview card for one collection.
- `src/components/collections/CollectionAgentPicker.jsx`
  - Searchable add/remove UI for agents within a collection.
- `src/components/collections/CollectionSidebarSection.jsx`
  - Optional extracted sidebar section to keep `Sidebar.jsx` from growing too much.
- `src/components/collections/AgentCollectionMenu.jsx`
  - Optional later component for adding/removing from an agent card or agent detail page.

## Files to avoid touching unless necessary

- `src/suites/suitesData.js`
  - Collections are user-defined and should not become curated suite data.
- `src/lib/useFavorites.js`
  - Reuse its pattern, but do not merge collection logic into the favorites hook.
- `src/hooks/useWorkflows.js` and `src/lib/supabase.js`
  - Supabase sync is explicitly a future stretch goal.
- `src/agents/definitions/*.js`
  - Collections reference existing agent IDs; no agent definitions are required for the feature.
- Battle mode pages under `src/pages/BattleMode*.jsx`
  - Collections live in the main app layout and should not affect battle mode.
- Runner/output components such as `src/components/AgentRunner.jsx`, `OutputRenderer.jsx`, and `ScorecardOutput.jsx`
  - Not required for the initial acceptance criteria unless a later design adds collection actions inside the runner.

## Implementation boundary

Keep the first implementation narrow:

1. Add collection persistence and hook.
2. Add collection pages and sidebar links.
3. Add collection add/remove controls on collection pages.
4. Avoid broad agent registry, suite, favorite, workflow, or battle-mode refactors.
