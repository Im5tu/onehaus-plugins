# OneHaus for Hermes Agent

Connect [Hermes Agent](https://hermes-agent.nousresearch.com) to the OneHaus household MCP server.

OneHaus (https://onehaus.app) is a shared home organiser for families, couples and housemates. Calendar, chores, shopping, meals and home admin are in one app. Tagline: "Run your whole home from one app, not five."

This directory is an Agent Plugins v1 package: `plugin.json`, `mcp.json`, and `skills/onehaus/SKILL.md`. It ships no code. Every tool call goes to `https://mcp.onehaus.app` (Streamable HTTP).

There is no `plugin.yaml` here on purpose. Hermes loads `plugin.yaml` as a native plugin and then ignores `plugin.json`, so a manifest beside this package would stop the MCP server from connecting.

## Install

Before this plugin is in the Hermes catalogue:

```bash
hermes plugins install Im5tu/onehaus-plugins/plugins/onehaus
hermes plugins enable onehaus
```

After a catalogue entry is merged:

```bash
hermes plugins install onehaus
```

## Sign in

A OneHaus account is required. The server uses OAuth 2.0 with PKCE and dynamic client registration. Scopes are `haus.read` and `haus.write`.

- Authorise: https://onehaus.app/app/authorize
- Token: https://mcp.onehaus.app/token
- Register: https://mcp.onehaus.app/register

The package sets the server to `untrusted`. Hermes asks before any tool that is not annotated read-only, and refuses those calls in unattended runs. The bundled skill says to confirm with the user before creating, updating or deleting.

The tool list is served by the remote server at runtime. It is not pinned by this package.

## Access

New households get a 30-day free trial of OneHaus Pro, with no card. Pro is £4.99 a month or £49.99 a year. The MCP server is part of Pro. One-off tasks, shopping lists and the feedback channel stay available without it.

Docs: https://onehaus.app/docs/connect. Privacy: https://onehaus.app/privacy. Support: support@onehaus.app.

OneHaus is a trading name of ASB Media Investment Group Ltd (UK company 14376894).
