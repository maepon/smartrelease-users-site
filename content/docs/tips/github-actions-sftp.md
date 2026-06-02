---
title: "GitHub ActionsによるSFTP自動デプロイ設定"
weight: 2
---

# GitHub ActionsによるSFTP自動デプロイ設定

SmartRelease U（SRU）のテストサーバーは、セキュリティの仕様上SFTPのみが許可されており、通常のSSHシェル接続やrsyncなどが制限されています。
しかし、GitHub Actionsと `wlixcc/SFTP-Deploy-Action` を組み合わせることで、GitHub上のブランチへのプッシュ（またはプルリクエストのマージ）をトリガーにした自動デプロイ（CI/CD）環境を簡単に構築できます。

本ドキュメントでは、シンプルなHTML/CSSサイトをそのままデプロイする基本設定と、Hugoなどのビルドステップを挟む応用設定について解説します。

---

## 1. 構成イメージと前提条件

GitHubリポジトリに変更をプッシュ（またはPRをマージ）した際、GitHub Actionsのランナーが起動し、指定したディレクトリ内のファイルをSFTP経由でSmartReleaseのテストサーバーへ同期します。

### 前提条件
* SmartRelease U のFTPアカウントが作成されていること。
* 接続に必要な秘密鍵（`id_rsa` 等）が手元にあること。
* デプロイに必要な認証情報をGitHubの「Secrets」に登録してあること。

---

## 2. GitHub Secrets の登録

安全にデプロイを行うため、接続情報はハードコーディングせず、GitHubリポジトリの **Settings > Secrets and variables > Actions** にて、以下の「Repository secrets」を登録します。

| シークレット名 | 説明 | 例 |
| :--- | :--- | :--- |
| `SFTP_SERVER` | テストサーバーのホスト名 | `ftp.pre-svr.jp` |
| `SFTP_USERNAME` | 作成したFTPユーザー名 | `your-ftp-user` |
| `SFTP_PRIVATE_KEY` | 接続用の秘密鍵（PEM形式）の内容 | `-----BEGIN OPENSSH PRIVATE KEY-----` から始まる全文 |
| `SFTP_PORT` | SSH/SFTPのポート番号（通常は22） | `22` |

> [!WARNING]
> 秘密鍵（`SFTP_PRIVATE_KEY`）を登録する際は、末尾の改行コードも含めてそのまま貼り付けてください。

---

## 3. 基本設定：ファイルをそのままデプロイする場合

リポジトリ内のソースコード（HTML/CSS/JSなど）をビルドせずにそのままアップロードする、最もシンプルな設定例です。

