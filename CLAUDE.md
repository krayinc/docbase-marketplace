# CLAUDE.md

## プロジェクト概要

DocBase用Claude Codeプラグインのマーケットプレイスリポジトリ。

## 構成

- `.claude-plugin/marketplace.json` — マーケットプレイス定義
- `plugins/` — 各プラグインのソース

## バージョン管理

- 相対パスプラグイン（`source: "./plugins/..."`）のバージョンは `marketplace.json` 側で管理する
- `plugin.json` に `version` を記載しない（marketplace.json が無視される恐れがあるため）

## バリデーション

変更後は必ず `claude plugin validate <plugin-dir>` を実行して確認する。
