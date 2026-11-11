---
name: plan-meals
description: Create or update saved Cadio meal menus, display their recipes, and build or edit grocery lists. Use when the user wants to plan meals or shop from a Cadio menu.
---

# Plan meals with Cadio

Use the connected Cadio MCP tools for saved menus and grocery lists. Follow the
user's explicit instructions over this workflow. Use tools only for the actions
they requested; planning a menu does not itself request a grocery list.

## Create a menu

- Establish the number of recipes and servings from the request or conversation.
  Ask for either when missing rather than silently choosing a household size.
- `menu_compose` creates and saves a menu. Its optional `recipe_constraints`
  supports `include_ingredients` (ingredient-name strings), `meal_tags`, and
  `max_ready_time` in minutes. Defaults are main course, 45 minutes, and no
  required ingredients. Only the first meal tag is applied; an empty list leaves
  meal type unrestricted. Recipe count must be between 1 and 100.
- Required ingredients are not exclusions. The current tools have no dietary or
  allergen exclusion filter. If the request depends on an unsupported constraint,
  explain the limitation before composing and agree on an achievable scope.
  Do not claim a recipe is allergy-safe based on search results.
- Show the resulting recipes with `menu_get_cards`, or use `menu_get_detail` for
  ingredients and instructions or when cards cannot be displayed. Keep returned
  menu and recipe IDs available for later edits; do not invent IDs or recipes.

## Change a saved menu

Use the menu ID from the conversation. If it is missing, ask for it: the current
tools cannot list or retrieve a user's latest menu automatically.

Read `menu_get_detail` to identify the current recipes, then call
`menu_replace_recipes` with the requested recipe IDs in `target_recipe_ids`.
Reuse any supported constraints the user still wants. Display the updated menu.
Replacing recipes does not update previously created grocery lists.

## Make or edit a grocery list

- When requested, call `grocery_list_from_menu` with the saved menu ID. It scales
  ingredients to the menu's servings and creates a new saved list each time.
  It does not retrieve or refresh an existing list or preserve its manual edits.
- Present the returned quantities and units without inventing conversions or
  combining incompatible units. Keep the returned list and item IDs for edits.
- Use `grocery_list_add_item` for requested additions. Supply the list ID,
  ingredient name, servings, quantity, and unit when applicable, using the
  tool's command schema. Clarify quantities or units when they are ambiguous.
- Use `grocery_list_remove_item` with the returned item ID to remove an entire
  item, including all its recipe and manual quantities. Clarify first if the
  user instead wants to subtract only part of an item.
- These tools create shopping lists; they do not purchase groceries.

## Connection problems and uncertain results

If Cadio tools are unavailable, ask the user to connect or authenticate Cadio
through the host's connection UI. Never request tokens or passwords in chat.

Reads are safe to retry. Compose, list creation, and item addition can duplicate
data when repeated after a lost response; report uncertainty instead of retrying
them automatically. For an uncertain replacement, read the menu before deciding
whether another write is needed. A repeated removal can return item-not-found.
If the requested state cannot be verified with available tools, explain that
limitation rather than claiming success.
