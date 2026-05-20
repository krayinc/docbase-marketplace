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
- `-F, --body-file <path>` 本文をファイルから読み込み
- `-d, --draft` 下書きとして投稿（デフォルト: false）
- `--tags <tags...>` タグ（複数指定可）
- `-s, --scope <scope>` 公開範囲: `everyone` / `group` / `private`（デフォルト: private）
- `--group-ids <ids...>` 公開グループID（scope=group時に必要）
- `--notice / --no-notice` 通知する/しない（デフォルト: true）

**scopeとdraftの組み合わせルール:**
- 指定なし or 下書き → `--draft`（scopeなし）
- 自分のみ公開 → `--scope private`（draftなし）
- グループ公開 → `--scope group --group-ids 100 200`（draftなし）

**長い本文はHEREDOCもしくは `--body-file` を使う:**
```bash
docbase posts create --title "タイトル" --draft --body "$(cat <<'EOF'
## 見出し

本文をここに書く。
EOF
)"

docbase posts create --title "タイトル" --draft --body-file ./body.md
```

### 更新
```bash
docbase posts update <id> --title "新タイトル" --body "新本文"
```
オプションは `create` と同じ。加えて `--no-draft` で下書き解除。

### 部分更新
```bash
docbase posts patch-body <id> \
  --op '{"start":2,"end":2,"old_content":"2行目","content":"新しい2行目"}'
```

本文を行単位で部分更新する。本文全体を送り直す `update` より転送量・競合リスクが小さい。
`old_content` が現在の本文と一致しないと 409 Conflict（楽観ロック）。複数 `--op` はアトミックに適用される。

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
docbase comments create <post-id> --body-file ./comment.md
docbase comments delete <comment-id>
```

長いコメントはHEREDOCもしくは `--body-file`（posts createと同様）を使う。

## その他

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

- 各コマンドの詳細なオプションは `docbase <command> --help` で確認できる
- メモURL `https://kray.docbase.io/posts/1234567` → ID は `1234567`
- 出力はすべてJSON。`jq` や `python3 -c "import json,sys; ..."` でパース可
- 更新時はまず `posts get` で現在の本文を取得してから変更を加える
