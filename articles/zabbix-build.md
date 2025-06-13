---
title: "Zabbix構築ガイド：オープンソース監視ツールの導入から初期設定まで"
emoji: "🎶"
type: "tech"
topics: ["zabbix", "linux", "server", "インフラ", "ネットワーク"]
published: true
---

## はじめに

Zabbixは、サーバー、ネットワーク機器、アプリケーションなど、ITインフラのあらゆる要素を集中監視できる強力なオープンソースの統合監視ソフトウェアです。本記事では、Zabbixサーバーを構築し、監視対象のサーバーを追加するまでの基本的な手順を解説します。

### この記事のゴール
*   Zabbixサーバーをインストールし、Webインターフェースにアクセスできる状態にする。
*   監視対象のサーバーにZabbixエージェントを導入し、基本的な監視を開始する。

### 対象環境
*   **OS**: Rocky Linux 9
*   **Zabbix Version**: 6.0 LTS (または最新安定版)
*   **Webサーバー**: Apache
*   **データベース**: MariaDB

---

## 1. Zabbixのアーキテクチャ概要

Zabbixは主に以下のコンポーネントで構成されています。

| コンポーネント | 役割 |
| :--- | :--- |
| **Zabbixサーバー** | 監視データ収集、トリガー評価、アラート通知などを行う中心的なプロセス。 |
| **データベース** | Zabbixサーバーが収集した監視データや設定情報を格納します。(MySQL/MariaDB, PostgreSQLなど) |
| **Webインターフェース** | PHPで書かれたWeb UI。監視データやグラフの可視化、各種設定を行います。 |
| **Zabbixエージェント** | 監視対象のサーバーにインストールし、CPU使用率やメモリ使用量などのローカルリソース情報を収集してZabbixサーバーに送信します。 |

---

## 2. Zabbixサーバーの構築手順

ここからは、Zabbixサーバーを構築する具体的な手順を解説します。

### Step 1: Zabbixリポジトリの追加

まず、Zabbix公式リポジトリをシステムに登録します。

```bash
# Zabbix 6.0 LTSリポジトリをインストール
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm

# パッケージキャッシュをクリア
dnf clean all
```

### Step 2: Zabbixコンポーネントのインストール

Zabbixサーバー、Webインターフェース、エージェントなど、必要なパッケージをインストールします。

```bash
# Zabbixサーバー(MySQL用), Webフロントエンド, Apache設定, SQLスクリプト, エージェントをインストール
dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

### Step 3: データベースのインストールと設定

Zabbixの設定情報や監視データを保存するためのMariaDBをインストール・設定します。

1.  **MariaDBのインストール**
    ```bash
    dnf install mariadb-server
    ```

2.  **MariaDBの起動と自動起動設定**
    ```bash
    systemctl enable --now mariadb
    ```

3.  **データベースの初期セキュリティ設定**
    以下のコマンドを実行し、対話形式で設定を進めます。rootパスワードの設定、匿名ユーザーの削除などを行ってください。
    ```bash
    mysql_secure_installation
    ```

4.  **Zabbix用データベースとユーザーの作成**
    `mysql`コマンドでMariaDBにログインし、データベースと専用ユーザーを作成します。
    ```bash
    # MariaDBにrootでログイン (先ほど設定したパスワードを入力)
    mysql -u root -p

    # Zabbix用データベースを作成 (文字コードはutf8mb4を推奨)
    MariaDB [(none)]> create database zabbix character set utf8mb4 collate utf8mb4_bin;

    # Zabbix用ユーザーを作成し、権限を付与 (パスワードは'password'の部分を強力なものに変更してください)
    MariaDB [(none)]> create user zabbix@localhost identified by 'password';
    MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost;

    # 設定を反映
    MariaDB [(none)]> flush privileges;

    # ログアウト
    MariaDB [(none)]> quit;
    ```
    > **重要**: `password` の部分は、必ず推測されにくい強力なパスワードに変更してください。このパスワードは後の手順で使います。

### Step 4: Zabbix Serverの設定

1.  **初期スキーマとデータのインポート**
    Zabbixがデータベースを利用するためのテーブル定義などをインポートします。
    ```bash
    zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p zabbix
    ```
    実行後、MariaDBで設定したZabbixユーザーのパスワードを入力してください。

2.  **Zabbixサーバー設定ファイルの編集**
    Zabbixサーバーがデータベースに接続できるよう、設定ファイルを編集します。
    ```bash
    vi /etc/zabbix/zabbix_server.conf
    ```
    `DBPassword`の項目を探し、コメントアウト（`#`）を解除して、Step 3で設定したパスワードを記述します。
    ```conf
    # Before
    # DBPassword=

    # After (例)
    DBPassword=password 
    ```

### Step 5: サービスの起動と自動起動設定

必要なサービスを起動し、OS起動時に自動で立ち上がるように設定します。

