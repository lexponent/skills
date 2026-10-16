# Lexponent

Lexponent answers questions about court rules and builds filing checklists, with every point cited to the rule text. It covers federal courts and state courts in a set of states, including local rules where Lexponent has them; see [lexponent.ai/coverage](https://www.lexponent.ai/coverage) for the current list. It is for attorneys, paralegals, legal secretaries, and filing and docketing staff.

The plugin bundles two skills and the Lexponent connector:

- **Answering rule questions** (`/lexponent:answering-rule-questions`): answers a specific procedural question from the rule text and cites each rule it relies on.
- **Generating filing checklists** (`/lexponent:generating-filing-checklists`): lists the formatting and content requirements for a filing in a given court, each one cited, and can check your draft against the list.

## Try asking

Ask in plain language; the right skill starts on its own. Name the court.

- "How do I demand a jury trial in the Northern District of California?"
- "Give me a filing checklist for a motion to compel further responses in LA Superior Court."
- "What is the deadline to file a notice of appeal from a civil judgment in federal court?"

## Install

You need a Lexponent account with connector access. Create one at [lexponent.ai](https://www.lexponent.ai) before you connect.

### Claude (web, desktop, Cowork)

1. Open **Customize → Plugins**, select **Add → Add marketplace → Add from a repository**, and enter `lexponent/skills`.
2. Install **Lexponent**.
3. On the plugin's **Connectors** tab, connect **Lexponent** and sign in with your Lexponent account.

If you already added Lexponent as a custom connector, remove it so you don't see every tool twice. If you installed the earlier `lexponent-plugin`, remove it so you don't see every skill twice.

### Claude Code

```
/plugin marketplace add lexponent/skills
/plugin install lexponent@lexponent
```

Run `/mcp` and sign in to **Lexponent** when prompted.

### Codex and the ChatGPT desktop app

```
codex plugin marketplace add lexponent/skills
codex plugin add lexponent@lexponent
```

### Other assistants

Connect `https://mcp.lexponent.ai/mcp` as a remote MCP server (Streamable HTTP, OAuth sign-in).

## Data

- **What the plugin runs.** The plugin is Markdown instructions and a connection setting. It runs no code on your computer and stores no credentials.
- **What it sends.** When your assistant calls a Lexponent tool, it sends the tool's arguments over HTTPS to `https://mcp.lexponent.ai/mcp`: the jurisdiction, the rule documents to search, and the search phrase, rule number, or text pattern the assistant composed. Your documents and conversation are not sent; when you ask for a draft review, the comparison happens in your assistant.
- **What Lexponent does with it.** Lexponent records each call (which tool, your account, the connecting app, and the result status) and logs rule numbers looked up by title. It does not log search phrases. To answer a search, Lexponent sends the search phrase to OpenAI to compute a text embedding.
- **Links.** Citation links open rule PDFs hosted by Lexponent at `lexponent-rule-documents.s3.us-east-1.amazonaws.com`.
- **Your assistant.** Rule text that Lexponent returns, and anything you type, is processed by the assistant you use under that provider's terms.

Keep client names and confidential facts out of your questions; the tools don't need them. See the [privacy policy](https://www.lexponent.ai/privacy-policy) and [terms of service](https://www.lexponent.ai/terms-of-service).

## Limits

Lexponent looks up and cites court rules and statutes. It does not search case law, file or serve documents, add dates to a calendar, or predict how a court will rule. A question about a jurisdiction Lexponent doesn't cover gets an answer saying so. Rules change: open the linked source before you file. Lexponent is a research tool, not legal advice.

## Troubleshooting

- **No Lexponent tools appear.** Check that the plugin is enabled and the Lexponent connector is connected (the plugin's **Connectors** tab, or `/mcp` in Claude Code).
- **"Your Lexponent account doesn't include connector access."** Your account has no plan with connector access. See [plans](https://www.lexponent.ai/pricing) or email support.
- **Signed in to the wrong account.** Disconnect the connector and connect again with the right account.

## Support

Email [support@lexponent.ai](mailto:support@lexponent.ai) or visit [lexponent.ai/support](https://www.lexponent.ai/support).

## Maintainers

- **Manifests.** Claude reads `.claude-plugin/` and `.mcp.json`. ChatGPT, Codex, Microsoft's Agents Toolkit, Perplexity and Mistral read the root `plugin.json` and `mcp.json`. Codex and the ChatGPT desktop app read `.agents/plugins/marketplace.json`. Keep the plugin name, server key and version the same in all of them; CI checks this.
- **Releases.** Raise `version` in both `plugin.json` files for every release, or installed copies won't update.
- **OpenAI upload.** Build the ZIP from an allowlist: `zip -r lexponent-openai.zip plugin.json mcp.json skills assets LICENSE`. Don't commit it.

## License

MIT. See [LICENSE](./LICENSE).
