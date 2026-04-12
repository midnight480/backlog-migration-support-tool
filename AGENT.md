# AGENT.md — backlog-migration-support-tool

## プロジェクト概要

**backlog-migration-support-tool** は、[Backlog 移行ツール](https://backlog.com/ja/backlog-migration/releases.html)（Zip版 / Jar版）を使用する際に発生しやすいエラーを解消するためのサポートツールです。

主な機能は以下の 2 つです。

1. **CSV エディタ (ブラウザ GUI)**  
   移行ツールの `init` コマンドで生成される `users.csv`（ユーザーマッピング）と `users_list.csv`（移行先ユーザー一覧）をローカルブラウザ上で簡単に編集できるシンプルな Web アプリ。
   文字コード (UTF-8) の保証やフォーマット検証も行う。

2. **ログ収集・エラー診断ツール**  
   移行実行後にエラーが残る場合、ログファイルを収集・解析し、既知のエラーパターンと照合してサポート問い合わせ用レポートを生成する。

---

## 対象ツール・バージョン

| 種別 | バージョン | ダウンロード |
|------|-----------|-------------|
| Zip版 (macOS / Windows) | 1.7.0 | [backlog.com](https://backlog.com/ja/backlog-migration/releases.html) |
| Jar版 | 1.7.0 | [backlog.com](https://backlog.com/ja/backlog-migration/releases.html) |

マニュアル:
- [マニュアル(Zip版) PDF](https://backlog.com/ja/backlog-migration/backlog-migration-manual-zip-1.7.0.pdf)
- [マニュアル(Jar版) PDF](https://backlog.com/ja/backlog-migration/backlog-migration-manual-jar-1.7.0.pdf)

---

## 既知エラーと対応方針

[Backlog 移行ツールでエラーが発生して移行できません](https://support-ja.backlog.com/hc/ja/articles/29471143249817) に記載されている既知エラー一覧。本ツールはこれらに対応するための補助を提供します。

### 1. `ユーザーをコンバートできませんでした。"ユーザー名"`

| 原因 | 対応 |
|------|------|
| `users.csv` / `userlist.csv` に該当ユーザーのメールアドレスが不足している | CSV エディタでユーザー行を追加する |
| ファイルの文字コードが UTF-8 以外 | CSV エディタで UTF-8 として保存し直す |

> **補足:** 削除済みユーザーの場合は、移行先に仮ユーザーを作成して対処する。

### 2. `message - The maximum number of status are 12 per project`

- 移行元プロジェクトで「状態を追加」「状態名の変更」を合計 9 回以上行っている場合に発生。
- **現状は移行不可**。事前にこの回数を確認する方法はない。
- ログレポートにてこのエラーを検出・説明を付記する。

### 3. `message - No such Attachment`

- 移行元の課題から添付ファイルが削除済みの場合に表示される。
- 課題自体は移行されているため、**修正不要**。
- ログレポートでは「警告」扱いとして分類する。

### 4. `message - project.EditProject.err.exist.key`

- 移行先にプロジェクトキーが既に存在、かつ APIキー所有者がそのプロジェクトに未参加。
- ログレポートにて原因説明と対処手順（プロジェクトへの参加）を案内する。

### 5. `message - You can upload to 500 attachments. code - 9`

- 添付ファイル移行の内部上限 (500) を超えた場合に発生。
- `--filter` オプション（`count`/`offset`）で分割移行を案内する。
- ログレポートにてサンプルコマンドを生成する。

---

## ディレクトリ構成（予定）

```
backlog-migration-support-tool/
├── AGENT.md                  # このファイル
├── LICENSE
├── README.md
├── index.html                # メインエントリ（シングルページアプリ）
├── css/
│   └── style.css             # スタイルシート
├── js/
│   ├── app.js                # アプリケーションルーター
│   ├── csv-editor.js         # CSV 編集機能
│   ├── log-analyzer.js       # ログ解析・エラー照合
│   └── report-generator.js   # サポート問い合わせレポート生成
└── assets/
    └── (アイコン等)
```

---

## 機能詳細

### Feature 1: CSV エディタ

`init` コマンドを実行すると、`mapping/` ディレクトリに以下の2ファイルが生成されます。

#### `users.csv` — ユーザーマッピングファイル

移行元ユーザーを移行先ユーザーへマッピングするファイル。**このファイルの `Destination Backlog user name` 列が空のままだとエラーになります。**

| カラム名（英語ヘッダー） | 説明 |
|--------------------------|------|
| `Source Backlog user id` | 移行元のユーザーID（`*` 始まりは登録済み、それ以外は表示名）|
| `Source Backlog user display name` | 移行元の表示名 |
| `Source Backlog user email` | 移行元のメールアドレス（空の場合はゲストユーザー等）|
| `Destination Backlog user name` | **移行先のユーザーID**（空欄を埋める必要がある）|

**編集ポイント:** `Destination Backlog user name` 列に移行先の `users_list.csv` に記載されているユーザーID（`Name` 列の値）を設定する。

#### `users_list.csv` — 移行先ユーザー一覧ファイル

移行先スペースに存在するユーザーを一覧したファイル。`users.csv` を埋める際の参照元として使用する。

| カラム名（英語ヘッダー） | 説明 |
|--------------------------|------|
| `Name` | 移行先のユーザーID |
| `Email` | 移行先のメールアドレス |

#### 実際のサンプル（init実行例より）

**users.csv（編集前）**
```csv
"Source Backlog user id","Source Backlog user display name","Source Backlog user email","Destination Backlog user name"
"柴尾　哲也","柴尾　哲也","",""           ← Destination が空 → エラー原因
"etgonqvPAV","[JAWS-UG佐賀]柴尾哲也","shibao.tetsuya@midnight480.com",""  ← 要設定
"*s09FMa4O96","Tetsuya Shibao","tetsuya.shibao@nulab.com",""          ← 要設定
```

**users_list.csv（参照用）**
```csv
"Name","Email"
"etgonqvPAV","shibao.tetsuya@midnight480.com"
"*s09FMa4O96","tetsuya.shibao@nulab.com"
```

#### 機能要件
- `users.csv` と `users_list.csv` の両ファイルを同時に読み込み可能
- `users.csv` をテーブルで表示・編集
- `users_list.csv` の内容をドロップダウンとして `Destination Backlog user name` 列に候補表示
- `Destination Backlog user name` が空の行を赤くハイライト（エラー予防）
- 移行元メールアドレスと移行先メールアドレスが一致するユーザーを自動マッチング提案
- 保存時は必ず UTF-8 (BOMなし) で書き出し（ダウンロードボタン）
- 変更前のバックアップを localStorage に保持

### Feature 2: ログ解析・エラー診断

**入力**
- 移行ツール実行後に生成されるログファイル（`.log` または標準出力のテキストをペースト）

**解析フロー**
1. ログを行単位でパース
2. 既知エラーパターン（上記 5 種）と照合して分類
3. 各エラーに対して原因・対処手順を付与
4. サポート問い合わせ用のサマリレポートを Markdown / テキストで出力

**出力**
- エラーサマリ（エラー種別ごとの件数・内容）
- 対処済み / 未対処 の分類
- サポート連絡用テンプレート（バージョン・OS・エラー内容を含む）

---

## 技術スタック

| 項目 | 採用技術 |
|------|---------|
| UI | HTML / Vanilla CSS / Vanilla JavaScript |
| CSV パース | PapaParse (CDN) |
| ファイル読み書き | File API / Blob / URL.createObjectURL |
| ローカル実行 | ブラウザで `index.html` を直接開く（サーバー不要）|
| ビルドツール | なし（単一ファイル構成） |

> **サーバー不要**: ユーザーがブラウザで `index.html` をローカルに開くだけで動作する。外部への通信は一切行わない。

---

## 開発コマンド

```bash
# ローカル確認（Python の簡易サーバーを使う場合）
cd backlog-migration-support-tool
python3 -m http.server 8080
# → http://localhost:8080 で確認

# または VS Code の Live Server 拡張機能を使用
```

---

## 実装上の注意点

1. **文字コード**: CSV の読み書きは必ず UTF-8 (BOMなし) に統一する。`TextDecoder` / `TextEncoder` を使用。
2. **ファイル書き出し**: `<a download>` + `Blob` パターンを使用。Safari でも動作確認を行う。
3. **バリデーション**: メールアドレス列が空のユーザーは保存前に警告ダイアログを表示する。
4. **ログ解析の正規表現**: エラーメッセージは英語で固定のため、完全一致パターンを先に定義し、部分一致をフォールバックとする。
5. **プライバシー**: ログ・CSV データはすべてブラウザのメモリ内のみで処理し、外部送信しない旨を UI 上に明示する。

---

## 参考リンク

- [Backlog 移行ツール ダウンロードページ](https://backlog.com/ja/backlog-migration/releases.html)
- [マニュアル(Zip版)](https://backlog.com/ja/backlog-migration/backlog-migration-manual-zip-1.7.0.pdf)
- [マニュアル(Jar版)](https://backlog.com/ja/backlog-migration/backlog-migration-manual-jar-1.7.0.pdf)
- [エラーが発生して移行できません（サポート記事）](https://support-ja.backlog.com/hc/ja/articles/29471143249817)
- [Backlog API ドキュメント](https://developer.nulab.com/ja/docs/backlog/)
- [PapaParse (CSV ライブラリ)](https://www.papaparse.com/)
