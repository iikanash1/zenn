---
title: "RHEL/CentOS系ローカルYUM/DNFリポジトリサーバーの構築と運用ガイド"
emoji: "🐧"
type: "tech"
topics: ["linux", "rhel", "centos", "yum", "dnf", "infrastructure"]
published: true
---

# はじめに

インターネットに接続できないクローズドな環境や、多数のサーバーに同じバージョンのパッケージを安定して配布したい環境では、ローカルにYUM/DNFリポジトリサーバー（パッチサーバー）を構築することが不可欠です。

この記事では、RHEL (CentOS, Rocky Linuxなど) 系のLinuxディストリビューション向けに、ローカルリポジトリサーバーを構築し、最新のセキュリティパッチやパッケージを同期（アップデート）し続けるための基本的な手順を解説します。

# なぜローカルリポジトリが必要か？

- **セキュリティ**: インターネットに直接接続できないサーバー群にも、セキュリティパッチを適用できます。
- **一貫性**: 全サーバーで同じバージョンのパッケージを利用させることができ、環境の差異による問題を防止します。
- **帯域幅の節約**: 各サーバーが個別にパッケージをダウンロードする必要がなくなり、ネットワーク帯域を節約できます。

# 前提条件

- RHEL系のLinuxサーバー (RHEL, CentOS, Rocky Linuxなど)
- 公式リポジトリのパッケージをすべて保存できる十分なディスク容量（数百GB以上を推奨）
- `sudo`権限を持つユーザー

---

# 手順1: 必要なツールのインストール

まず、リポジトリの同期と配信用に必要なツールをインストールします。

```bash
# dnf-utils: reposyncコマンドを提供
# createrepo_c: リポジトリのメタデータを作成
# httpd: パッケージを配信するためのWebサーバー
sudo dnf install -y dnf-utils createrepo_c httpd
```

# 手順2: Webサーバーの設定とディレクトリ作成

パッケージを格納するディレクトリを作成し、Webサーバー経由でアクセスできるように設定します。

```bash
# パッケージを格納するディレクトリを作成 (RHEL 9の例)
sudo mkdir -p /var/www/html/repos/rhel9/baseos
sudo mkdir -p /var/www/html/repos/rhel9/appstream

# Webサーバーを起動し、自動起動を有効化
sudo systemctl enable --now httpd

# ファイアウォールでHTTPアクセスを許可
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
```

# 手順3: リポジトリの同期 (サーバーのアップデート)

`reposync`コマンドを使い、公式リポジトリからローカルディレクトリにパッケージをダウンロードします。これがパッチサーバーを「アップデート」する中心的な作業です。

```bash
# RHEL 9の主要なリポジトリを同期する例
# baseosリポジトリの同期
sudo reposync --repo=baseos --download-path=/var/www/html/repos/rhel9/

# appstreamリポジトリの同期
sudo reposync --repo=appstream --download-path=/var/www/html/repos/rhel9/
```
**注意:** `reposync`は、指定したリポジトリの全パッケージをダウンロードするため、非常に時間がかかり、多くのディスク容量を消費します。

# 手順4: リポジトリメタデータの作成・更新

パッケージを同期した後、クライアントがリポジトリとして認識できるように`createrepo_c`コマンドでメタデータを作成します。この作業は、`reposync`を実行するたびに必要です。

```bash
# 同期した各リポジトリのメタデータを作成
sudo createrepo_c /var/www/html/repos/rhel9/baseos/
sudo createrepo_c /var/www/html/repos/rhel9/appstream/
```

# 手順5: 同期作業の自動化

ここまでの手順をスクリプト化し、`cron`で定期的に実行することで、パッチサーバーを自動で最新の状態に保ちます。

1.  **自動化スクリプトの作成**

    ```bash
    # /usr/local/bin/update_repo.sh などの名前でファイルを作成
    sudo vi /usr/local/bin/update_repo.sh
    ```

    以下の内容を貼り付けます。

    ```sh
    #!/bin/bash

    REPO_PATH="/var/www/html/repos/rhel9"
    LOG_FILE="/var/log/repo_sync.log"

    echo "--- Starting repo sync at $(date) ---" | tee -a $LOG_FILE

    # リポジトリを同期
    sudo reposync --repo=baseos --download-path=$REPO_PATH | tee -a $LOG_FILE
    sudo reposync --repo=appstream --download-path=$REPO_PATH | tee -a $LOG_FILE

    # メタデータを更新
    sudo createrepo_c $REPO_PATH/baseos/ | tee -a $LOG_FILE
    sudo createrepo_c $REPO_PATH/appstream/ | tee -a $LOG_FILE

    echo "--- Finished repo sync at $(date) ---" | tee -a $LOG_FILE
    ```

2.  **スクリプトに実行権限を付与**

    ```bash
    sudo chmod +x /usr/local/bin/update_repo.sh
    ```

3.  **cronに登録**

    毎日深夜2時にスクリプトを実行する例です。

    ```bash
    # crontabを開いて編集
    sudo crontab -e

    # 以下の行を追記
    0 2 * * * /usr/local/bin/update_repo.sh
    ```

# 手順6: クライアントの設定

最後に、クライアントサーバーがこのローカルリポジトリサーバーを参照するように設定します。

1.  **リポジトリ設定ファイルの作成**

    クライアントサーバー上で、`/etc/yum.repos.d/local.repo` というファイルを作成します。

    ```ini
    [local-baseos]
    name=Local RHEL9 BaseOS
    baseurl=http://<リポジトリサーバーのIPアドレス>/repos/rhel9/baseos
    gpgcheck=0
    enabled=1

    [local-appstream]
    name=Local RHEL9 AppStream
    baseurl=http://<リポジトリサーバーのIPアドレス>/repos/rhel9/appstream
    gpgcheck=0
    enabled=1
    ```
    **`<リポジトリサーバーのIPアドレス>`** の部分を、構築したサーバーのIPアドレスに書き換えてください。
    `gpgcheck=0`はGPGキーの検証を無効にする設定です。セキュリティを重視する場合は、公式のGPGキーをクライアントにインポートし、`gpgcheck=1`としてください。

2.  **設定の確認**

    クライアントサーバーで以下のコマンドを実行し、新しいローカルリポジトリが認識されていることを確認します。

    ```bash
    # キャッシュをクリア
    sudo dnf clean all

    # リポジトリ一覧を再読み込みして確認
    sudo dnf repolist
    ```
    `repo id`に`local-baseos`や`local-appstream`が表示されれば成功です。

# まとめ

以上の手順で、RHEL系のローカルリポジトリサーバーを構築し、定期的にアップデート（同期）する仕組みが完成しました。これにより、セキュアで一貫性のあるパッケージ管理が実現できます。
