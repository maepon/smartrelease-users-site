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

### 2. リポジトリのクローン
テーマ（git submodule）を含めて一括でクローンします。

```bash
git clone --recursive https://github.com/maepon/smartrelease-users-site.git
cd smartrelease-users-site
```
※ すでにクローン済みの場合は `git submodule update --init --recursive` を実行してください。

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

日常的な記事の更新や修正は、以下のフローで行います。

- **コミット署名**: すべてのコミットに **署名（GPGまたはSSH）が必須** です。署名がないコミットは GitHub 上でマージできません。
1. **ブランチ作成**: `main` ブランチから作業用ブランチ（`feat/xxx`, `fix/xxx`, `entry/xxx`）を作成し、変更を Push します。
2. **Pull Request (PR)**: GitHub 上で `main` ブランチに向けて PR を作成します。
3. **レビューとマージ**: コードオーナー1名以上の承認を得て **Squash and merge** でマージします。
4. **自動デプロイ**: マージ完了と同時に GitHub Actions が自動で走り、SRUのテストサーバーへデプロイされます。

---

## 注意事項
- **テーマの変更**: `themes/book` 配下のファイルを直接編集しないでください。
- **管理者権限**: 緊急時や軽微な修正については、コードオーナーの判断で直接マージを行う場合があります。
