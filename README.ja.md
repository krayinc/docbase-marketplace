[English](README.md)

# DocBase Marketplace

[DocBase](https://docbase.io)をAIエージェントから活用するためのClaude Codeプラグイン集です。

## 概要

DocBase Marketplaceは、AIエージェント（Claude Code）からDocBaseのナレッジベースを直接操作できるプラグインを提供します。自然言語での指示だけで、メモの検索・作成・編集、コメント、ユーザー・グループ管理などが行えます。

## 含まれるプラグイン

### docbase-cli

[DocBase CLI](https://www.npmjs.com/package/@krayinc/docbase-cli)を活用し、Claude CodeからDocBaseを操作するスキルです。

#### 主な機能

- **メモ操作** — 検索、取得、作成、更新、削除、アーカイブ
- **コメント** — 一覧取得、投稿、削除
- **ユーザー管理** — 検索、プロフィール取得
- **グループ管理** — 検索、取得、作成、メンバー追加・削除
- **タグ** — 一覧取得
- **グッジョブ** — 送信
- **添付ファイル** — アップロード、ダウンロード

#### 使用例

```
「DocBaseで"議事録"を検索して」
「メモID 1234567 の内容を見せて」
「今日の作業ログをDocBaseに下書きで投稿して」
「メモ 1234567 にコメントして」
「DocBaseのタグ一覧を見せて」
```

## 前提条件

- Claude Code、Gemini CLI、Codex CLI、Cursor のいずれかがインストール済みであること
- [DocBase CLI](https://www.npmjs.com/package/@krayinc/docbase-cli) (`@krayinc/docbase-cli`) がインストール済みであること
- DocBase CLIで認証済みであること（`docbase auth login`）

### DocBase CLI のインストール

```bash
npm install -g @krayinc/docbase-cli
```

### 認証

```bash
docbase auth login
```

DocBaseのAPIトークンとチーム名を入力します。APIトークンは [DocBaseの設定画面](https://help.docbase.io/posts/45703#アクセストークン) から取得できます。

## インストール

### Claude Code

```
/plugin marketplace add krayinc/docbase-marketplace
/plugin install docbase-cli@docbase-marketplace
```

### Codex CLI

Codex CLI上で以下を実行します。

```
$skill-installer https://github.com/krayinc/docbase-marketplace
```

または、スキルをプロジェクトまたはユーザーの[スキルディレクトリ](https://developers.openai.com/codex/skills#install-curated-skills-for-local-use)に手動コピーします。

```bash
git clone https://github.com/krayinc/docbase-marketplace.git

# プロジェクトレベル
mkdir -p .agents/skills
cp -r docbase-marketplace/skills/docbase-cli .agents/skills/

# ユーザーレベル
cp -r docbase-marketplace/skills/docbase-cli ~/.agents/skills/
```

それぞれCodexを再起動してください。

### Cursor

> 現在、公式Cursorマーケットプレイスに公開のため申請中です。

#### チームマーケットプレイス

Teams/Enterpriseプランの管理者は、このリポジトリを[チームマーケットプレイス](https://cursor.com/ja/docs/plugins#-4)としてインポートできます。

1. **[プラグイン](https://cursor.com/dashboard/plugins) → チームマーケットプレイス** にアクセス
2. **+ import** をクリック
3. リポジトリURL `https://github.com/krayinc/docbase-marketplace` を入力
4. 解析されたプラグインを確認し、アクセスグループを設定して保存

チームメンバーは Cursor Settings → Plugins から `docbase` で検索してプラグインをインストールできます。

#### ローカルインストール

```bash
git clone https://github.com/krayinc/docbase-marketplace.git
cp -r docbase-marketplace/plugins/docbase-cli ~/.cursor/plugins/local/docbase-cli
```

コピー後、Cursorを再起動してください。

> **注意:** シンボリックリンクには対応していません。ファイルをコピーしてください。

### Gemini CLI

```bash
gemini extensions install https://github.com/krayinc/docbase-marketplace
```

## 作者

[KRAY Inc.](https://kray.jp)

## ライセンス

MIT
