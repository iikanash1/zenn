---
title: " AWS CLI入門：コマンドラインでAWSを自由自在に操る"
emoji: "🐈‍⬛"
type: "tech"
topics: ["aws", "インフラ", "クラウド", "devops", "初心者"]
published: true
---

# AWS CLI入門：コマンドラインでAWSを自由自在に操る

AWS Management Consoleは直感的で便利ですが、定型作業の自動化や大量のリソースの一括操作には限界があります。そこで活躍するのが**AWS Command Line Interface (AWS CLI)** です。

この記事では、AWS CLIの基本から、日々の業務を効率化するための便利な使い方までを網羅的に解説します。

## 目次
1.  [AWS CLIとは？](#1-aws-cliとは)
2.  [導入方法](#2-導入方法)
    *   [インストール](#インストール)
    *   [初期設定 (aws configure)](#初期設定-aws-configure)
3.  [基本的な使い方](#3-基本的な使い方)
    *   [基本構文](#基本構文)
    *   [主要サービスの操作例](#主要サービスの操作例)
4.  [知っておくと便利な機能](#4-知っておくと便利な機能)
    *   [プロファイル (複数アカウントの切り替え)](#プロファイル-複数アカウントの切り替え)
    *   [出力形式の制御 (`--output`)](#出力形式の制御---output)
    *   [結果の絞り込み (`--query`)](#結果の絞り込み---query)
    *   [ドライラン (`--dry-run`)](#ドライラン---dry-run)
5.  [まとめ](#5-まとめ)

## 1. AWS CLIとは？
AWS CLIは、ターミナル（コマンドライン）からAWSの各種サービスを操作するための統一されたツールです。これを使うことで、以下のようなメリットがあります。

*   **自動化**: シェルスクリプトと組み合わせることで、リソースの作成・削除・設定変更といった一連の作業を自動化できます。CI/CDパイプラインへの組み込みも容易です。
*   **効率化**: マネジメントコンソールで何度もクリックが必要な繰り返し作業も、コマンド一つで実行できます。
*   **再現性**: コマンドやスクリプトとして手順を保存しておくことで、誰が実行しても同じ環境を正確に再現できます。
*   **一括操作**: 数百のS3オブジェクトやEC2インスタンスに対して、一度に同じ操作を適用する、といったことが得意です。

コンソールでできる操作のほとんどは、AWS CLIでも実行可能です。

## 2. 導入方法
### インストール
AWS CLIは、macOS, Linux, Windowsに対応しています。お使いのOSに合わせた最新のインストール方法は、公式ドキュメントを参照するのが最も確実です。

[公式ドキュメント: AWS CLI の最新バージョンをインストールまたは更新します。](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)

インストールが完了したら、バージョンを確認してみましょう。

```bash
aws --version
# 例: aws-cli/2.15.0 Python/3.11.6 Linux/5.15.90.1-microsoft-standard-WSL2...
```

バージョン情報が表示されれば、インストールは成功です。

### 初期設定 (aws configure)
次に、AWS CLIがあなたのAWSアカウントにアクセスするための認証情報を設定します。この設定には、IAMユーザーの**アクセスキーID**と**シークレットアクセスキー**が必要です。

> **⚠️ 注意:**
> アクセスキーは非常に強力な権限を持つため、取り扱いには細心の注意を払ってください。絶対に公開リポジトリ（GitHubなど）にコミットしないでください。

以下のコマンドを実行すると、対話形式で設定が始まります。

```bash
aws configure
```

各項目を順に入力します。
*   **AWS Access Key ID**: `[あなたのアクセスキーID]`
*   **AWS Secret Access Key**: `[あなたのシークレットアクセスキー]`
*   **Default region name**: よく利用するリージョン名 (例: `ap-northeast-1` (東京))
*   **Default output format**: 出力形式 ( `json`, `yaml`, `text`, `table` から選択。通常は `json` がおすすめです)

入力した情報は、ホームディレクトリ配下の `~/.aws/credentials` と `~/.aws/config` というファイルに保存されます。

## 3. 基本的な使い方
### 基本構文
AWS CLIのコマンドは、一貫した構文を持っています。

```
aws <サービス名> <操作名> [パラメータ]
```

| 要素 | 説明 | 例 |
| :--- | :--- | :--- |
| **`aws`** | すべてのコマンドの接頭辞 | `aws` |
| **`<サービス名>`** | 操作対象のAWSサービス | `s3`, `ec2`, `iam` |
| **`<操作名>`** | 実行したいアクション | `ls`, `describe-instances`, `list-users` |
| **`[パラメータ]`** | 操作に必要な追加情報 | `--bucket-name my-bucket`, `--instance-ids i-12345` |

### 主要サービスの操作例

#### Amazon S3
S3は `aws s3` コマンドで直感的に操作できます。

```bash
# S3バケットの一覧を表示
aws s3 ls

# 特定のバケット内のオブジェクト一覧を表示
aws s3 ls s3://my-example-bucket

# ローカルファイルをS3にアップロード
aws s3 cp local-file.txt s3://my-example-bucket/

# S3からファイルをダウンロード
aws s3 cp s3://my-example-bucket/remote-file.txt .
```

#### Amazon EC2
EC2インスタンスの情報を取得したり、起動・停止したりできます。

```bash
# EC2インスタンスの一覧を取得 (JSON形式で詳細情報が出力される)
aws ec2 describe-instances

# 特定のインスタンスを停止
aws ec2 stop-instances --instance-ids i-0123456789abcdef0

# 特定のインスタンスを起動
aws ec2 start-instances --instance-ids i-0123456789abcdef0
```

#### AWS IAM
IAMユーザーやロールの情報を取得します。

```bash
# IAMユーザーの一覧を取得
aws iam list-users

# 現在の認証情報に紐づくユーザー名やARNを確認
aws sts get-caller-identity
```

## 4. 知っておくと便利な機能

### プロファイル (複数アカウントの切り替え)
複数のAWSアカウント（本番環境と開発環境など）を使い分ける場合、`--profile` オプションが非常に便利です。

まず、`aws configure` に `--profile` を付けて、新しいプロファイルを追加します。

```bash
# "develop" という名前でプロファイルを作成
aws configure --profile develop

# 各種情報を入力...
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

コマンド実行時に、このプロファイルを指定します。

```bash
# "develop" アカウントのS3バケット一覧を取得
aws s3 ls --profile develop
```
これで、コマンドごとに認証情報を簡単かつ安全に切り替えることができます。

### 出力形式の制御 (`--output`)
`aws configure` で設定したデフォルトの出力形式を、コマンド実行時に `--output` オプションで上書きできます。

`table` 形式は、人間が素早く確認したい場合に非常に見やすいです。

```bash
# IAMユーザーの一覧をテーブル形式で表示
aws iam list-users --output table
```

```
---------------------------------------------------------------------------------
|                                   ListUsers                                   |
+-----------------------------------------------------+-------------------------+
|                        Users                        |                         |
+-----------------------------------------------------+-------------------------+
|  Arn                                                |  aws:iam::12345:user/user-a |
|  CreateDate                                         |  2023-01-01T00:00:00Z   |
|  Path                                               |  /                      |
|  UserId                                             |  AIDA...                |
|  UserName                                           |  user-a                 |
+-----------------------------------------------------+-------------------------+
|  Arn                                                |  aws:iam::12345:user/user-b |
...
```

### 結果の絞り込み (`--query`)
`json` 形式の膨大な出力から、必要な情報だけを抜き出すには `--query` オプションが強力です。これはJMESPathというクエリ言語を使用します。

例えば、`ec2 describe-instances` の結果から、**インスタンスID**と**現在の状態**だけを抜き出してみましょう。

```bash
aws ec2 describe-instances --query "Reservations[].Instances[].{ID:InstanceId, State:State.Name}" --output table
```
実行結果：
```
-------------------------------------
|        DescribeInstances          |
+----------------------+------------+
|          ID          |   State    |
+----------------------+------------+
|  i-0123456789abcdef0 |  running   |
|  i-fedcba9876543210f |  stopped   |
+----------------------+------------+
```
これにより、必要な情報だけを簡潔に表示でき、スクリプトでの後続処理も楽になります。

### ドライラン (`--dry-run`)
リソースを停止・削除するような破壊的な操作を行う前に、コマンドが成功するかどうかを事前にテストできるのが `--dry-run` オプションです。

このオプションを付けると、**実際には操作は実行されず**、権限が十分か、パラメータが正しいかといったチェックのみが行われます。

```bash
# インスタンスを "実際に停止せず" に、停止コマンドが成功するか試す
aws ec2 stop-instances --instance-ids i-0123456789abcdef0 --dry-run
```
成功する場合、`DryRunOperation` というメッセージが返されます。権限不足などで失敗する場合は、エラーメッセージが表示されるため、本番環境でのオペレーションミスを防ぐのに役立ちます。

## 5. まとめ
AWS CLIは、AWSを効率的かつ確実に管理するための必須ツールです。最初は覚えることが多いと感じるかもしれませんが、まずは `ls` や `describe-*` のような参照系のコマンドから試してみてください。

慣れてくると、日々の面倒な作業をスクリプト化し、大幅な時間短縮とミスの削減を実現できるようになります。この記事を参考に、ぜひコマンドラインでのAWS操作に挑戦してみてください。

---
**参考リンク**
*   [AWS CLI Command Reference (公式リファレンス)](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
*   [JMESPath Tutorial ( `--query` の使い方)](https://jmespath.org/tutorial.html)