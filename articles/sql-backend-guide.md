---
title: "【SQL入門】これだけは押さえたい！Web開発者のためのSQL基本と実践テクニック"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sql", "database", "backend", "初心者"]
published: true
---

## はじめに

Web開発の世界に足を踏み入れると、必ずと言っていいほど出会うのが **SQL** です。
「データベースを操作するための言語」と聞いても、最初は「何から学べばいいの？」「なんだか難しそう…」と感じるかもしれません。

この記事では、そんなSQL初心者の方向けに、**これだけは押さえておきたい基本**から、一歩進んだ**実践的なテクニック**までを、具体的なコード例と共に分かりやすく解説します。
この記事を読み終える頃には、SQLの基本的な読み書きができるようになり、自信を持ってデータベースを扱えるようになるはずです！

## 対象読者

- SQLをこれから学び始めるプログラミング初学者の方
- Web開発者を目指していて、バックエンドの基礎を固めたい方
- なんとなくSQLを使っているけど、一度しっかり基礎を復習したい方

## 1. SQLとは？ データベースとの対話言語 💬

SQL（Structured Query Language）は、データベースと「対話」するための言語です。
私たちが普段使っているWebサービスやアプリの裏側には、ユーザー情報や投稿データなどを保存しておくための**データベース(DB)**が存在します。

SQLを使うことで、このデータベースに対して以下のような命令ができます。

- **データの取得 (SELECT)**:「このユーザーの情報を教えて」
- **データの追加 (INSERT)**:「新しいユーザーを登録して」
- **データの更新 (UPDATE)**:「このユーザーの名前を変更して」
- **データの削除 (DELETE)**:「この投稿を削除して」

この記事では、特に利用頻度が高い**データの取得 (SELECT)** を中心に解説していきます。

## 2. すべての基本！`SELECT`文でデータを取得する

まずは、データベースからデータを取得するための最も基本的な構文です。

### 準備：サンプルテーブル

この記事では、以下のような2つのテーブルを例に進めていきます。

**`users`テーブル (ユーザー情報)**
| id | name | email | created_at |
| -- | ---- | ----- | ---------- |
| 1 | 田中 太郎 | taro@example.com | 2023-01-10 |
| 2 | 鈴木 花子 | hanako@example.com | 2023-02-05 |
| 3 | 佐藤 次郎 | jiro@example.com | 2023-03-15 |

**`posts`テーブル (投稿情報)**
| id | user_id | title | created_at |
| -- | ------- | ----- | ---------- |
| 101 | 1 | SQLはじめました | 2023-04-01 |
| 102 | 2 | 今日のランチ | 2023-04-02 |
| 103 | 1 | SQL楽しい | 2023-04-03 |

### すべての列を取得する (`SELECT *`)

`users`テーブルのすべてのデータを取得してみましょう。`*`（アスタリスク）は「すべての列」を意味します。

```sql
SELECT * FROM users;
```

**実行結果**
| id | name | email | created_at |
| -- | ---- | ----- | ---------- |
| 1 | 田中 太郎 | taro@example.com | 2023-01-10 |
| 2 | 鈴木 花子 | hanako@example.com | 2023-02-05 |
| 3 | 佐藤 次郎 | jiro@example.com | 2023-03-15 |

### 特定の列だけ取得する

`name`と`email`だけが欲しい場合は、列名を指定します。

```sql
SELECT name, email FROM users;
```

**実行結果**
| name | email |
| ---- | ----- |
| 田中 太郎 | taro@example.com |
| 鈴木 花子 | hanako@example.com |
| 佐藤 次郎 | jiro@example.com |

---

## 3. 条件を指定して絞り込む `WHERE`句

`WHERE`句を使うと、特定の条件に一致するデータ（行）だけを絞り込むことができます。

### IDが1のユーザーを取得する

```sql
SELECT * FROM users WHERE id = 1;
```

**実行結果**
| id | name | email | created_at |
| -- | ---- | ----- | ---------- |
| 1 | 田中 太郎 | taro@example.com | 2023-01-10 |

### 複数の条件を組み合わせる (`AND`, `OR`)

`AND`は「AかつB」、`OR`は「AまたはB」の条件を指定できます。

- `user_id`が1 **かつ**、タイトルに「SQL」という文字が含まれる投稿を取得
- `LIKE '%キーワード%'` で、部分一致検索ができます。

```sql
SELECT * FROM posts WHERE user_id = 1 AND title LIKE '%SQL%';
```

**実行結果**
| id | user_id | title | created_at |
| -- | ------- | ----- | ---------- |
| 101 | 1 | SQLはじめました | 2023-04-01 |
| 103 | 1 | SQL楽しい | 2023-04-03 |

---

## 4. 結果を並び替える `ORDER BY`句

`ORDER BY`句を使うと、取得した結果を指定した列で並び替えることができます。

- `ASC`: 昇順（小さい順、古い順） ※デフォルト
- `DESC`: 降順（大きい順、新しい順）

### ユーザーを登録日が新しい順に並べる

```sql
SELECT * FROM users ORDER BY created_at DESC;
```

