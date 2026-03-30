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

- Claude Code installed
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

```
/plugin marketplace add krayinc/docbase-marketplace
/plugin install docbase-cli@docbase-marketplace
```

## Author

[KRAY Inc.](https://kray.jp)

## License

MIT
