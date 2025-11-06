---
title: "CursorのGUIでGitHubプルリクエストを作成する一連の操作"
emoji: "🤷‍♂️"
type: "tech"
topics: ["git", "github", "cursor", "vscode", "pullrequest"]
published: false
---

## CursorのGUIでGitHubプルリクエストを作成する一連の操作

CursorはVS Codeをベースとした高機能エディタであり、GitHubとの連携をGUIで直感的に行うことができます。本記事では、Cursorのソース管理（Git）拡張機能を活用し、新しいブランチの作成から変更のコミット、プッシュ、そしてプルリクエストの作成までの一連のワークフローを解説します。

### 前提条件

*   Cursorがインストールされており、GitHubアカウントで認証済みであること（[CursorでGitHubアカウントにログインする方法](先ほど説明したログイン方法へのリンクをここに挿入)）。
*   作業対象のGitリポジトリがローカルにクローンされていること。
*   Gitのユーザー名とメールアドレスが設定済みであること。
    ```bash
    git config --global user.name "Your GitHub Username"
    git config --global user.email "your.email@example.com"
    ```

### 1. ブランチの作成

新しい機能開発やバグ修正は、既存のコードベース（通常は `main` または `master` ブランチ）に直接変更を加えるのではなく、新しいブランチを作成して行います。

1.  **ソース管理ビューを開く**:
    Cursorの左側にあるアクティビティバーから、**ソース管理アイコン**（Gitのアイコン）をクリックします。

    