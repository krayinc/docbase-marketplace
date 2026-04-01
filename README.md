[日本語](README.ja.md)

# DocBase Marketplace

A collection of Claude Code plugins for interacting with [DocBase](https://docbase.io) — a knowledge sharing platform for teams.

## Overview

DocBase Marketplace provides plugins that let AI agents (Claude Code) directly operate your DocBase knowledge base. With natural language instructions, you can search, create, and edit memos, post comments, manage users and groups, and more.

## Plugins

### docbase-cli

A skill that enables Claude Code to operate DocBase through the [DocBase CLI](https://www.npmjs.com/package/@krayinc/docbase-cli).

#### Features

- **Memos** — Search, get, create, update, delete, archive
- **Comments** — List, create, delete
- **Users** — Search, get profile
- **Groups** — Search, get, create, add/remove members
- **Tags** — List all tags
- **Good Jobs** — Send a "Good Job" reaction
- **Attachments** — Upload, download

#### Usage Examples

```
"Search DocBase for meeting notes"
"Show me memo ID 1234567"
"Post today's work log as a draft to DocBase"
"Add a comment to memo 1234567"
"List all tags on DocBase"
```

## Prerequisites

- Claude Code, Gemini CLI, Codex CLI, or Cursor installed
- [DocBase CLI](https://www.npmjs.com/package/@krayinc/docbase-cli) (`@krayinc/docbase-cli`) installed
- Authenticated with DocBase CLI (`docbase auth login`)

### Install DocBase CLI

```bash
npm install -g @krayinc/docbase-cli
```

### Authentication

```bash
docbase auth login
```

Enter your DocBase API token and team name. You can generate an API token from your [DocBase settings](https://help.docbase.io/posts/45703#アクセストークン).

## Installation

### Claude Code

```
/plugin marketplace add krayinc/docbase-marketplace
/plugin install docbase-cli@docbase-marketplace
```

### Codex CLI

Run the following in Codex CLI:

```
$skill-installer https://github.com/krayinc/docbase-marketplace
```

Or manually copy the skill to your project or user [skills directory](https://developers.openai.com/codex/skills#install-curated-skills-for-local-use):

```bash
git clone https://github.com/krayinc/docbase-marketplace.git

# Project-level
mkdir -p .agents/skills
cp -r docbase-marketplace/skills/docbase-cli .agents/skills/

# Or user-level
cp -r docbase-marketplace/skills/docbase-cli ~/.agents/skills/
```

Restart Codex after installation.

### Cursor

> The plugin is currently under review for publication on the official Cursor Marketplace.

#### Team Marketplace

Admins on Teams/Enterprise plans can import this repository as a [team marketplace](https://cursor.com/docs/plugins#add-a-team-marketplace):

1. Go to **[Plugins](https://cursor.com/dashboard/plugins) → Team Marketplaces**
2. Click **+ Import**
3. Enter the repository URL: `https://github.com/krayinc/docbase-marketplace`
4. Review parsed plugins, configure access groups, then save

Team members can search for `docbase` in Cursor Settings → Plugins to install the plugin.

#### Local Installation

```bash
git clone https://github.com/krayinc/docbase-marketplace.git
cp -r docbase-marketplace/plugins/docbase-cli ~/.cursor/plugins/local/docbase-cli
```

Restart Cursor after copying.

> **Note:** Symbolic links are not supported. You must copy the files.

### Gemini CLI

```bash
gemini extensions install https://github.com/krayinc/docbase-marketplace
```

## Author

[KRAY Inc.](https://kray.jp)

## License

MIT
