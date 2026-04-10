# テーブル設計とリレーションシップ

データベース設計において、テーブル間の関係（カーディナリティ）をどう表現するかは非常に重要。<br>
関係の種類によって適切な設計パターンが決まっている。<br>

---

## カーディナリティとは

テーブル間の関係性を表す概念。<br>
「1つのAに対して、いくつのBが関連するか」「1つのBに対して、いくつのAが関連するか」を表す。<br>

※ 文脈によって、そのカラムの「値の種類数」を指し、「カーディナリティが高い/低い」と言うこともあり、ここでのカーディナリティとは異なる意味を持つこともある。

---

## 1対1（One-to-One）の関係

**1つのAに対して、1つのBだけが関連する。**<br>
逆に、1つのBに対しても、1つのAだけが関連する。<br>

### 具体例

#### 例1：ユーザーと詳細プロフィール
- 1人のユーザーに対して、1つの詳細プロフィールのみ
- 1つの詳細プロフィールは、1人のユーザーにのみ紐づく

#### 例2：国と首都
- 1つの国に対して、1つの首都のみ
- 1つの首都は、1つの国にのみ紐づく

### 設計パターン

#### パターン1：テーブルを分けない（推奨：データが常に存在する場合）

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    -- プロフィール情報も同じテーブルに
    bio TEXT,
    avatar_url VARCHAR(200),
    website VARCHAR(200)
);
```

**メリット：**<br>
- JOINが不要で高速<br>
- シンプルで分かりやすい<br>

**デメリット：**<br>
- プロフィール情報がない場合もカラムが必要（NULLが増える）<br>

---

#### パターン2：テーブルを分ける

```sql
-- ユーザーテーブル（基本情報）
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

-- プロフィールテーブル（詳細情報）
CREATE TABLE user_profiles (
    id INT PRIMARY KEY,
    user_id INT UNIQUE NOT NULL,  -- UNIQUE制約で1対1を保証
    bio TEXT,
    avatar_url VARCHAR(200),
    website VARCHAR(200),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**使い所：**<br>
- プロフィール情報が任意（ない場合もある）<br>
- プロフィール情報が大きい（テキストやBLOB）<br>
- アクセス頻度が異なる（基本情報は頻繁、詳細情報はたまに）<br>

**ポイント：**<br>
- `user_id` に **UNIQUE制約** をつけることで、1対1を保証<br>
- 外部キーは「片方だけ」に持つ（user_profiles側に持つのが一般的）<br>

### インデックス設計

```sql
-- 外部キーにインデックス（JOIN用）
CREATE UNIQUE INDEX idx_user_profiles_user_id ON user_profiles(user_id);
```

### よく使うSQL

```sql
-- ユーザーとプロフィールを取得
SELECT 
    users.name,
    users.email,
    user_profiles.bio,
    user_profiles.avatar_url
FROM users
LEFT JOIN user_profiles ON users.id = user_profiles.user_id
WHERE users.id = 1;
```

---

## 1対多（One-to-Many）の関係

**1つのAに対して、複数のBが関連する。**<br>
逆に、1つのBに対しては、1つのAだけが関連する。<br>

**最も基本的で頻繁に使われる関係。**<br>

### 具体例

#### 例1：ユーザーと投稿
- 1人のユーザーは複数の投稿を作成できる
- 1つの投稿は1人のユーザーにのみ紐づく

#### 例2：部署と社員
- 1つの部署には複数の社員が所属
- 1人の社員は1つの部署にのみ所属

#### 例3：注文と注文明細
- 1つの注文には複数の商品（明細）が含まれる
- 1つの注文明細は1つの注文にのみ紐づく

### 設計パターン

**「多」側に外部キーを持つ。**<br>

```sql
-- ユーザーテーブル（1側）
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- 投稿テーブル（多側）
CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,  -- 外部キーは「多」側に
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**ポイント：**<br>
- 外部キーは必ず「多」側（posts）に配置<br>
- 「1」側（users）には何も追加しない<br>

### インデックス設計

```sql
-- 外部キーには必ずインデックス（超重要！）
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- よく一緒に使う条件を複合インデックスで
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);
```

**理由：**<br>
「ユーザーIDで投稿を取得する」クエリが頻繁に実行されるため、<br>
インデックスがないとフルスキャンになり致命的に遅くなる。<br>

### よく使うSQL

#### 特定ユーザーの投稿を取得
```sql
SELECT * FROM posts 
WHERE user_id = 1
ORDER BY created_at DESC;
```

#### ユーザーごとの投稿数を集計
```sql
SELECT 
    users.name,
    COUNT(posts.id) AS post_count
