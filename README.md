# smartrelease-users-site
SmartRelease U のユーザー向けドキュメントサイトです。Hugo を使用して構築されています。

## 環境構築

このプロジェクトをローカルで実行するには、**Hugo Extended** バージョンが必要です。

### 1. Hugo のインストール
OS に合わせて Hugo Extended をインストールしてください。

- **macOS (Homebrew):**
  ```bash
  brew install hugo
  ```
- **Windows (Chocolatey):**
  ```bash
  choco install hugo-extended
  ```

### 2. リポジトリのクローンとサブモジュールの初期化
テーマが含まれているため、サブモジュールの初期化が必要です。

```bash
git clone https://github.com/maepon/smartrelease-users-site.git
cd smartrelease-users-site
git submodule update --init --recursive
```

### 3. ローカルサーバーの起動
```bash
hugo server -D
```
起動後、 `http://localhost:1313/` でプレビューを確認できます。

---

## 記事の追加・更新方法

### 1. 記事の追加
新しいドキュメントを追加するには、以下のコマンドを使用します。

```bash
hugo new docs/new-guide.md
```
`content/docs/new-guide.md` が生成されます。

### 2. コンテンツの編集
- **保存場所:** `content/docs/` 配下の Markdown ファイルを編集します。
- **フロントマター:** 各ファイルの冒頭にある設定を確認してください。
  - `title`: ページタイトル
  - `weight`: メニューの表示順（数値が小さいほど上）
  - `draft`: `true` の間は本番サイトに表示されません。

### 3. プレビューと確認
ローカルサーバー（`hugo server`）で表示を確認し、リンク切れや画像が表示されているかチェックしてください。

---

## 運用ルール
詳細な開発フローや技術仕様については [GEMINI.md](./GEMINI.md) を参照してください。
- 変更はブランチ（`feat/` または `fix/`）を作成し、Pull Request を出してください。
- `main` ブランチへのマージにより、自動的に本番サーバーへデプロイされます。
