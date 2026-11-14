# Contributing

Questions and reports go to support@onehaus.app.

This repository is public configuration only: install snippets, the official registry manifest, the skills.sh skill, and the Hermes Agent plugin. Changes should stay factual and match the published OneHaus docs at https://onehaus.app/docs. Do not add tool names or product behaviour that the public docs or the live MCP tool list do not show.

The Hermes package lives in `plugins/onehaus`. It is an Agent Plugins v1 package (`plugin.json`, `mcp.json`, and `skills/onehaus/SKILL.md`). Do not add a `plugin.yaml` beside `plugin.json`. Hermes treats `plugin.yaml` as a native plugin and then ignores `plugin.json`, so the MCP server would not load.
