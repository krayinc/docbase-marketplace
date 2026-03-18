---
name: docbase-cli
description: DocBase knowledge base CLI. Search/read/create/update memos, comments, users, groups, tags, attachments. Use when the user needs to interact with DocBase memos or team knowledge.
---

# DocBase CLI

Knowledge base operations via CLI. Check `docbase --help` for full options.

## Quick start

```bash
docbase auth status
docbase posts search -q "search query"
docbase posts get 1234567
docbase posts create --title "Title" --body "Body" --draft
```

## Core workflow

1. Search: `docbase posts search -q "keyword"`
2. Read: `docbase posts get <id>`
3. Create/Update with `--draft` by default

## Commands

### Memos

```bash
docbase posts search -q "query" --per-page 20 --summary
docbase posts get <id>
docbase posts create --title "Title" --body "Body" --draft --tags "tag1" "tag2" --scope private --no-notice
docbase posts update <id> --title "New title" --body "New body"
docbase posts delete <id>
docbase posts archive <id>
docbase posts unarchive <id>
```

Key options for `create`/`update`:
- `--scope <everyone|group|private>` (default: private)
- `--group-ids <ids...>` (required when scope=group)
- `--draft` / `--no-draft`
- `--tags <tags...>`
- `--notice` / `--no-notice`
- `--author-id <number>` - poster user ID
- `--published-at <datetime>` - ISO-8601 format

### Comments

```bash
docbase comments list <post-id> --per-page 20 --order desc
docbase comments list <post-id> --created-after 2026-01-01 --created-before 2026-03-01
docbase comments create <post-id> --body "Comment body" --no-notice
docbase comments delete <comment-id>
```

### Users

```bash
docbase users search -q "name"
docbase users get-profile
docbase users get-groups <user-id>
docbase users delete <user-id>
```

### Groups

```bash
docbase groups search --name "Group name"
docbase groups get <group-id>
docbase groups create --name "Name" --description "Desc"
docbase groups add-users <group-id> --user-ids 1 2 3
docbase groups remove-users <group-id> --user-ids 1 2 3
```

### Tags / Good Jobs / Attachments

```bash
docbase tags list
docbase good-jobs list <post-id>
docbase good-jobs create <post-id> --no-notice
docbase good-jobs delete <post-id> <good-job-id>
docbase attachments upload --file /path/to/file
docbase attachments download <file-id> -o /path/to/save
```

## Example: Search and summarize

```bash
docbase posts search -q "quarterly report" --summary
docbase posts get 1234567
docbase posts create --title "Summary" --body "..." --draft --tags "summary"
```

## Example: Long body with HEREDOC

```bash
docbase posts create --title "Title" --draft --body "$(cat <<'EOF'
## Heading

Markdown body here.
EOF
)"
```

## Example: Update memo scope

```bash
docbase posts update <id> --scope group --group-ids 100 200 --no-draft
```

## Tips

- Memo URL `https://your-team.docbase.io/posts/1234567` → ID is `1234567`
- Output is JSON. Parse with `jq` or `python3 -c "import json,sys; ..."`
- Use `docbase <command> --help` for detailed options
- Install: `npm install -g @krayinc/docbase-cli`
- Auth: `docbase auth login` (OAuth) then set team via env or config