> [!NOTE]
> 本項では解説をシンプルにするため、`main`ブランチへの直接プッシュ（またはマージ）をトリガーとするシンプルな `push` イベントを例にしています。実運用において本番環境などへの誤デプロイ（誤爆）を防ぐためのより厳密なトリガー制御については、[5.4 誤デプロイ防止のためのトリガー制御（PRマージトリガー）](#54-誤デプロイ防止のためのトリガー制御prマージトリガー)を参照してください。

`.github/workflows/deploy.yml` に以下のように記述します。

```yaml
name: Deploy to Test Server

on:
  push:
    branches:
      - main # mainブランチへのプッシュ（マージ）時に実行
  workflow_dispatch: # 手動実行用のボタンをGitHub上に表示

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v6

      - name: Deploy via SFTP
        uses: wlixcc/SFTP-Deploy-Action@v1.2.6
        with:
          server: ${{ secrets.SFTP_SERVER }}
          username: ${{ secrets.SFTP_USERNAME }}
          ssh_private_key: ${{ secrets.SFTP_PRIVATE_KEY }}
          port: ${{ secrets.SFTP_PORT || '22' }}
          local_path: './*'      # リポジトリ直下の全ファイルをデプロイする場合（※末尾の「/*」が必須）
          remote_path: '/www'
          sftp_only: true
          args: '-o ConnectTimeout=5 --size-only'
```

---

## 4. 応用設定：ビルドステップを挟む場合（HugoなどのSSG）

静的サイトジェネレータ（Hugo、Vite、Next.jsなど）を使用しており、一度ビルドを行ってから生成された公開用ディレクトリ（`public` や `dist` など）をデプロイする場合の設定例です。

以下は、**Hugo（Extended版）**を使用してサイトをビルドした後にデプロイを行う例です。

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v6
        with:
          submodules: true # テーマをサブモジュール化している場合は必須
          fetch-depth: 0   # 最終更新日時などを取得するために必要

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true   # SCSSコンパイル等を行う場合は必須

      - name: Build Hugo Site
        run: hugo --minify # 静的ファイルを './public' に出力

      - name: Deploy via SFTP
        uses: wlixcc/SFTP-Deploy-Action@v1.2.6
        with:
          server: ${{ secrets.SFTP_SERVER }}
          username: ${{ secrets.SFTP_USERNAME }}
          ssh_private_key: ${{ secrets.SFTP_PRIVATE_KEY }}
          port: ${{ secrets.SFTP_PORT || '22' }}
          local_path: './public/*' # ビルドで生成された公開用ディレクトリの中身を指定（※末尾の「/*」が必須）
          remote_path: '/www'
          sftp_only: true
          args: '-o ConnectTimeout=5 --size-only'
```

---

## 5. 設定のポイントと注意点

### 5.1 `local_path` の指定にはワイルドカード `/*` が必須（重要）
`wlixcc/SFTP-Deploy-Action` の仕様上、指定したディレクトリの**「中身」**を `remote_path` へ展開してアップロードするには、末尾に `/*`（ワイルドカード）を付与する必要があります。

もし `local_path: './public'` のようにワイルドカードなしで指定すると、リモートの `/www` 配下に `public` フォルダ自体（`/www/public/`）が作成されてしまい、意図したドキュメントルート（`/www`）の直下にファイルが配置されなくなってしまいます。
意図通りのディレクトリ構造でデプロイするため、フォルダ内を展開して転送する場合は必ず `local_path: './public/*'` もしくは `local_path: './*'` と記述してください。
*(※一般的な SFTP アクションや lftp 同期ツールではワイルドカード不要なケースが多いですが、本アクション特有の仕様となります)*

### 5.2 `sftp_only: true` の指定
SmartRelease U ではセキュリティ上、シェルアクセスが拒否されています。そのため、SSHコマンド経由で処理を行うのではなく、純粋なSFTPプロトコルのみで処理を完結させるために `sftp_only: true` を指定する必要があります。

### 5.3 `--size-only` による転送の高速化
`args` パラメータで `--size-only` オプションを渡しています。
```yaml
args: '-o ConnectTimeout=5 --size-only'
```
通常、ファイルのタイムスタンプ比較などで同期判定を行いますが、SmartRelease U のような一部のサーバー環境ではタイムスタンプが正しく維持されないことがあります。`--size-only` を指定することで、ファイルサイズに変化がないファイルをスキップし、変更のあった最小限のファイルだけを高速に差分アップロードすることができます。

### 5.4 誤デプロイ防止のためのトリガー制御（PRマージトリガー）
`main`ブランチへの直接プッシュや意図しないコミットによる自動デプロイ（誤爆）を防ぐため、実際の開発運用では「プルリクエスト（PR）が承認され、マージされてクローズされた時のみ」実行されるように制御する設計が推奨されます。

その場合は、ワークフローのトリガー（`on`）とジョブの実行条件（`if`）を以下のように設定します。

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - closed # PRがクローズされたときにトリガーします
  workflow_dispatch: # 手動実行用

jobs:
  deploy:
    # PRが「マージされて」クローズされた場合、または手動実行の場合のみ実行
    if: github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    
    steps:
      # ... (チェックアウトやビルドなどのステップ) ...
      
      - name: Deploy via SFTP
        # ... (SFTP-Deploy-Actionの設定) ...
```

この制御を入れることで、単にPRが却下（マージされずに）されてクローズされた場合にはデプロイが走らず、マージに成功した時のみ安全にデプロイを実行することができます。
