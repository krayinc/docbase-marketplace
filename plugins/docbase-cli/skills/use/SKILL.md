---
name: use
description: DocBaseのメモ・コメント・ユーザー・グループ・タグ・グッジョブ・添付ファイルを操作するCLIツール。「メモ読んで」「メモ検索して」「コメントして」「DocBaseに投稿して」といったリクエストで使う。
---

# DocBase CLI

## 認証

`docbase auth status` で確認。未認証なら `docbase auth login` を実行。

## 出力の形

出力はすべてJSON。

- `posts search` → `{posts, meta}`
- `posts get` → メモ1件。本文 `body` は `\r\n` 区切り
- `posts create` / `update` / `patch-body` → メモ1件（既定で `body` なし）
- `comments list` → `{comments, meta}`
- `tags list` / `groups search` / `users search` → 配列

`meta` には `total`、`next_page`（次のページが無ければ null）、検索では `semantic_search` が入る。

```bash
docbase posts search -q "仕様" --per-page 5 --summary | jq '.posts[] | {id, title, url, body}'
docbase posts get <id> | jq -r '.body'
docbase posts create --title "タイトル" --draft --body-file ./body.md | jq '{id, url, draft, scope}'
docbase comments list <post-id> | jq '.comments[] | {id, user: .user.name, body}'
docbase tags list | jq '.[].name'
```

## メモ（posts）

### 検索
```bash
docbase posts search -q "リリース手順 desc:changed_at" --per-page 5 --summary
docbase posts search -q "障害時の対応方法" --per-page 5 --summary --semantic-search
```

- 候補を絞ってから、必要なメモだけ `posts get` で全文を取る
- `--summary` は本文を要約（無ければ先頭1000文字）に置き換えて返す
- 既定は10件（上限100）。続きは `-p 2`
- 並び順はクエリに書く: `desc:score`（関連度）、`desc:changed_at`（最近更新）、`desc:published_at`（最近公開）。`asc:` も使える
- 絞り込み: `title:` `tag:` `group:` `author:` `changed_at:2026-01-01~*` など
- `--semantic-search` は意味的に近いメモも探す。言い回しが揺れる自然文で使う。使えないチームでは通常の検索になる（`meta.semantic_search` で分かる）か、403 `rag_unavailable` になる（外して再実行）
- 候補が少なければ、キーワードを短くする・絞り込みを外す・`OR` でつなぐ

### 取得
```bash
docbase posts get <id>
```

IDやURLが分かっていれば検索せずに `get` する。`https://<team>.docbase.io/posts/1234567` の ID は `1234567`。

### 作成
```bash
docbase posts create --title "タイトル" --draft --body-file ./body.md --tags "タグ1" "タグ2" --no-notice
```
- `-t, --title` タイトル（必須）
- `-b, --body` 本文（Markdown）／ `-F, --body-file <path>` 本文をファイルから読み込み
- `-d, --draft` 下書きとして投稿
- `--tags <tags...>` タグ（複数指定可）
- `-s, --scope` 公開範囲: `everyone` / `group` / `private`（既定: private）
- `--group-ids <ids...>` 公開グループID（scope=group のときに必要）
- `--no-notice` 通知しない

**scope と draft の組み合わせ:**
- 指定なし or 下書き → `--draft`（`--scope` は付けない。応答の scope は private）
- 自分のみ公開 → `--scope private`（draft なし）
- グループ公開 → `--scope group --group-ids 100 200`（draft なし）

### 本文の書き方の使い分け

| やりたいこと | 使うもの |
| --- | --- |
| 長文や改行・記号の多い本文を書く | `--body-file`（`--body` に長文を渡すとシェルの引数の長さ制限で失敗することがある） |
| 本文の一部だけ直す | `patch-body` |
| 本文を全部書き直す | `update --body-file` |

### 更新
```bash
docbase posts update <id> --title "新タイトル"
docbase posts update <id> --body-file ./body.md --no-notice
```

- 変える項目だけを渡す（ほかの項目は変わらない）。`--body` / `--body-file` は本文全体を置き換える
- `--draft`（下書きにする）／`--no-draft`（下書きを解除する）／`--scope`／`--group-ids` は公開状態を変えるときだけ付ける

### 部分更新
```bash
# 行番号付きで本文を見る（長いメモは末尾に | sed -n '40,60p' を付けて範囲を絞る）
docbase posts get <id> | jq -r '.body | gsub("\r"; "") | split("\n") | to_entries[] | "\(.key + 1): \(.value)"'

# 2行目を置き換える
docbase posts patch-body <id> \
  --op '{"start":2,"end":2,"old_content":"2行目","content":"新しい2行目"}' \
  --no-notice
```

- 行番号は1始まりで、`end` の行も含む。`\r\n` で1行と数える
- `old_content` は `start`〜`end` の行の現在の内容、`content` は置き換え後の内容。どちらも複数行は `\n` でつなぐ
- `old_content` が一致しないと 409 Conflict。`posts get` で最新の本文を取り直してやり直す
- `--op` は複数指定でき、まとめて適用される（1つでも失敗すれば何も変わらない）。範囲は重ねない
- 自分で書いた直後など本文が分かっていれば、行番号を見るための `get` は不要

### 応答の確認

`create` / `update` / `patch-body` の結果は、応答の `id`・`url`・`draft`・`scope` などで確かめ、確認のためだけに `posts get` しない。本文も確かめたいときは `--include-body` を付けて応答で見る。

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

- ここに無いオプションは `docbase <command> --help` で確認できる
- 書き込み系（create / update / patch-body / comments create など）は既定で通知する。通知が要らなければ `--no-notice` を付ける
