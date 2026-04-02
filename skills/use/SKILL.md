---
name: use
description: DocBaseのメモ・コメント・ユーザー・グループ・タグ・グッジョブ・添付ファイルを操作するCLIツール。「メモ読んで」「メモ検索して」「コメントして」「DocBaseに投稿して」といったリクエストで使う。
---

# DocBase CLI

## 認証

`docbase auth status` で確認。未認証なら `docbase auth login` を実行。

## メモ（posts）

### 検索
```bash
docbase posts search -q "検索クエリ" --per-page 20 -p 1
```

### 取得
```bash
docbase posts get <id>
```

### 作成
```bash
docbase posts create \
  --title "タイトル" \
  --body "本文（Markdown）" \
  --draft \
  --tags "タグ1" "タグ2" \
  --scope private \
  --no-notice
```
- `-t, --title <string>` タイトル
- `-b, --body <string>` 本文（Markdown）
- `-d, --draft` 下書きとして投稿（デフォルト: false）
- `--tags <tags...>` タグ（複数指定可）
- `-s, --scope <scope>` 公開範囲: `everyone` / `group` / `private`（デフォルト: private）
- `--group-ids <ids...>` 公開グループID（scope=group時に必要）
- `--notice / --no-notice` 通知する/しない（デフォルト: true）

**scopeとdraftの組み合わせルール:**
- 指定なし or 下書き → `--draft`（scopeなし）
- 自分のみ公開 → `--scope private`（draftなし）
- グループ公開 → `--scope group --group-ids 100 200`（draftなし）

**長い本文はHEREDOCを使う:**
```bash
docbase posts create --title "タイトル" --draft --body "$(cat <<'EOF'
## 見出し

本文をここに書く。
EOF
)"
```

### 更新
```bash
docbase posts update <id> --title "新タイトル" --body "新本文"
```
オプションは `create` と同じ。加えて `--no-draft` で下書き解除。

### 削除 / アーカイブ
```bash
docbase posts delete <id>
docbase posts archive <id>
docbase posts unarchive <id>
```

## コメント（comments）

```bash
docbase comments list <post-id> --per-page 20 --order desc
docbase comments create <post-id> --body "コメント内容" --no-notice
docbase comments delete <comment-id>
```

長いコメントはHEREDOC（posts createと同様）を使う。

## その他（`docbase <command> --help` で詳細確認）

```bash
docbase users search -q "名前" --per-page 100
docbase users get-profile
docbase groups search --name "グループ名"
docbase groups get <group-id>
docbase groups create --name "名前"
docbase groups add-users <group-id> --user-ids 1 2 3
docbase groups remove-users <group-id> --user-ids 1 2 3
docbase tags list
docbase good-jobs create <post-id>
docbase attachments upload --file /path/to/file
docbase attachments download <file-id> -o /path/to/save
```

## Tips

- メモURL `https://kray.docbase.io/posts/1234567` → ID は `1234567`
- 出力はすべてJSON。`jq` や `python3 -c "import json,sys; ..."` でパース可
- 更新時はまず `posts get` で現在の本文を取得してから変更を加える