FROM users
LEFT JOIN posts ON users.id = posts.user_id
GROUP BY users.id, users.name
ORDER BY post_count DESC;
```

#### 投稿とユーザー名を一緒に取得
```sql
SELECT 
    posts.title,
    posts.content,
    users.name AS author_name
FROM posts
INNER JOIN users ON posts.user_id = users.id
WHERE posts.id = 100;
```

---

## 多対多（Many-to-Many）の関係

**1つのAに対して、複数のBが関連する。**<br>
**逆に、1つのBに対しても、複数のAが関連する。**<br>

**中間テーブルを使わないと表現できない。**<br>

### 具体例

#### 例1：学生と授業
- 1人の学生は複数の授業を受講できる
- 1つの授業は複数の学生に受講される

#### 例2：ユーザーとタグ（ブログ）
- 1人のユーザーは複数のタグを持てる
- 1つのタグは複数のユーザーに使われる

#### 例3：商品とカテゴリ（ECサイト）
- 1つの商品は複数のカテゴリに属せる
- 1つのカテゴリには複数の商品がある

### 設計パターン

**中間テーブル（Junction Table / Bridge Table）を作成する。**<br>

```sql
-- 学生テーブル
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- 授業テーブル
CREATE TABLE courses (
    id INT PRIMARY KEY,
    course_name VARCHAR(100),
    teacher VARCHAR(100)
);

-- 中間テーブル（student_courses）
CREATE TABLE student_courses (
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id, course_id),  -- 複合主キー
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

**ポイント：**<br>
- 中間テーブルは基本的に「2つの外部キー」だけを持つ<br>
- `PRIMARY KEY (student_id, course_id)` で重複を防ぐ<br>
- 追加のカラム（enrolled_at、gradeなど）も持てる<br>

### 中間テーブルに追加情報を持つパターン

```sql
-- 成績や登録日時などを追加
CREATE TABLE student_courses (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- 独自のID
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    grade VARCHAR(2),  -- 成績（A, B, C...）
    UNIQUE KEY unique_enrollment (student_id, course_id),  -- 重複防止
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

### インデックス設計

```sql
-- 両方向の検索に対応
CREATE INDEX idx_student_courses_student ON student_courses(student_id);
CREATE INDEX idx_student_courses_course ON student_courses(course_id);

-- または複合インデックス（片方向の検索が多い場合）
CREATE INDEX idx_student_courses ON student_courses(student_id, course_id);
```

**重要：**<br>
中間テーブルは JOIN で頻繁に使われるため、インデックスが必須！<br>

### よく使うSQL

#### 特定の学生が受講している授業を取得
```sql
SELECT 
    courses.id,
    courses.course_name,
    courses.teacher
FROM courses
INNER JOIN student_courses ON courses.id = student_courses.course_id
WHERE student_courses.student_id = 1;
```

#### 特定の授業を受講している学生を取得
```sql
SELECT 
    students.id,
    students.name,
    students.email
FROM students
INNER JOIN student_courses ON students.id = student_courses.student_id
WHERE student_courses.course_id = 101;
```

#### 学生ごとの受講授業数を集計
```sql
SELECT 
    students.name,
    COUNT(student_courses.course_id) AS course_count
