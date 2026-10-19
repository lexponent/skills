# Contributing

- **Manifests.** Claude reads `.claude-plugin/` and `.mcp.json`. ChatGPT, Codex, Microsoft's Agents Toolkit, Perplexity and Mistral read the root `plugin.json` and `mcp.json`. Codex and the ChatGPT desktop app read `.agents/plugins/marketplace.json`. Keep the plugin name, server key and version the same in all of them; CI checks this.
- **Releases.** Raise `version` in both `plugin.json` files for every release, or installed copies won't update.
- **OpenAI upload.** Build the ZIP from an allowlist: `zip -r lexponent-openai.zip plugin.json mcp.json skills assets LICENSE`. Don't commit it.
