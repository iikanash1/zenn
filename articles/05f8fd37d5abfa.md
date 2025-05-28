---
title: "Gitの事故を防ぐ！チーム開発のための安全対策ガイド"
emoji: "🛡️"
type: "tech"
topics: ["git", "github", "開発手法", "チーム開発", "事故防止"]
published: true
---

# Gitの事故を防ぐ！チーム開発のための安全対策ガイド

この記事では、Gitを使用したチーム開発において発生しがちな事故とその防止策について解説します。特に、mainブランチへの誤pushや重要なコードの削除など、よくある事故の防止方法に焦点を当てています。

## 目次
1. よくあるGitの事故とその影響
2. 事前の予防策
3. ブランチ保護の設定
4. Git hooks の活用
5. チーム全体での取り組み
6. 事故が起きた時の対処法

## 1. よくあるGitの事故とその影響

### 主な事故パターン

1. mainブランチへの直接push
2. 誤ったブランチへのマージ
3. 重要なコードの誤削除
4. 機密情報の誤コミット
5. 大きなバイナリファイルの誤追加

### 事故の影響

- チーム全体の開発の停滞
- 本番環境への影響
- セキュリティリスク
- データ損失

## 2. 事前の予防策

### ローカル環境での設定

```bash
# 現在のブランチ名をプロンプトに表示（.bashrcや.zshrcに追加）
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
export PS1="\u@\h \w\$(parse_git_branch)\$ "

# エイリアスの設定
git config --global alias.st status
git config --global alias.br branch
git config --global alias.co checkout

# プッシュ先の制限
git config --global push.default current
```

### 必須の.gitignore設定

```gitignore
# 機密情報
.env
*.key
credentials.json

# IDE/エディタ固有のファイル
.idea/
.vscode/
*.swp

# ビルド成果物
/dist
/build
node_modules/

# ログファイル
*.log
npm-debug.log*
```

## 3. ブランチ保護の設定

### GitHub上での設定

```yaml
# ブランチ保護ルールの例
protected_branches:
  main:
    required_status_checks:
      strict: true
    required_pull_request_reviews:
      required_approving_review_count: 2
    restrictions:
      users: ["admin1", "admin2"]
    enforce_admins: true
```

### 重要な設定項目

1. レビュー必須化
2. ステータスチェック必須化
3. 管理者にも制限を適用
4. 直接プッシュの禁止

## 4. Git hooks の活用

### pre-commit フック

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 現在のブランチ名を取得
branch="$(git rev-parse --abbrev-ref HEAD)"

# mainブランチへの直接コミットを防止
if [ "$branch" = "main" ]; then
  echo "mainブランチへの直接コミットは禁止されています"
  exit 1
fi

# 機密情報のチェック
if git diff --cached | grep -i "password\|secret\|api_key"; then
  echo "機密情報らしき文字列が含まれています"
  exit 1
fi
```

### pre-push フック

```bash
#!/bin/bash
# .git/hooks/pre-push

# mainブランチへのプッシュを防止
if [ "$(git rev-parse --abbrev-ref HEAD)" = "main" ]; then
  echo "mainブランチへの直接プッシュは禁止されています"
  exit 1
fi

# 大きなファイルのチェック
git diff --cached --name-only | while read file; do
  if [ -f "$file" ]; then
    size=$(stat -f%z "$file")
    if [ $size -gt 5000000 ]; then
      echo "$file is larger than 5MB"
      exit 1
    fi
  fi
done
```

## 5. チーム全体での取り組み

### ブランチ運用ルール

1. ブランチ命名規則
```
feature/  # 新機能開発
bugfix/   # バグ修正
hotfix/   # 緊急の修正
release/  # リリース準備
```

2. 作業開始時のルール
```bash
# 必ず最新のmainから作業ブランチを作成
git checkout main
git pull origin main
git checkout -b feature/new-function
```

3. 作業中のルール
```bash
# 定期的にmainの変更を取り込む
git checkout feature/new-function
git fetch origin
git merge origin/main
```

### コードレビュープロセス

1. セルフレビュー
```bash
# 変更内容の確認
git diff main...feature/new-function
```

2. レビュー依頼前のチェックリスト
- [ ] 不要なファイルが含まれていないか
- [ ] 機密情報が含まれていないか
- [ ] コードフォーマットは適切か
- [ ] テストは通っているか

## 6. 事故が起きた時の対処法

### 誤ってコミットした場合

```bash
# 直前のコミットを取り消し（変更は保持）
git reset --soft HEAD^

# コミット自体を完全に取り消し
git reset --hard HEAD^

# 特定のファイルを前のバージョンに戻す
git checkout HEAD^ -- path/to/file
```

### 誤ってプッシュした場合

```bash
# 強制プッシュで戻す（チーム内で要相談）
git reset --hard HEAD^
git push -f origin branch-name

# revertコミットで打ち消す（推奨）
git revert HEAD
git push origin branch-name
```

### 機密情報を誤ってプッシュした場合

1. 即座にパスワード等の変更
2. コミット履歴からの完全削除
```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/sensitive-file" \
  --prune-empty --tag-name-filter cat -- --all
```

## まとめ

Gitの事故を防ぐためのポイント：

1. 適切なブランチ戦略の採用
2. 自動チェックの導入
3. チーム内でのルール共有
4. 定期的な研修・勉強会の実施

これらの対策を実施することで、多くの事故を未然に防ぐことができます。また、事故が発生した場合でも、迅速かつ適切な対応が可能になります。

## 参考リンク
- [Git公式ドキュメント](https://git-scm.com/doc)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)
- [Git Hooks Documentation](https://git-scm.com/docs/githooks)
