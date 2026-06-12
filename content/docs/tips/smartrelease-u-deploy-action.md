---
title: "SmartRelease U専用デプロイアクションの利用"
weight: 3
---

# SmartRelease U専用デプロイアクションの利用

SmartRelease U（SRU）のテストサーバーへのSFTPデプロイをより簡単かつ安全に行うため、専用のカスタムアクション `maepon/smartrelease-u-deploy-action` が提供されています。

本ドキュメントでは、このカスタムアクションを使用したデプロイワークフローの設定方法について解説します。

---

## 1. 概要とメリット

通常、GitHub ActionsからSmartRelease Uへのデプロイには `wlixcc/SFTP-Deploy-Action` を使用しますが、ホスト名やタイムアウト、SFTPプロトコルの制限などを手動で細かく定義する必要がありました。

専用アクション `maepon/smartrelease-u-deploy-action` を利用することで、以下のメリットが得られます。

* **設定ファイルの記述が大幅に削減される**：
  SmartRelease U のデフォルトサーバー (`sftp.pre-svr.jp`) やポート番号 (`22`) があらかじめ設定されているため、これらに関する変数の記述を省略できます。
* **最適な設定が自動適用される**：
  SmartRelease U の仕様である「SFTP接続のみ許可」が強制され、接続タイムアウト (`-o ConnectTimeout=5`) も自動でセットされます。
* **`size_only` の切り替えが簡単**：
  ファイルサイズのみで同期判定を行う設定が boolean (文字列) で容易に切り替え可能です。

---

## 2. 必要な GitHub Secrets

リポジトリの **Settings > Secrets and variables > Actions** にて、以下のリポジトリシークレット（Repository secrets）を登録します。

| シークレット名 | 説明 | 必須設定 |
| :--- | :--- | :--- |
| `SFTP_USERNAME` | FTPアカウントのユーザー名 | **必須** |
| `SFTP_PRIVATE_KEY` | SSH/SFTP接続用の秘密鍵（PEM形式） | **必須** |
| `SFTP_SERVER` | テストサーバーのホスト名（独自のものを使用する場合のみ） | 任意（デフォルト: `sftp.pre-svr.jp`） |
| `SFTP_PORT` | SSH/SFTPのポート番号（ポートを変更している場合のみ） | 任意（デフォルト: `22`） |

> [!NOTE]
> 通常の SmartRelease U 環境を利用している場合、`SFTP_SERVER` や `SFTP_PORT` を個別に登録する必要はありません。

---

## 3. ワークフローの設定例

リポジトリの `.github/workflows/` ディレクトリ内に YAML ファイル（例: `deploy.yml`）を作成し、以下のように記述します。

### ビルドを伴わないシンプルな静的サイトのデプロイ（HTML/CSSのみ）

ビルドステップがなく、Gitで管理しているHTML/CSS等のソースファイルを直接アップロードする例です。ここではリポジトリのルートにあるファイルを丸ごとデプロイします。

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy to SmartRelease U
        uses: maepon/smartrelease-u-deploy-action@v1
        with:
          sftp_username: ${{ secrets.SFTP_USERNAME }}
          sftp_private_key: ${{ secrets.SFTP_PRIVATE_KEY }}
          local_path: './*'
          remote_path: '/www'
```

> [!NOTE]
> Vite や Webpack などのビルドツールを使用しており、生成される `dist/` ディレクトリが `.gitignore` に指定されている場合は、デプロイ前に `npm run build` などのビルドステップをワークフローに追加する必要があります。


### Hugo サイトのビルド＆デプロイ例

このサイトのように、Hugo によるビルドステップ（SCSSコンパイル等を含む）を実行したのち、生成された `public/` ディレクトリを同期する構成です。変更されたファイルサイズのみで比較する `--size-only` を適用しています。

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build Hugo Site
        run: hugo --minify

      - name: Deploy via SFTP
        uses: maepon/smartrelease-u-deploy-action@v1
        with:
          sftp_username: ${{ secrets.SFTP_USERNAME }}
          sftp_private_key: ${{ secrets.SFTP_PRIVATE_KEY }}
          local_path: './public/*'
          remote_path: '/www'
          size_only: 'true' # ファイルサイズのみで同期判定を行う
```

---

## 4. アクションのインプットパラメータ詳細

`maepon/smartrelease-u-deploy-action` で指定できるパラメータは以下の通りです。

| パラメータ名 | 説明 | デフォルト値 | 必須 |
| :--- | :--- | :--- | :--- |
| `local_path` | デプロイするローカルパス（例: `./public/*`） | - | **Yes** |
| `remote_path` | デプロイ先のリモートパス（例: `/www`） | - | **Yes** |
| `sftp_username` | SFTPユーザー名 | - | **Yes** |
| `sftp_private_key`| SSH/SFTP接続用の秘密鍵（PEM形式） | - | **Yes** |
| `sftp_server` | SFTPサーバーのホスト名 | `sftp.pre-svr.jp` | No |
| `sftp_port` | SFTPポート番号 | `22` | No |
| `size_only` | ファイルサイズのみで同期判定を行うか (`true`/`false`) | `false` | No |
