---
name: plan-meals
description: Create or update Cadio meal menus, grocery lists, and a personal pantry. Use when the user wants to plan meals, shop, or apply pantry staples to a grocery list.
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
- Use `grocery_list_get` to read an existing list, including skipped items.
  Normal shopping output omits items with `shopping.status=skipped`. When
  `shopping.purchase_amount` is present, show that final purchase amount instead
  of adding it to the origin quantities. Keep origins as context during review.
- For an existing item's purchase decision, use `grocery_list_update_item`
  with its explicit list and item IDs. Set `status=active` to reactivate a
  skipped item, or `status=skipped` when the user has enough. Set
  `purchase_amount` to replace the final amount; omit it to preserve the amount,
  or use `reset_purchase_amount=true` to return to origin quantities. Do not
  combine reset with a new amount. Changing an amount alone does not reactivate
  a skipped item. Resolve ambiguous product matches and additive versus
  replacement intent with the user; the update tool does not match names.
- Use `grocery_list_add_item` for requested additions. Supply the list ID,
  ingredient name, servings, quantity, and unit when applicable, using the
  tool's command schema. Clarify quantities or units when they are ambiguous.
  It appends an origin to an exact-name match and preserves that item's shopping
  state and purchase override. Prefer an explicit item update when the intent is
  to reactivate an item or change how much to buy.
- Use `grocery_list_remove_item` with the returned item ID to remove an entire
  item, including all its recipe and manual quantities. Clarify first if the
  user instead wants to subtract only part of an item.
- These tools create shopping lists; they do not purchase groceries.

## Maintain and apply a pantry

- `pantry_get` reads the user's personal pantry. Use `pantry_add_entry`,
  `pantry_update_entry`, and `pantry_remove_entry` only for requested changes.
  A staple has a name and optional reminder, not a stock count or usual purchase
  amount. Do not infer exact inventory from a reminder.
- When the user asks to apply their pantry to an existing grocery list, use the
  list ID from the conversation. Ask for it if missing; the current tools do
  not list the user's grocery lists. Call `pantry_prepare_review` with that ID.
  It returns the current pantry, grocery items (including skipped items),
  automatic ID matches, and two snapshot tokens.
- Review the automatic matches and propose only additional entry/item ID pairs
  that are clear from product names and context. One pantry entry may match
  several grocery items, such as two butter lines from different recipes; keep
  each as a separate decision. Each grocery item may match only one pantry
  entry. Do not merge quantities, convert units, or guess a purchase amount.
  Leave uncertain pairs unmatched so the user can decide in the form.
- Call `pantry_review_ui` with the same grocery list ID, both returned snapshot
  tokens, and the additional pairs in `proposed_matches` (an empty list when
  there are none). The tool renders automatic and proposed matches together.
  The user can reject a match and choose Keep/Don't buy for each matched item;
  a pantry entry with no accepted matches offers Add/Don't add. The form asks
  for a purchase amount when adding an item and submits through
  `grocery_list_submit_review_form`, which uses the same atomic apply handler
  as `grocery_list_apply_review`. Let the form handle the user's inventory answers;
  do not supply them on the user's behalf.
- If rendering reports that the pantry or list changed, prepare a fresh review
  and present a new form. Do not reuse stale snapshot tokens or replay a review
  after an uncertain submission. Read `grocery_list_get` to establish the saved
  state before advising another write.

## Connection problems and uncertain results

If Cadio tools are unavailable, ask the user to connect or authenticate Cadio
through the host's connection UI. Never request tokens or passwords in chat.

Reads are safe to retry. Compose, list creation, and item addition can duplicate
data when repeated after a lost response; report uncertainty instead of retrying
them automatically. For an uncertain replacement, read the menu before deciding
whether another write is needed. A repeated removal can return item-not-found.
If the requested state cannot be verified with available tools, explain that
limitation rather than claiming success.
