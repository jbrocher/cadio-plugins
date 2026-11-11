# Cadio Pantry

A plugin marketplace for Cadio: plan meals, view recipe cards, swap recipes,
and build grocery lists in ChatGPT desktop or Codex.

## Install

With Codex CLI installed, register this marketplace:

```sh
codex plugin marketplace add jbrocher/cadio-pantry --ref main
```

Restart ChatGPT desktop, open the Plugins Directory, select **Cadio Pantry**,
and install **Cadio**. Complete the Cadio OAuth connection when prompted, then
start a new conversation. The plugin includes the **Plan meals with Cadio** skill.

Try: "Use Cadio to plan five dinners for two, ready in 30 minutes."
Then: "Make a grocery list from that menu."

This MVP targets clients supporting repository marketplaces. It is not a listing
in OpenAI's public directory, and registering this source does not establish
availability in ChatGPT web or mobile. See the official
[marketplace setup guide](https://developers.openai.com/plugins/build/plugins#add-a-marketplace-from-the-cli).

## Connection and data

The plugin connects to `https://cadio.fly.dev/mcp`. Menus and grocery lists are
saved by the hosted Cadio service under your authenticated account. Recipe
searches use Spoonacular. The service must be running and your account must be
able to authenticate; installing this repository does not run the backend.

No API keys or tokens belong in this repository. Sign in through the host's
connection UI. If OAuth cannot complete, stop setup and report the error to the
maintainer; do not paste credentials into a conversation.

The current tools support required ingredients, meal type, and maximum preparation
time. They do not enforce dietary or allergen exclusions. Replacing menu recipes
does not refresh existing grocery lists. Repeating menu or grocery-list creation
creates another saved object.

## Update

```sh
codex plugin marketplace upgrade cadio-pantry
```

Use the plugin browser to update or reinstall Cadio if needed, and start a new
conversation to load the changed skill. Plugin versions use semantic versioning.

## Package contents

`.agents/plugins/marketplace.json` lists the plugins under `plugins/`.
Cadio uses the supported `.codex-plugin/plugin.json` manifest format, `.mcp.json`
for its remote connection, and `skills/plan-meals/` for workflow instructions.
The backend source and deployment credentials are maintained separately.