**実行結果**
| id | name | email | created_at |
| -- | ---- | ----- | ---------- |
| 3 | 佐藤 次郎 | jiro@example.com | 2023-03-15 |
| 2 | 鈴木 花子 | hanako@example.com | 2023-02-05 |
| 1 | 田中 太郎 | taro@example.com | 2023-01-10 |

---

## 5. 複数のテーブルを合体！ `JOIN`句 🤝

ここがSQLの強力なところであり、初心者がつまずきやすいポイントです。
`JOIN`を使うと、関連する複数のテーブルを1つの大きな表のように扱えます。

例えば、「投稿一覧に、投稿者の名前も表示したい」というケースを考えます。
`posts`テーブルには投稿者のID (`user_id`) しかなく、名前は`users`テーブルにあります。
この2つのテーブルを`users.id`と`posts.user_id`をキーにして連結します。

### `INNER JOIN`で内部結合する

`INNER JOIN`は、両方のテーブルに存在するデータだけを結合します。

```sql
SELECT
  posts.id,
  posts.title,
  users.name -- usersテーブルからnameを取得
FROM
  posts
INNER JOIN
  users ON posts.user_id = users.id; -- ここで2つのテーブルを連結
```

**実行結果**
| id | title | name |
| -- | ----- | ---- |
| 101 | SQLはじめました | 田中 太郎 |
| 102 | 今日のランチ | 鈴木 花子 |
| 103 | SQL楽しい | 田中 太郎 |

:::tip `AS`で別名をつける
テーブル名や列名が長くなるときは、`AS`を使って別名（エイリアス）をつけると、クエリがスッキリして読みやすくなります。

```sql
SELECT
  p.id AS post_id, -- 列名に別名
  p.title,
  u.name
FROM
  posts AS p -- テーブル名に別名
INNER JOIN
  users AS u ON p.user_id = u.id;
```
:::

---

## 6. データを集計・グループ化する (`GROUP BY`と集計関数)

`COUNT`や`SUM`などの**集計関数**と`GROUP BY`を組み合わせることで、データをグループ化して集計できます。

### `COUNT`：件数を数える

`COUNT(*)`で、テーブルの全行数を数えられます。

```sql
SELECT COUNT(*) FROM posts;
```

**実行結果**
| COUNT(*) |
| -------- |
| 3 |

### `GROUP BY`: グループごとに集計する

「ユーザーごとの投稿数」を数えてみましょう。
`user_id`でグループ化 (`GROUP BY`) し、それぞれのグループの行数を`COUNT`します。

```sql
SELECT
  user_id,
  COUNT(*) AS post_count -- ユーザーごとの投稿数を計算
FROM
  posts
GROUP BY
  user_id;
```

**実行結果**
| user_id | post_count |
| ------- | ---------- |
| 1 | 2 |
| 2 | 1 |

これを応用して、`JOIN`と組み合わせれば「ユーザー名ごとの投稿数」も取得できますね！

```sql
SELECT
  u.name,
  COUNT(p.id) AS post_count
FROM
  users AS u
JOIN
  posts AS p ON u.id = p.user_id
GROUP BY
  u.name
ORDER BY
  post_count DESC; -- 投稿数が多い順に並び替え
```

**実行結果**
| name | post_count |
| ---- | ---------- |
| 田中 太郎 | 2 |
| 鈴木 花子 | 1 |

---

## 7. [発展] 一歩進んだテクニック

### サブクエリ：`SELECT`文の中に`SELECT`文

`WHERE`句の中などで、別の`SELECT`文の結果を利用する方法です。
例えば、「2023年3月以降に登録したユーザーの投稿」を取得したい場合。

```sql
SELECT * FROM posts
WHERE user_id IN (
  SELECT id FROM users WHERE created_at >= '2023-03-01'
);
-- ()の中が先に実行され、結果は (3) となる
-- SELECT * FROM posts WHERE user_id IN (3); と同じ意味になる
```

### `HAVING`：`GROUP BY`の結果をさらに絞り込む

`WHERE`は`GROUP BY`で集計する**前**のデータを絞り込むのに対し、`HAVING`は`GROUP BY`で集計した**後**の結果を絞り込みます。

例えば、「投稿数が2件以上のユーザー」を探したい場合。

```sql
SELECT
  user_id,
  COUNT(*) AS post_count
FROM
  posts
GROUP BY
  user_id
HAVING
  post_count >= 2; -- GROUP BYの結果に対して条件を指定
```

**実行結果**
| user_id | post_count |
| ------- | ---------- |
| 1 | 2 |

## おわりに

今回は、SQLの中でも特に重要な`SELECT`文を中心に、データの取得・絞り込み・結合・集計といった基本的な操作を解説しました。

- `SELECT ... FROM ...`: 基本のデータ取得
- `WHERE`: 条件で絞り込み
- `ORDER BY`: 結果を並び替え
- `JOIN`: テーブルを結合
- `GROUP BY` & 集計関数: グループ化して集計

これらの基本を組み合わせるだけで、かなり複雑なデータも取り出せるようになります。
SQLは一度身につければ、どんなプログラミング言語やフレームワークを扱う上でも役立つ強力なスキルです。

ぜひ、実際に手を動かしながら、データベースとの対話を楽しんでみてください！

Happy Hacking! 🚀
```
