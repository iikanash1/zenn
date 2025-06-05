---
title: "ターミナルで技術記事を投稿してみた：Git/GitHub活用の実践記録"
emoji: "📝"
type: "tech"
topics: ["git", "github", "terminal", "zenn", "初心者"]
published: true
---

# ターミナルで技術記事を投稿してみた体験記

## はじめに

最近、技術記事の執筆・管理をすべてターミナル上で行う方法を試してみました。この記事では、その過程で学んだことや、つまずいたポイントを共有したいと思います。

## 1. 環境構築

### 必要なツール
- Git（バージョン管理）
- Node.js（Zenn CLIの実行に必要）
- テキストエディタ（VSCode等）
- GitHub（記事の管理用）

### Zenn CLIのセットアップ
1. プロジェクトの初期化
   ```bash
   npm init --yes
   ```

2. Zenn CLIのインストール
   ```bash
   npm install zenn-cli
   ```

3. Zennの初期化
   ```bash
   npx zenn init
   ```

## 2. 記事作成のワークフロー

### ブランチの作成と切り替え
```bash
git checkout -b add-new-article
```

### 記事の作成
1. 新規記事の作成
   ```bash
   npx zenn new:article
   ```

2. Markdownでの執筆
   - フロントマターの設定
   - 本文の執筆
   - 画像の配置

### プレビューの確認
```bash
npx zenn preview
```
- http://localhost:8000 でプレビュー確認
- リアルタイムでの更新反映

## 3. Git/GitHubでの管理

### コミットの作成
```bash
git add articles/記事ファイル名.md
git commit -m "feat: 新しい記事を追加"
```

### リモートへのプッシュ
```bash
git push origin ブランチ名
```

### プルリクエストの作成
1. GitHubでプルリクエストを作成
2. レビュー依頼（必要に応じて）
3. マージして公開

## 4. つまずきやすいポイント

### ファイル構成の問題
- 記事は必ず `articles` ディレクトリに配置
- 画像は `images` ディレクトリに配置
- フロントマターの形式に注意

### Git操作での注意点
- ブランチ名の付け方
- コミットメッセージの書き方
- コンフリクトの解決方法

### node_modulesの扱い
- `.gitignore` での除外設定
- パッケージのバージョン管理
- 依存関係の解決

## 5. 効率化のためのTips

### エイリアスの設定
```bash
# .bashrcや.zshrcに追加
alias zp='npx zenn preview'
alias za='npx zenn new:article'
```

### VSCodeの拡張機能活用
- Markdown All in One
- markdownlint
- Git Graph

### GitHub CLIの活用
```bash
# プルリクエスト作成
gh pr create --title "新規記事追加" --body "技術記事を追加しました"
```

## 6. トラブルシューティング

### よくある問題と解決策
1. プレビューが表示されない
   - ポート番号の確認
   - プロセスの再起動

2. プッシュが失敗する
   - リモートの状態確認
   - 認証情報の確認

3. 画像が表示されない
   - パスの指定方法
   - 画像ファイルの配置場所

## まとめ

ターミナルでの記事管理は最初は慣れが必要ですが、以下のメリットがあります：

1. バージョン管理が容易
2. チーム作業との親和性
3. 自動化の可能性
4. コマンドラインスキルの向上

また、この経験を通じて以下のことを学びました：

- Git/GitHubの実践的な使い方
- コマンドラインツールの活用
- 効率的な文書管理の方法

これから同じように記事管理を始める方の参考になれば幸いです。 