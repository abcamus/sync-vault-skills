# Sync Vault Skills

Agent Skills for use with [Sync Vault](https://github.com/abcamus/obsidian-sync-vault-ce). 

These skills follow the [Agent Skills specification](https://github.com/kepano/agent-skills) so they can be used by any skills-compatible agent, including Trae, Claude Code, and Codex CLI.

## Installation

### Marketplace
```bash
/plugin marketplace add abcamus/sync-vault-skills
/plugin install sync-vault-skills
```

### npx
```bash
npx skills add git@github.com:abcamus/sync-vault-skills.git
```

### Manually

#### Trae
Add the contents of this repo to a `.trae/skills` folder in the root of your Obsidian vault. Trae will auto-discover all `SKILL.md` files.

#### Claude Code
Add the contents of this repo to a `/.claude` folder in the root of your Obsidian vault (or whichever folder you're using with Claude Code). See more in the [official Claude Skills documentation](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/custom-capabilities).

#### Codex CLI
Copy the `skills/` directory into your Codex skills path (typically `~/.codex/skills`).

---

## Skills

| Skill | Description |
| :--- | :--- |
| [**cloud-file**](./skills/cloud-file/SKILL.md) | Search, list, and manage files and account information in cloud storage (Baidu, Aliyun, Quark, etc.) |
| [**video-note**](./skills/video-note/SKILL.md) | Manage video annotations, timestamps, and playback links in Sync Vault. Supports AI note conversion for Quark and Baidu. |

## Requirements

These skills require the **Sync Vault** Obsidian plugin to be installed and the **MCP Server** to be enabled in the plugin settings.

- **Sync Vault Plugin**: [Download on GitHub](https://github.com/abcamus/obsidian-sync-vault-ce)
- **Minimum Version**: 1.10.6

---
Made by [Camus Qiu](https://kqiu.top)