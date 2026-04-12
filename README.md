# backlog-migration-support-tool

[Backlog 移行ツール](https://backlog.com/ja/backlog-migration/releases.html) の利用時に発生しやすいエラーを解消するためのブラウザベースのサポートツールです。

## 概要

`init` コマンド実行後に生成される CSV ファイルを簡単に編集できる Web アプリです。  
サーバー不要・インストール不要。`index.html` をブラウザで開くだけで動作します。

## 機能

### 1. ユーザーマッピング CSV エディタ

| ステップ | 内容 |
|----------|------|
| **STEP 1** 作業フォルダ選択 | 移行ツールを展開したフォルダを選択すると `mapping/`・`log/` を自動読み込み |
| **STEP 2** 自動マッチング確認 | メールアドレス一致で移行先ユーザーを自動提案。行ごとの承認/拒否 + 一括承認/拒否 |
| **STEP 3** 手動編集・保存 | ドロップダウンで移行先を設定・絞り込み検索・UTF-8 で CSV ダウンロード |

### 2. ログ収集（今後実装予定）

移行実行後のログファイルを解析し、既知エラーパターンと照合してサポート問い合わせ用レポートを生成します。

## 対象ファイル

`init` コマンドを実行すると `mapping/` ディレクトリに以下のファイルが生成されます。

| ファイル | 説明 |
|----------|------|
| `mapping/users.csv` | ユーザーマッピングファイル（編集対象） |
| `mapping/users_list.csv` | 移行先スペースのユーザー一覧（参照用） |

### `users.csv` のカラム

| カラム | 説明 |
|--------|------|
| `Source Backlog user id` | 移行元ユーザーID |
| `Source Backlog user display name` | 移行元表示名 |
| `Source Backlog user email` | 移行元メールアドレス |
| `Destination Backlog user name` | **移行先ユーザーID**（ここを埋める必要がある）|

> ⚠️ `Destination Backlog user name` が空のままだと移行時に「ユーザーをコンバートできませんでした」エラーが発生します。

## 使い方

### 事前準備

1. [Backlog 移行ツール](https://backlog.com/ja/backlog-migration/releases.html) をダウンロード・展開
2. `init` コマンドを実行して `mapping/` ディレクトリを生成

```bash
# Zip版（macOS）の場合
./bin/backlog-migration init \
  --src.key <移行元APIキー> \
  --src.url https://<スペース名>.backlog.com/ \
  --dst.key <移行先APIキー> \
  --dst.url https://<スペース名>.backlog.com/ \
  --projectKey <移行元プロジェクトキー>:<移行先プロジェクトキー>
```

### ツールの起動

```bash
# index.html をブラウザで開く（ダブルクリックでも可）
open index.html
```

> ⚠️ **フォルダ選択機能（File System Access API）は Chrome / Edge のみ対応**です。  
> Safari・Firefox をお使いの場合は「個別ファイルで読み込む」欄を展開してください。

### 操作フロー

```
① 作業フォルダを選択
   └─ backlog-migration-1.7.0/ フォルダ全体を選択
      → mapping/users.csv ✅
      → mapping/users_list.csv ✅
      → log/*.log ✅（自動取得）

② 自動マッチング確認
   └─ メールアドレスが一致するユーザーを自動提案
      → 行ごとに ✅ / ✖ で承認・拒否
      → 「全て承認」で一括処理も可能

③ 手動編集・保存
   └─ 未設定の行（赤ハイライト）をドロップダウンで設定
      → 「users.csv をダウンロード」で保存
      → 移行ツールの mapping/ に上書き保存して移行を再実行
```

## 対応エラー

[Backlog 移行ツールでエラーが発生して移行できません](https://support-ja.backlog.com/hc/ja/articles/29471143249817) に記載されている以下のエラーに対応します。

| エラー | 原因 | 対応 |
|--------|------|------|
| `ユーザーをコンバートできませんでした` | `Destination Backlog user name` が未設定 | 本ツールで設定 |
| `The maximum number of status are 12` | 状態数が上限超過 | ログレポートで検出・案内（実装予定）|
| `No such Attachment` | 添付ファイル削除済み | 警告として分類（実装予定）|
| `project.EditProject.err.exist.key` | プロジェクトキー重複 | ログレポートで案内（実装予定）|

## 技術スタック

- HTML / Vanilla CSS / Vanilla JavaScript（ビルド不要）
- [PapaParse](https://www.papaparse.com/)（CSV パース、CDN 経由）
- File System Access API（フォルダ選択）

## 参考リンク

- [Backlog 移行ツール ダウンロードページ](https://backlog.com/ja/backlog-migration/releases.html)
- [マニュアル（Zip版）](https://backlog.com/ja/backlog-migration/backlog-migration-manual-zip-1.7.0.pdf)
- [マニュアル（Jar版）](https://backlog.com/ja/backlog-migration/backlog-migration-manual-jar-1.7.0.pdf)
- [エラーが発生して移行できません（サポート記事）](https://support-ja.backlog.com/hc/ja/articles/29471143249817)

## ライセンス

MIT License — Copyright (c) 2026 Tetsuya Shibao
