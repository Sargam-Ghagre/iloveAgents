# Agent Collections: Manual QA Checklist

## Setup

- [ ] Start from a clean browser profile or clear only the collections key `ila_agent_collections_v1`.
- [ ] Keep existing `ila_favorites`, `recentAgents`, and `iloveAgents_history` values available for regression checks when possible.
- [ ] Run the app with `npm run dev`.
- [ ] Open the app in both desktop and mobile-width layouts because collections must appear in the sidebar.

## Creating collections

- [ ] Open `/collections`.
- [ ] Verify the empty state explains that no collections exist yet.
- [ ] Create a collection named `My Morning Tools`.
- [ ] Verify the collection appears on the collections page.
- [ ] Verify the collection appears under the sidebar “Collections” section.
- [ ] Create a collection named `Writing Stack`.
- [ ] Verify multiple collections render in a stable order, preferably newest first or created order as intentionally designed.

## Duplicate names

- [ ] Try creating `Writing Stack` again.
- [ ] Try creating ` writing stack ` with different casing/whitespace.
- [ ] Verify the app follows the chosen rule.
  - Recommended rule: duplicate names are rejected case-insensitively after trimming.
- [ ] Verify the error message is inline and does not use `alert()`.

## Max 10 collections

- [ ] Create collections until there are 10 total.
- [ ] Verify the create control is disabled or returns a clear validation error.
- [ ] Try creating an 11th collection.
- [ ] Verify localStorage still contains only 10 collections.
- [ ] Delete one collection.
- [ ] Verify creating another collection is possible again.

## Adding agents

- [ ] Open a collection detail page or collection management UI.
- [ ] Add `tone-rewriter` or another known agent.
- [ ] Verify the agent appears in the collection.
- [ ] Verify the sidebar collection count increments.
- [ ] Refresh the page.
- [ ] Verify the agent remains in the collection.

## Duplicate agent prevention

- [ ] Attempt to add the same agent to the same collection again.
- [ ] Verify the duplicate is blocked or the add control is disabled.
- [ ] Verify the collection still contains only one copy of that agent ID in localStorage.

## Max 15 agents per collection

- [ ] Add agents until the collection contains 15 unique agents.
- [ ] Verify the UI communicates the 15-agent limit.
- [ ] Try adding a 16th agent.
- [ ] Verify the 16th agent is not persisted.
- [ ] Remove one agent.
- [ ] Verify another agent can be added afterward.

## Removing agents

- [ ] Remove an agent from a collection.
- [ ] Verify it disappears from the collection UI.
- [ ] Verify sidebar count decrements.
- [ ] Refresh the page.
- [ ] Verify the removed agent does not reappear.

## Renaming collections

- [ ] Rename `My Morning Tools` to `Daily Launchpad`.
- [ ] Verify the page title/card/sidebar label update.
- [ ] Refresh the page.
- [ ] Verify the new name persists.
- [ ] Try renaming to an empty value.
- [ ] Verify validation blocks it.

## Deleting collections

- [ ] Delete a collection with agents in it.
- [ ] Verify confirmation or clear destructive UI is shown.
- [ ] Verify the collection is removed from `/collections`.
- [ ] Verify it is removed from the sidebar.
- [ ] Refresh the page.
- [ ] Verify it remains deleted.
- [ ] If currently on `/collections/:collectionId`, verify the app navigates gracefully or shows a not-found state.

## localStorage persistence after refresh

- [ ] Create at least two collections with different agents.
- [ ] Refresh on `/collections`.
- [ ] Verify all collections and counts persist.
- [ ] Refresh on `/collections/:collectionId`.
- [ ] Verify detail content persists.
- [ ] Close and reopen the tab.
- [ ] Verify collections still persist.

## Sidebar visibility

- [ ] Verify “Collections” appears as a separate section from “Suites”.
- [ ] Verify Collections are not included in agent category counts.
- [ ] Verify agent search in the sidebar still filters only agents/categories unless intentionally changed.
- [ ] Verify clicking a collection link closes the mobile sidebar.
- [ ] Verify active route styling works for `/collections` and collection detail pages.
- [ ] Verify `/agent/:id` still auto-expands the active agent category.

## Empty states

- [ ] Clear the collections key and reload.
- [ ] Verify `/collections` shows a helpful empty state.
- [ ] Create an empty collection.
- [ ] Verify the detail page explains how to add agents.
- [ ] Remove all agents from a collection.
- [ ] Verify the empty collection state returns.

## Existing Favorites and Suites regression

- [ ] Favorite an agent from `AgentCard`.
- [ ] Verify `HomePage` still shows “Your Favorites”.
- [ ] Unfavorite the agent and verify favorites update.
- [ ] Visit `/suites`.
- [ ] Verify suite cards still render.
- [ ] Open a suite quiz through `SuiteWizard`.
- [ ] Verify suite recommendations still link to agents.
- [ ] Confirm no collection appears inside `src/suites/suitesData.js` data-driven UI.

## Other regression checks

- [ ] Visit `/` and verify agent search/category filter still work.
- [ ] Visit an `/agent/:id` page and verify recent agents still persist under `recentAgents`.
- [ ] Visit `/workflows` and verify workflow routes are unaffected.
- [ ] Visit `/battle` and verify battle mode still uses its full-screen layout without sidebar dependencies.
