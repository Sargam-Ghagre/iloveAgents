# Agent Collections: Proposed Feature Architecture

## Goals

Agent Collections should let users create personal named groups of agents while keeping them separate from Favorites and maintainer-curated Suites. The first implementation should be browser-local only and persisted in localStorage.

Acceptance limits:

- Maximum **10 collections**.
- Maximum **15 agents per collection**.
- Supabase sync is a future stretch goal only.

## Data model

Recommended persisted shape:

```js
{
  version: 1,
  collections: [
    {
      id: 'col_...',
      name: 'Writing Stack',
      agentIds: ['tone-rewriter', 'blog-post-seo-optimizer'],
      createdAt: '2026-06-22T00:00:00.000Z',
      updatedAt: '2026-06-22T00:00:00.000Z'
    }
  ]
}
```

Why this shape:

- `version` allows future migrations.
- `id` decouples URL identity from mutable names.
- `name` supports user-defined labels.
- `agentIds` mirrors existing favorites/suites/workflows patterns that reference agents by ID.
- Timestamps make sorting and future sync conflict handling easier.

## localStorage key

Recommended key: `ila_agent_collections_v1`.

Rationale:

- Uses the existing `ila_` prefix from `ila_favorites` and `ila_theme`.
- Includes feature name and version.
- Avoids collisions with `recentAgents` and `iloveAgents_history`.

## Constants and validation

Create a small constants module, likely `src/lib/agentCollections/constants.js`, with:

- `COLLECTIONS_STORAGE_KEY = 'ila_agent_collections_v1'`
- `MAX_COLLECTIONS = 10`
- `MAX_AGENTS_PER_COLLECTION = 15`
- `COLLECTION_NAME_MAX_LENGTH` if the UI needs a reasonable name cap.

Validation rules should live in the collection service/hook, not just the UI:

- Trim names before saving.
- Reject empty names.
- Decide whether duplicate names are allowed. Recommended: reject duplicates case-insensitively after trimming because sidebar labels become ambiguous.
- Reject creating the 11th collection.
- Reject adding the 16th unique agent to a collection.
- Prevent duplicate agent IDs in one collection.
- Ignore or surface errors for unknown agent IDs. Persisting unknown IDs is risky because renamed/removed agents could leave dead links; rendering pages can filter unknown IDs against the current registry.

## Collection service/store design

Recommended structure:

- `src/lib/agentCollections/storage.js`
  - `loadCollectionsState()`
  - `saveCollectionsState(state)`
  - `normalizeCollectionsState(raw)` for corrupted/missing data and future migrations.
- `src/lib/agentCollections/useAgentCollections.js`
  - Hook API modeled after `useFavorites.js`, with a module-level listener set for cross-component synchronization.
  - Returns state and actions:
    - `collections`
    - `createCollection(name)`
    - `deleteCollection(collectionId)`
    - `renameCollection(collectionId, name)`
    - `addAgentToCollection(collectionId, agentId)`
    - `removeAgentFromCollection(collectionId, agentId)`
    - `isAgentInCollection(collectionId, agentId)`
    - `getCollectionsForAgent(agentId)`
  - Actions should return a structured result such as `{ ok: true, collection }` or `{ ok: false, error }` so UI can display validation messages without throwing.

This mirrors Favorites while keeping data and behavior separate. Do not overload `useFavorites()` or `suitesData.js`.

## UI components needed

Recommended new components:

- `src/components/collections/CreateCollectionForm.jsx`
  - Name input, create button, validation message, disabled state at 10 collections.
- `src/components/collections/CollectionCard.jsx`
  - Displays collection name, count, sample agents, actions.
- `src/components/collections/CollectionAgentPicker.jsx`
  - Lets users add/remove agents from a specific collection from the full registry.
- `src/components/collections/AgentCollectionMenu.jsx`
  - Optional compact menu on `AgentCard` or `AgentPage` to add/remove the current agent from collections.
- `src/components/collections/CollectionSidebarSection.jsx`
  - Displays a “Collections” heading and collection links in `Sidebar.jsx`.

Keep styling consistent with existing cards: rounded borders, `dark:bg-surface-card`, `dark:border-border`, small text sizes, badges, and Lucide icons.

## Route/page changes needed

Recommended routes in `src/App.jsx` under `MainLayout`:

- `/collections` → `src/pages/CollectionsPage.jsx`
- `/collections/:collectionId` → `src/pages/CollectionDetailPage.jsx` if a detail route is desired.

If the initial UI can manage everything on one page, `/collections` can be enough. However, the acceptance criterion says collections appear in the sidebar, and individual collection sidebar links are more useful if `/collections/:collectionId` exists.

Recommended pages:

- `src/pages/CollectionsPage.jsx`
  - Overview of all collections, create form, empty state, max-limit messaging.
- `src/pages/CollectionDetailPage.jsx`
  - Single collection, rename/delete actions, agent grid, add/remove controls, empty collection state.

## Sidebar integration plan

Modify `src/components/Sidebar.jsx` later to:

1. Import `useAgentCollections()` and maybe `Icons.FolderHeart`/`Icons.Folder`.
2. Add a top-level `/collections` link or section header below Suites.
3. Render a “Collections” section only when collections exist, or render a small “Create collection” link for discoverability.
4. Render each collection as a `NavLink` to `/collections/:collectionId` with a count badge.
5. Keep collection entries outside the category reducer and outside the existing agent search results.

Search behavior assumption: the sidebar’s current search is agent-only. Collections should not be filtered by that search unless explicitly designed later.

## Add/remove agents interaction

Recommended UX paths:

- From collection detail page: add agents through a searchable picker and remove from the collection’s agent grid/list.
- From `AgentCard.jsx`: add an optional collections button/menu next to the favorite star. This is convenient but higher risk because `AgentCard` is used throughout favorites, recent, search, and future collection pages.
- From `AgentPage.jsx`/`AgentRunner`: optional action near the agent title. This may require touching runner UI and should be considered after the collection page works.

For a minimal acceptance implementation, collection detail page add/remove controls are enough.

## Error and validation handling

- Use inline messages near forms and buttons.
- Do not rely on `alert()`.
- Disable create when max collections is reached.
- Disable add buttons when a collection already contains the agent or is at 15 agents.
- Handle corrupted localStorage by falling back to an empty state and overwriting on next save.
- Handle localStorage quota errors by returning a failed action result and showing a generic persistence error.
- Filter stale agent IDs during rendering so deleted/renamed agent definitions do not break pages.

## Separation from Suites and Favorites

- Collections should use their own localStorage key and own hook/service.
- Do not import or mutate `suites` in `src/suites/suitesData.js`.
- Do not store collections inside `ila_favorites`.
- Favorites remain a single ordered list of agent IDs; collections are multiple named lists.
- Suites remain static curated objects with quizzes; collections are user-created and do not have suite quizzes or colors by default.

## Future Supabase extensibility

Prepare for sync without implementing it:

- Keep `storage.js` as an adapter boundary so a future `supabaseStorage.js` can implement the same load/save contract.
- Include `id`, `createdAt`, `updatedAt`, and `version` now.
- Keep action APIs asynchronous-ready if desired. Even if initial actions are sync, avoid UI assumptions that saving is always instant.
- Avoid Supabase imports in collection code for Phase 1.
- Future database shape could be either a `collections` table plus `collection_agents` join table or a JSONB `agent_ids` column. The local model should not depend on either choice yet.
