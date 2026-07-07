# CardoAI Client Plugins

Plugins from [CardoAI](https://cardoai.com) for asking Claude about your Prism data.

## Which app do you use?

- **Claude Code** — the terminal or desktop coding app, where you type commands like `/plugin`. Follow **Install** below.
- **Claude app** — claude.ai in the browser, or the desktop chat app (no terminal). Follow [Using the Claude app](plugins/prism-analyst/README.md#using-the-claude-app-claudeai) instead.

## Install (Claude Code)

**Step 1 — inside Claude Code**, add the CardoAI plugin catalog, then install the plugin:

```
/plugin marketplace add CardoAI/cardoai-client-plugins
/plugin install prism-analyst@cardoai-client-plugins
```

**Step 2 — in your terminal**, connect your Prism data:

```
claude mcp add --transport http --scope user --client-id prism-mcp prism-mcp https://prism.<your-tenant>.cardoaiapps.com/mcp/
```

`prism-mcp` appears **twice** — both are required, exactly as written:

| Part | What it is |
|---|---|
| `--client-id prism-mcp` | The OAuth client ID. Prism only accepts `prism-mcp`. |
| second `prism-mcp` | The name the connection is saved under. The plugin looks for this exact name. |
| `--scope user` | Makes the connection work in every folder, not just where you ran the command. |
| the URL | Your Prism endpoint — CardoAI sent it to you. Paste yours in place of the example. |

Your browser opens to sign in — one time per machine. Then just ask Claude Code about your deals.

Full details, verification, and troubleshooting: [prism-analyst README](plugins/prism-analyst/README.md).

## Plugins

| Plugin | What it does |
|---|---|
| [`prism-analyst`](plugins/prism-analyst/README.md) | Ask questions about your Prism deals in plain language — verified answers with confidence scores and source citations. |

New plugins get added over time — the setup above stays the same.

## License

Proprietary — for licensed use by CardoAI clients only. Contact CardoAI for terms.