FROM students
LEFT JOIN student_courses ON students.id = student_courses.student_id
GROUP BY students.id, students.name
ORDER BY course_count DESC;
```

#### 授業を1つも受講していない学生を抽出
```sql
SELECT students.name
FROM students
LEFT JOIN student_courses ON students.id = student_courses.student_id
WHERE student_courses.course_id IS NULL;
```

---

## 自己参照（Self-referencing）の関係

**同じテーブル内で親子関係を持つパターン。**<br>

### 具体例

#### 例1：従業員と上司
- 従業員も上司も同じ「社員」テーブル
- 各社員には「上司」がいる（社長を除く）

#### 例2：カテゴリの階層構造
- 大カテゴリ → 中カテゴリ → 小カテゴリ
- 「親カテゴリ」自身もカテゴリ

#### 例3：SNSのフォロー関係
- ユーザーが別のユーザーをフォロー

### 設計パターン

#### パターン1：単純な親子関係（ツリー構造）

```sql
-- 社員テーブル
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    position VARCHAR(50),
    manager_id INT,  -- 上司の社員ID（自己参照）
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);
```

**データ例：**
```sql
INSERT INTO employees VALUES
(1, '社長', 'CEO', NULL),          -- 社長には上司がいない
(2, '部長A', '営業部長', 1),       -- 社長の部下
(3, '部長B', '開発部長', 1),       -- 社長の部下
(4, '課長A', '営業課長', 2),       -- 部長Aの部下
(5, '社員A', '営業', 4);           -- 課長Aの部下
```

#### パターン2：多対多の自己参照（SNSフォロー）

```sql
-- ユーザーテーブル
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- フォロー関係テーブル（中間テーブル）
CREATE TABLE follows (
    follower_id INT NOT NULL,   -- フォローする人
    followee_id INT NOT NULL,   -- フォローされる人
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id),
    FOREIGN KEY (follower_id) REFERENCES users(id),
    FOREIGN KEY (followee_id) REFERENCES users(id)
);
```

### インデックス設計

```sql
-- 上司で検索する場合（「この上司の部下は誰？」）
CREATE INDEX idx_employees_manager ON employees(manager_id);

-- フォロー関係の検索
CREATE INDEX idx_follows_follower ON follows(follower_id);  -- 「Aさんがフォローしている人」
CREATE INDEX idx_follows_followee ON follows(followee_id);  -- 「Aさんをフォローしている人」
```

### よく使うSQL

#### 特定の上司の直属の部下を取得
```sql
SELECT * FROM employees
WHERE manager_id = 2;  -- 部長Aの直属の部下
```

#### 上司の情報を含めて社員を取得（自己結合）
```sql
SELECT 
    e.name AS employee_name,
    e.position,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**結果：**
```
employee_name | position   | manager_name
--------------|------------|-------------
社長          | CEO        | NULL
部長A         | 営業部長   | 社長
部長B         | 開発部長   | 社長
課長A         | 営業課長   | 部長A
社員A         | 営業       | 課長A
```

#### ユーザーAがフォローしている人を取得
```sql
SELECT 
    users.id,
    users.name
FROM users
INNER JOIN follows ON users.id = follows.followee_id
WHERE follows.follower_id = 1;  -- ユーザーID=1がフォローしている人
```

#### ユーザーAをフォローしている人を取得
```sql
SELECT 
    users.id,
    users.name
FROM users
INNER JOIN follows ON users.id = follows.follower_id
WHERE follows.followee_id = 1;  -- ユーザーID=1をフォローしている人
```

---

## ポリモーフィック関連（実務応用）

**1つのテーブルが複数の異なるテーブルと関連を持つパターン。**<br>
主にRuby on Railsなどのフレームワークで使われる高度なパターン。<br>

### 具体例：コメント機能

- 投稿（posts）にもコメントできる<br>
- 写真（photos）にもコメントできる<br>
→ コメントテーブルは複数のテーブルと関連<br>

### 設計パターン

```sql
-- 投稿テーブル
CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT
);

-- 写真テーブル
CREATE TABLE photos (
    id INT PRIMARY KEY,
    url VARCHAR(200),
    caption TEXT
);

-- コメントテーブル（ポリモーフィック）
CREATE TABLE comments (
    id INT PRIMARY KEY,
    commentable_type VARCHAR(50) NOT NULL,  -- 'Post' or 'Photo'
    commentable_id INT NOT NULL,            -- posts.id or photos.id
    content TEXT,
    user_id INT,
    created_at TIMESTAMP
);
```

