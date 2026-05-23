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

## 運用ルールとデプロイの流れ

詳細な開発フローや技術仕様については [GEMINI.md](./GEMINI.md) を参照してください。
日常的な記事の更新や修正は、以下のフローで行います。

1. **ブランチ作成**: `main` ブランチから作業用ブランチ（`feat/xxx` や `fix/xxx`）を作成し、変更を Push します。
2. **Pull Request (PR)**: GitHub 上で `main` ブランチに向けて PR を作成します（変更内容と確認方法を記載）。
3. **レビューとマージ**: 1名以上の承認を得て **Squash and merge** でマージします（Code Owner は緊急時等バイパス可）。
4. **自動デプロイ**: マージ完了と同時に GitHub Actions が自動で走り、本番サーバーへデプロイされます。