```bash
systemctl restart zabbix-server zabbix-agent httpd php-fpm
systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

### Step 6: ファイアウォールの設定

Zabbixが使用するポートをファイアウォールで許可します。

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=10051/tcp --permanent  # Zabbix Serverポート
firewall-cmd --add-port=10050/tcp --permanent  # Zabbix Agentポート
firewall-cmd --reload
```
> HTTPSを使用する場合は `--add-service=https` も追加してください。

### Step 7: Webインターフェースによる初期設定

ブラウザを開き、以下のURLにアクセスします。
`http://<ZabbixサーバーのIPアドレス>/zabbix`

1.  **Welcome screen**: `Next step`をクリックします。
2.  **Check of pre-requisites**: すべての項目が "OK" になっていることを確認し、`Next step` をクリックします。（もしNG項目があれば、PHPの拡張機能不足などが考えられます）
3.  **Configure DB connection**: データベース接続設定を入力します。
    *   **Database type**: `MySQL`
    *   **Database host**: `localhost`
    *   **Database port**: `3306` (または空欄)
    *   **Database name**: `zabbix`
    *   **User**: `zabbix`
    *   **Password**: Step 3-4で設定したパスワード
    入力後、`Next step`をクリックします。
4.  **Zabbix server details**: Zabbixサーバーの情報を入力します。通常はデフォルトのままで問題ありません。`Next step`をクリックします。
5.  **Pre-installation summary**: 設定内容を確認し、`Next step`をクリックします。
6.  **Install**: インストールが完了したら、`Finish`をクリックします。

これでZabbixのWebインターフェースにログイン画面が表示されます。
*   **初期ユーザー名**: `Admin`
*   **初期パスワード**: `zabbix`

ログイン後、必ずパスワードを変更しておきましょう。

---

## 3. 監視対象ホストの追加 (Zabbixエージェント)

次に、監視したいサーバー（Linux）にZabbixエージェントを導入し、Zabbixサーバーにホストとして登録します。

### 監視対象サーバーでの作業

1.  **Zabbixリポジトリの追加**
    ```bash
    # Rocky Linux 9の場合
    rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-6.0-4.el9.noarch.rpm
    dnf clean all
    ```
2.  **Zabbixエージェントのインストール**
    ```bash
    dnf install zabbix-agent
    ```
3.  **設定ファイルの編集**
    ```bash
    vi /etc/zabbix/zabbix_agentd.conf
    ```
    以下の3項目を環境に合わせて変更します。
    ```conf
    # ZabbixサーバーのIPアドレスを指定
    Server=<ZabbixサーバーのIPアドレス>

    # アクティブチェック用にもZabbixサーバーのIPアドレスを指定
    ServerActive=<ZabbixサーバーのIPアドレス>

    # Zabbix Web UIで表示されるホスト名 (一意である必要があります)
    # デフォルトではシステムホスト名が使われますが、明示的に指定することを推奨します。
    Hostname=<このサーバーのホスト名>
    ```
4.  **エージェントの起動と自動起動設定**
    ```bash
    systemctl restart zabbix-agent
    systemctl enable zabbix-agent
    ```
5.  **ファイアウォールの設定**
    ```bash
    # Zabbixエージェントの待ち受けポートを許可
    firewall-cmd --add-port=10050/tcp --permanent
    firewall-cmd --reload
    ```

### ZabbixサーバーのWeb UIでの作業

1.  左メニューの [設定] -> [ホスト] をクリックします。
2.  右上の [ホストの作成] をクリックします。
3.  **ホスト** タブで、以下の情報を入力します。
    *   **ホスト名**: 監視対象サーバーの`zabbix_agentd.conf`で設定した`Hostname`と**同じ名前**を入力します。
    *   **テンプレート**: この後設定します。
    *   **ホストグループ**: 「Linux servers」など、適切なグループを選択（または新規作成）します。
    *   **インターフェース**: `追加`をクリックし、`エージェント`を選択します。
        *   **IPアドレス**: 監視対象サーバーのIPアドレスを入力します。
4.  **テンプレート** タブをクリックします。
    *   **新規テンプレートをリンク** の入力欄で `Linux by Zabbix agent` を検索し、選択します。このテンプレートには、CPU、メモリ、ディスク、ネットワークなど一般的なLinux監視項目が多数含まれています。
5.  最後に [追加] ボタンをクリックします。

ホスト一覧画面に戻り、追加したホストの「可用性」列の「ZBX」アイコンが、数分後に緑色に変われば、Zabbixサーバーとエージェントの通信が正常に行われています。

---

## まとめ

この記事では、Zabbixサーバーの基本的な構築手順と、監視対象ホストを追加する方法を解説しました。これで、Zabbixによる監視の第一歩は完了です。

ここからさらに、
*   **ダッシュボード**で監視状況をグラフィカルに確認する
*   障害発生時にメールなどで**通知を飛ばすアクションを設定**する
*   独自の監視項目を追加するために**テンプレートをカスタマイズ**する

など、Zabbixの強力な機能を活用していくことができます。ぜひ公式ドキュメントなども参考に、より高度な監視環境を構築してみてください。

- [Zabbix Documentation](https://www.zabbix.com/documentation/current/ja)