**データ例：**
```sql
-- 投稿ID=1へのコメント
INSERT INTO comments VALUES (1, 'Post', 1, 'いいね！', 100, NOW());

-- 写真ID=5へのコメント
INSERT INTO comments VALUES (2, 'Photo', 5, '綺麗ですね', 101, NOW());
```

### インデックス設計

```sql
-- type と id の複合インデックス
CREATE INDEX idx_comments_commentable ON comments(commentable_type, commentable_id);
```

### SQL

#### 特定の投稿へのコメントを取得
```sql
SELECT * FROM comments
WHERE commentable_type = 'Post' AND commentable_id = 1;
```

**注意：**<br>
- データベースレベルでの外部キー制約が使えない<br>
- アプリケーション側で整合性を保つ必要がある<br>
- 複雑になりやすいので、本当に必要か検討すること<br>

---

## まとめ：関係の種類と設計パターン

| 関係 | 説明 | 設計方法 | インデックス |
|------|------|---------|-------------|
| **1対1** | 1つのAに1つのBのみ | 片方に外部キー + UNIQUE制約 | 外部キーにUNIQUE INDEX |
| **1対多** | 1つのAに複数のB | 「多」側に外部キー | 外部キーにINDEX（必須！） |
| **多対多** | 複数のAと複数のB | 中間テーブルを作成 | 両方の外部キーにINDEX |
| **自己参照** | 同じテーブル内で親子 | 同じテーブルのIDを参照 | 親IDにINDEX |

---

## 実務での設計手順

### 1. 要件からエンティティを洗い出す
「ユーザー」「投稿」「コメント」「タグ」など

### 2. エンティティ間の関係を明確にする
- ユーザーと投稿：1対多
- 投稿とタグ：多対多
- 投稿とコメント：1対多

### 3. 関係に応じた設計パターンを適用
- 1対多 → 「多」側に外部キー
- 多対多 → 中間テーブル作成

### 4. 必要なインデックスを設計
- 外部キーには必ずインデックス
- JOIN、WHERE、ORDER BY で使うカラムにもインデックス

### 5. 正規化とのバランスを取る
- 基本は第3正規形まで
- パフォーマンスが問題なら意図的に非正規化も検討

---

## よくある間違い

### 間違い1：多対多を直接表現しようとする
```sql
-- ❌ こんな設計はできない
CREATE TABLE students (
    id INT,
    course_ids VARCHAR(100)  -- 'カンマ区切りで授業IDを保存' は絶対NG！
);
```

**なぜダメ？**<br>
- 第1正規形違反<br>
- JOIN できない<br>
- データの整合性が保てない<br>

**正解：**<br>
中間テーブルを作る。<br>

---

