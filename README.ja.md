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

- Claude Code がインストール済みであること
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

```
/plugin marketplace add krayinc/docbase-marketplace
/plugin install docbase-cli@docbase-marketplace
```

## 作者

[KRAY Inc.](https://kray.jp)

## ライセンス

MIT
