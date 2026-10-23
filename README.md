# offgen for AI tools

Generate branded, consultant-quality PowerPoint decks from your organization's [offgen](https://offgen.ai) workflows, right inside Claude, Codex, Cursor, VS Code, Langdock or ChatGPT. Coding agents can also onboard slides as Library templates, search them, and merge and check decks built from them.

Server URL: `https://api.offgen.ai/mcp`. Sign in with your offgen account (OAuth), or use an organization API key from [Settings → API keys](https://app.offgen.ai/settings/api-keys).

## Install

**Claude Code** (plugin with the offgen skill):

```bash
claude plugin marketplace add offgen-ai/offgen-plugins
claude plugin install offgen@offgen
```

Leave the API key empty to sign in with offgen, or pass `--config offgen_api_key=offgen_…`.

**Codex** (plugin with the offgen skill):

```bash
codex plugin marketplace add offgen-ai/offgen-plugins
codex plugin add offgen@offgen
```

**Any MCP client**: add `https://api.offgen.ai/mcp` as a remote (Streamable HTTP) server and sign in with offgen.

## Contents

| Path                                                     | For                                                          |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `plugins/offgen/.claude-plugin/plugin.json`, `.mcp.json` | Claude Code plugin                                           |
| `plugins/offgen/plugin.json`, `mcp.json`                 | Agent Plugins 1.0 (Codex, Cursor, VS Code)                   |
| `plugins/offgen/skills/offgen-presentations/SKILL.md`    | Agent Skills: how to drive the offgen tools                  |
| `plugins/offgen/skills/offgen-templates/SKILL.md`        | Agent Skills: onboard, find and build from Library templates |
| `.claude-plugin/`, `.agents/plugins/`, `.cursor-plugin/` | marketplace manifests                                        |
| `server.json`                                            | MCP Registry entry `ai.offgen/offgen`                        |

This repository is published automatically from offgen's internal repository, so pull requests here are not merged. Questions or problems: [support@offgen.ai](mailto:support@offgen.ai).