### 間違い2：外部キーにインデックスを張らない
```sql
-- ❌ インデックスなしで外部キーを作成
CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,  -- インデックスなし → 遅い！
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**問題：**<br>
`WHERE user_id = 1` などの検索が致命的に遅くなる。<br>

**正解：**
```sql
CREATE INDEX idx_posts_user_id ON posts(user_id);
```

---

### 間違い3：1対1なのにUNIQUE制約を忘れる
```sql
-- ❌ 1対1のはずなのにUNIQUE制約なし
CREATE TABLE user_profiles (
    user_id INT,
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
-- → 同じユーザーに複数のプロフィールが作れてしまう
```

**正解：**
```sql
CREATE TABLE user_profiles (
    user_id INT UNIQUE,  -- UNIQUE制約で1対1を保証
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## 主キーの設計：サロゲートキー と ナチュラルキー

テーブルの主キーを何にするかは、データベース設計の重要な判断ポイント。<br>
主キーには大きく分けて**サロゲートキー**と**ナチュラルキー**の2種類がある。<br>

---

### サロゲートキー（Surrogate Key / 代理キー）

**データの内容とは関係ない、人工的に割り当てたID。**<br>
AUTO_INCREMENTやUUIDで自動生成される値を主キーにする方式。<br>

**具体例：**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- サロゲートキー
    email VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMP
);
```

**特徴：**<br>
- ビジネス上の意味を持たない（単なる識別番号）<br>
- DBが自動で採番（1, 2, 3...）<br>
- 変わることがない（不変）<br>
- シンプルな整数値なので高速<br>

**メリット：**<br>
✅ **不変性が保証される**：メールアドレスや会員番号が変わってもIDは変わらない<br>
✅ **JOIN性能が良い**：整数のインデックスは高速<br>
✅ **シンプル**：どのテーブルも `id` で統一できる<br>
✅ **外部キーが安定**：参照先のキーが変わらないので、外部キー制約が壊れない<br>

**デメリット：**<br>
❌ **意味がない**：IDだけ見てもそれが何のデータか分からない<br>
❌ **インデックスが増える**：ビジネスキー（email等）にも別途UNIQUE INDEXが必要<br>

---

### ナチュラルキー（Natural Key / 自然キー）

**データ自体が持つ、ビジネス上意味のある値を主キーにする。**<br>
メールアドレス、会員番号、商品コードなど、実際に使われる識別子。<br>

**具体例：**
```sql
CREATE TABLE users (
    email VARCHAR(100) PRIMARY KEY,  -- ナチュラルキー
    name VARCHAR(100),
    created_at TIMESTAMP
);

CREATE TABLE products (
    product_code VARCHAR(20) PRIMARY KEY,  -- 商品コード（JAN等）
    name VARCHAR(200),
    price INT
);
```

**特徴：**<br>
- ビジネス上の意味がある（会員番号、商品コードなど）<br>
- データそのものが主キー<br>
- 理論的には変わる可能性がある<br>

**メリット：**<br>
✅ **意味がある**：キーを見ればそれが何か分かる<br>
✅ **インデックスが少ない**：主キー = ビジネスキーなので重複しない<br>
✅ **無駄なIDカラムがいらない**：テーブルがシンプル<br>

**デメリット：**<br>
❌ **変更のリスク**：メールアドレス変更、商品コード変更があると大変<br>
❌ **外部キー更新が複雑**：主キー変更時にCASCADE UPDATEが必要<br>
❌ **文字列の場合は遅い**：VARCHAR(100)のJOINは整数より遅い<br>
❌ **複合キーになりがち**：意味のある一意な値が1つに絞れない場合がある<br>

---

### 実務での使い分け

**現代の主流：サロゲートキー**<br>
ほとんどのWebアプリケーションでは**サロゲートキー（AUTO_INCREMENT）**を使うのが標準的。<br>

**サロゲートキーを使うべき場合（ほとんどのケース）：**<br>
- **ユーザー、投稿、注文など、一般的なテーブル**<br>
  → メールアドレスや注文番号が変わる可能性があるから<br>
- **外部キーで参照される可能性が高いテーブル**<br>
  → 参照先が変わると大変なので、不変のIDを使う<br>
- **性能を重視する場合**<br>
  → 整数のJOINは高速<br>

**ナチュラルキーを使うべき場合（限定的）：**<br>
- **マスターテーブル（国コード、通貨コードなど）**<br>
  ```sql
  CREATE TABLE countries (
      country_code CHAR(2) PRIMARY KEY,  -- 'JP', 'US' など（ISO 3166-1）
      name VARCHAR(100)
  );
  ```
  → 国コードは絶対に変わらないので安全<br>

- **中間テーブルの複合主キー**<br>
  ```sql
  CREATE TABLE enrollments (
      student_id INT,
      course_id INT,
      enrolled_at TIMESTAMP,
      PRIMARY KEY (student_id, course_id)  -- 複合ナチュラルキー
  );
  ```
  → 「学生ID + 講座ID」の組み合わせが一意で、変わらない<br>

- **絶対に変わらないビジネスキーがある場合**<br>
  - 社会保障番号（アメリカのSSN）<br>
  - ISBN（書籍コード）<br>
  - JANコード（商品コード）<br>
  ただし、「絶対に変わらない」は実際には少ない<br>

---

### よくある議論と結論

**「メールアドレスは一意だから主キーにできる？」**<br>
❌ **やめるべき**<br>
- メールアドレスは変更される可能性がある<br>
- 文字列なのでJOINが遅い<br>
- 外部キーで参照していると、変更時に大変<br>

**正解：**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- サロゲートキー
    email VARCHAR(100) UNIQUE NOT NULL,  -- 一意性はUNIQUE制約で保証
    name VARCHAR(100)
);
```

---

**「注文番号は一意だから主キーにできる？」**<br>
△ **できるが、サロゲートキーの方が安全**<br>
- 注文番号の採番ルールが将来変わる可能性がある<br>
- サロゲートキー + 注文番号（UNIQUE）の方が柔軟<br>

**推奨設計：**
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- サロゲートキー
    order_number VARCHAR(50) UNIQUE NOT NULL,  -- ビジネス上の注文番号
    user_id INT,
    total_amount INT,
    created_at TIMESTAMP
);
```

---

### UUID vs AUTO_INCREMENT

サロゲートキーの中でも、**整数（AUTO_INCREMENT）**と**UUID**のどちらを使うか議論がある。<br>

**AUTO_INCREMENT（整数）：**
```sql
id INT PRIMARY KEY AUTO_INCREMENT
-- 例: 1, 2, 3, 4, 5...
```

**メリット：**<br>
✅ シンプルで軽量（4バイトまたは8バイト）<br>
✅ インデックスが効率的（連番なので追加が高速）<br>
✅ 人間が見て分かりやすい<br>

**デメリット：**<br>
❌ 採番がDB任せ（分散システムで競合の可能性）<br>
❌ 連番なので、データ量が推測される（セキュリティ懸念）<br>

---

**UUID（ランダム識別子）：**
```sql
id CHAR(36) PRIMARY KEY
-- 例: '550e8400-e29b-41d4-a716-446655440000'
```

**メリット：**<br>
✅ グローバルに一意（複数DBで重複しない）<br>
✅ アプリ側で生成可能（DBに依存しない）<br>
✅ データ量が推測されない<br>

**デメリット：**<br>
❌ サイズが大きい（36バイト、UUIDバイナリでも16バイト）<br>
❌ ランダムなのでインデックス追加時に断片化（性能低下）<br>
❌ 人間が見て分かりにくい<br>

**実務の選択：**<br>
- **一般的なWebアプリ**: AUTO_INCREMENT（シンプル、高速）<br>
- **分散システム、マイクロサービス**: UUID（DBを跨いで一意）<br>
- **API公開**: UUID（連番だとデータ量が推測されるのを避ける）<br>

---

### まとめ

| 項目 | サロゲートキー | ナチュラルキー |
|------|--------------|--------------|
| **定義** | 人工的に割り当てたID | データ自体が持つ識別子 |
| **例** | AUTO_INCREMENT, UUID | メールアドレス、商品コード |
| **不変性** | 絶対に変わらない | 変わる可能性がある |
| **性能** | 高速（整数の場合） | 文字列なら遅い |
| **外部キー** | 安定している | 変更時に複雑 |
| **実務での推奨** | ✅ ほとんどのケースで推奨 | ⚠️ 限定的なケースのみ |

**基本方針：**<br>
- **迷ったらサロゲートキー（AUTO_INCREMENT）を使う**<br>
- ビジネスキー（メール、商品コード）は**UNIQUE制約**で一意性を保証<br>
- マスターデータ（国コード等）や変わらないことが確実な場合のみナチュラルキー<br>

**典型的な設計パターン：**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,  -- サロゲートキー（主キー）
    email VARCHAR(100) UNIQUE NOT NULL,  -- ナチュラルキー（一意制約）
    name VARCHAR(100)
);
```

この設計なら、IDで高速にJOINしつつ、emailでも一意性を保証できる。<br>

---

## 参考：正規化との関係

- **正規化**：テーブル内のデータの重複を排除する技法
- **リレーションシップ設計**：テーブル間の関係をどう表現するか

**両方を組み合わせて、良いデータベース設計になる。**<br>

例：ユーザーと投稿の1対多の関係<br>
- リレーションシップ設計で「posts に user_id を持つ」と決める<br>
- 正規化で「user_name を posts に持たない（重複するから）」と決める<br>

[正規化の詳細ページ](normalization.md)<br>
[インデックス最適化の詳細ページ](index_optimization.md)<br>
[基本SQL集](basic_sql.md)<br>
