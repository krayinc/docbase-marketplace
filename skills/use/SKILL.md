---
name: use
description: DocBaseのメモ・コメント・ユーザー・グループ・タグ・グッジョブ・添付ファイルを操作するCLIツール。「メモ読んで」「メモ検索して」「コメントして」「DocBaseに投稿して」といったリクエストで使う。
---

# DocBase CLI

## 認証

`docbase auth status` で確認。未認証なら `docbase auth login` を実行。

## 出力の形

出力は JSON（`posts get --body-only` / `--lines` を除く）。失敗したときは標準エラーに JSON が出る（「エラーと再試行」を参照）。

- `posts search` → `{posts, meta}`
- `posts get` → メモ1件。本文 `body` は `\r\n` 区切り（`--lf` で `\n`）
- `posts create` / `update` / `patch-body` → メモ1件（既定で `body` なし）
- `comments list` → `{comments, meta}`
- `tags list` / `groups search` / `users search` → 配列

`meta` には `total`、`next_page`（次のページが無ければ null）、検索では `semantic_search` が入る。

```bash
docbase posts search -q "仕様" --per-page 5 --summary | jq '.posts[] | {id, title, url, body}'
docbase posts get <id> --body-only --lf
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
docbase posts get <id> --body-only --lf   # 本文だけ（JSON にしない。改行は \n）
docbase posts get <id> --lines 40-60      # 40〜60 行目を「行番号<タブ>内容」で
docbase posts get <id> --no-body          # 本文以外の情報だけ
```

IDやURLが分かっていれば検索せずに `get` する。`https://<team>.docbase.io/posts/1234567` の ID は `1234567`。

- 長いメモの一部だけ読むときは `--lines`（`10-20` / `10` / `10-` で指定）、本文が要らないときは `--no-body` で出力を減らす
- `--lines` の行番号は `patch-body` の `start` / `end` と同じ数え方。行頭に番号が付くので、出力をそのまま `old_content` には使えない
- `--body-only` と `--lines`、`--no-body` と `--body-only`・`--lines` は同時に指定できない

### 作成
```bash
docbase posts create --title "タイトル" --draft --body-file ./body.md --tags "タグ1" "タグ2" --no-notice
```
- `-t, --title` タイトル（必須）
- `-b, --body` 本文（Markdown）／ `-F, --body-file <path>` 本文をファイルから読み込み
- `-d, --draft` 下書きとして投稿
- `--tags <tags...>` タグ（複数指定可。update ではタグ全体を置き換える）
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
# 行番号付きで本文を見る
docbase posts get <id> --lines 1-

# 2行目を置き換える
docbase posts patch-body <id> \
  --op '{"start":2,"end":2,"old_content":"2行目","content":"新しい2行目"}' \
  --no-notice

# operation の配列を JSON ファイルで渡す（シェルで JSON をエスケープせずに済む）
docbase posts patch-body <id> --ops-file ops.json --no-notice

# 10〜12 行目を new.md の内容に置き換える（CLI が本文を取得して old_content を組み立てる）
docbase posts patch-body <id> --replace 10-12 --content-file new.md --dry-run   # 送信内容の確認だけ
docbase posts patch-body <id> --replace 10-12 --content-file new.md --no-notice
```

- 更新内容は `--op`（複数回指定可）・`--ops-file`・`--replace` のどれか1つで渡す（同時には使えない）
- `--dry-run` は送信する operation を `{"operations":[...]}` で出すだけで、更新しない
- `--replace` は古い行番号を使うと意図しない行を置き換えても 409 にならない。直前の `posts get --lines` で行番号を確かめ、`--dry-run` で `old_content` を見てから実行する
- `--content-file` の末尾の改行1つは取り除かれる。空のファイルなら指定した行を削除する

- 行番号は1始まりで、`end` の行も含む。`\r\n` も `\n` も1つの改行と数える。本文が改行で終わるときは、最後に空の行が1行ある
- `old_content` は `start`〜`end` の行の現在の内容、`content` は置き換え後の内容。どちらも複数行は `\n` でつなぐ。`content` を空文字にするとその行を削除する
- `old_content` が一致しないと 409 Conflict。`posts get` で最新の本文を取り直してやり直す
- `--op` は複数指定でき、まとめて適用される（1つでも失敗すれば何も変わらない）。行番号はどれも変更前の本文で数える。範囲が重なると 400
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

## エラーと再試行

失敗すると、標準エラーの最後の1行に JSON を1つ出し、種類ごとの終了コードで終わる。

```json
{"error":{"type":"not_found","status":404,"message":"Not Found","retryable":false,"retry_after":null,"next_action":"ID と接続先チーム（team_domain）を確かめてください","team_domain":"example","details":{...}}}
```

| type | 終了コード | 主な原因 |
| --- | --- | --- |
| `invalid_input` | 2 | 引数・入力の誤り、HTTP 400・422 |
| `auth` | 3 | 未認証、トークン切れ、HTTP 401 |
| `permission` / `not_found` | 4 | HTTP 403 / 404 |
| `conflict` | 5 | HTTP 409（`patch-body` の `old_content` が一致しない） |
| `rate_limit` | 6 | HTTP 429 |
| `server` / `network` | 7 | HTTP 5xx / 応答なし |
| `unknown` | 1 | その他 |

- 次にすることは `next_action` に従う。`retryable` が true なら `retry_after` 秒待って同じ操作をやり直してよい
- 書き込みが `retryable: false` で失敗したとき（502・503・504、接続が途中で切れた）は反映済みかもしれない。やり直す前に `posts get` などで確かめる
- 読み取り（GET）は 429・502・503・504 のとき CLI が自動で最大3回再試行する。書き込みは再試行しない
- パイプでつなぐと終了コードは最後のコマンドのものになる。失敗を検知するなら `set -o pipefail` を使う

## Tips

- ここに無いオプションは `docbase <command> --help` で確認できる
- 書き込み系（create / update / patch-body / comments create など）は既定で通知する。通知が要らなければ `--no-notice` を付ける
