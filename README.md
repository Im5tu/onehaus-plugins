<p align="center">
  <img src="assets/apple-icon.png" alt="OneHaus" width="180" height="180">
</p>

# OneHaus

Run your whole home from one app, not five.

[OneHaus](https://onehaus.app) is a shared home organiser for families, couples and housemates. Calendar, chores, shopping, meals and home admin live in one app, and the same household is available to an AI assistant through a remote MCP server.

OneHaus is a trading name of ASB Media Investment Group Ltd (UK company 14376894).

A new household gets a 30-day free trial of OneHaus Pro, with no card. Pro is £4.99 a month or £49.99 a year, and one subscription covers the whole household. Connecting an assistant needs a OneHaus account. The MCP server is part of Pro, which the trial includes. One-off tasks, shopping lists and the feedback channel stay available after the trial. Prices shown in the app can vary by store and region. Details: [Plans and subscription](https://onehaus.app/docs/subscription).

## MCP server

| | |
| --- | --- |
| Endpoint | `https://mcp.onehaus.app` |
| Transport | Streamable HTTP |
| Registry name | `app.onehaus/haus` |
| Registry version | 1.3.0 |
| Scopes | `haus.read`, `haus.write` |
| Authorise | https://onehaus.app/app/authorize |
| Token | https://mcp.onehaus.app/token |
| Register | https://mcp.onehaus.app/register |

Sign-in is OAuth 2.0 with PKCE and dynamic client registration. The assistant opens the OneHaus sign-in page. You never paste a password or a key into the assistant.

The published registry record for `app.onehaus/haus` 1.3.0 uses this same remote URL. Its repository field still points at the private application repo. `server.json` in this repo repeats the public registry fields and points the repository at this public config repo. This repo does not contain the server implementation.

The server exposes 77 tools for calendar, chores, shopping lists, meals and home admin (contacts, pets, vehicles, appliances, documents, subscriptions, house review and household memory). Names are listed in [`skills/onehaus/SKILL.md`](skills/onehaus/SKILL.md), taken from the public tool list on 14 November 2026. Connect docs describe what you can ask for: [Connect AI assistants](https://onehaus.app/docs/connect).

## Install

### Claude (Desktop and claude.ai)

Custom connectors need a paid Claude plan.

1. Open Settings, then Connectors.
2. Choose Add custom connector. Name it `OneHaus`.
3. Server address: `https://mcp.onehaus.app`.
4. Sign in with your OneHaus account and approve access.

Claude Code:

```bash
claude mcp add --transport http onehaus https://mcp.onehaus.app
```

Add `--scope user` to make it available in every project. Start Claude Code, run `/mcp`, select onehaus, and authenticate.

### ChatGPT

Custom connectors are a paid ChatGPT feature. Some plans also need developer mode.

1. Open Settings, then Connectors (this may be under Apps).
2. Add a custom connector or a remote MCP server.
3. Name it `OneHaus`. Server URL: `https://mcp.onehaus.app`.
4. Sign in with your OneHaus account and approve access.

ChatGPT uses the server's OAuth flow. Do not paste a token into the connector form.

### Cursor

Settings, then MCP (or Tools), and add a server with the URL `https://mcp.onehaus.app`. Or edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (this project):

```json
{
  "mcpServers": {
    "onehaus": {
      "url": "https://mcp.onehaus.app"
    }
  }
}
```

One-click install:

[cursor://anysphere.cursor-deeplink/mcp/install?name=onehaus&config=eyJ1cmwiOiJodHRwczovL21jcC5vbmVoYXVzLmFwcCJ9](cursor://anysphere.cursor-deeplink/mcp/install?name=onehaus&config=eyJ1cmwiOiJodHRwczovL21jcC5vbmVoYXVzLmFwcCJ9)

The `config` value is the base64 of `{"url":"https://mcp.onehaus.app"}`, which is the remote server object from the [Cursor install-link docs](https://cursor.com/docs/mcp/install-links).

### VS Code

In Copilot, run Chat: Open Customizations, open MCP Servers, choose Add Server, then HTTP, and enter `https://mcp.onehaus.app`.

The current portable file is `.mcp.json` at the workspace root, or `~/.copilot/mcp-config.json` for every workspace (`$COPILOT_HOME/mcp-config.json` when that variable is set). From the [VS Code MCP configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration):

```json
{
  "mcpServers": {
    "onehaus": {
      "type": "http",
      "url": "https://mcp.onehaus.app"
    }
  }
}
```

The older editor-managed file is `.vscode/mcp.json` (user profile: MCP: Open User Configuration). It uses `servers` rather than `mcpServers`:

```json
{
  "servers": {
    "onehaus": {
      "type": "http",
      "url": "https://mcp.onehaus.app"
    }
  }
}
```

### Windsurf

VS Code's MCP configuration reference still discovers Windsurf at `~/.codeium/windsurf/mcp_config.json`. Published Windsurf examples put the remote URL in `serverUrl` (this field was not restated on the current docs page, which has moved to Devin Desktop):

```json
{
  "mcpServers": {
    "onehaus": {
      "serverUrl": "https://mcp.onehaus.app"
    }
  }
}
```

The current docs on that site describe Devin Desktop. Open Customize, then MCPs, add the URL `https://mcp.onehaus.app`, and authenticate when the server asks. Those docs did not include a JSON example, so if the field name differs in your build, paste the URL into the form rather than guessing another key.

### Hermes Agent

The plugin in [`plugins/onehaus`](plugins/onehaus) is an Agent Plugins v1 package: `plugin.json`, `mcp.json` (Streamable HTTP), and a bundled skill. It has no `plugin.yaml`, because Hermes would then ignore `plugin.json` and the server would not connect.

```bash
hermes plugins install Im5tu/onehaus-plugins/plugins/onehaus
hermes plugins enable onehaus
```

Once the catalogue entry in [`hermes/plugin-catalog/onehaus.yaml`](hermes/plugin-catalog/onehaus.yaml) is merged into NousResearch/hermes-agent:

```bash
hermes plugins install onehaus
```

The package marks the server untrusted, so Hermes asks before any tool that is not annotated read-only, and refuses those calls when nobody can answer.

To add the server in `config.yaml` without the plugin:

```yaml
mcp_servers:
  onehaus:
    url: https://mcp.onehaus.app
    auth: oauth
    trust: untrusted
```

### Generic `mcpServers`

[`mcp.json`](mcp.json) at the root of this repo:

```json
{
  "mcpServers": {
    "onehaus": {
      "type": "streamableHttp",
      "url": "https://mcp.onehaus.app"
    }
  }
}
```

Clients spell the transport differently. Cursor often wants `url` alone. VS Code's portable file wants `"type": "http"`. The Hermes package file wants `"type": "streamable-http"` and the Agent Plugins schema. Use the section for your client if this generic file does not connect.

## Skill

```bash
npx skills add Im5tu/onehaus-plugins
```

The skill is [`skills/onehaus/SKILL.md`](skills/onehaus/SKILL.md). It tells an agent when to use the server, how sign-in works, and how to add a calendar event, assign a chore, and build a shopping list from a meal plan. It says to confirm before creating or deleting. The same file is bundled with the Hermes plugin.

## Layout

| Path | What it is |
| --- | --- |
| `mcp.json` | Generic `mcpServers` config (`streamableHttp`) |
| `server.json` | Official MCP registry manifest for `app.onehaus/haus` 1.3.0 |
| `glama.json` | Glama maintainer claim (`Im5tu`) |
| `skills/onehaus/SKILL.md` | skills.sh skill |
| `plugins/onehaus/` | Hermes Agent plugin |
| `hermes/plugin-catalog/onehaus.yaml` | Draft catalogue entry, for a later pull request to hermes-agent |
| `assets/apple-icon.png` | Logo, 180 by 180, from https://onehaus.app/apple-icon.png |

MCP Market takes a public GitHub URL through its submit form. It does not read a manifest in the repo. Smithery's current manifest format was not confirmed from Smithery's own docs, so this repo does not include a `smithery.yaml`.

## Docs and support

- Docs: https://onehaus.app/docs
- Connect an assistant: https://onehaus.app/docs/connect
- Privacy: https://onehaus.app/privacy
- Support: support@onehaus.app
