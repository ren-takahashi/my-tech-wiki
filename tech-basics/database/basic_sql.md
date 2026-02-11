# 基本SQL集

現場でよく使うSQLをまとめたチートシート。<br>
コピーして使える実用的なSQL文を記載。<br>

---

## テーブル例

以下のサンプルテーブルを前提としたSQL集。<br>

```sql
-- ユーザーテーブル
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    age INT,
    department VARCHAR(50),
    created_at TIMESTAMP
);

-- 注文テーブル
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    product_name VARCHAR(100),
    price INT,
    status VARCHAR(20),
    order_date DATE
);
```

---

## SELECT（基本）

### 全件取得
```sql
SELECT * FROM users;
```

### 特定のカラムだけ取得
```sql
SELECT id, name, email FROM users;
```

### WHERE条件（完全一致）
```sql
SELECT * FROM users WHERE department = '営業部';
```

### WHERE条件（複数）
```sql
SELECT * FROM users 
WHERE department = '営業部' 
  AND age >= 30;
```

### WHERE条件（OR）
```sql
SELECT * FROM users 
WHERE department = '営業部' 
   OR department = '開発部';
```

### IN句（複数の値に一致）
```sql
SELECT * FROM users 
WHERE department IN ('営業部', '開発部', '管理部');
```

### LIKE（部分一致）
```sql
-- 前方一致
SELECT * FROM users WHERE name LIKE '山田%';

-- 後方一致
SELECT * FROM users WHERE name LIKE '%太郎';

-- 部分一致
SELECT * FROM users WHERE email LIKE '%@example.com%';
```

### BETWEEN（範囲指定）
```sql
SELECT * FROM users 
WHERE age BETWEEN 25 AND 35;
```

### IS NULL / IS NOT NULL
```sql
-- NULLのデータを取得
SELECT * FROM users WHERE email IS NULL;

-- NULL以外のデータを取得
SELECT * FROM users WHERE email IS NOT NULL;
```

---

## ORDER BY（並び替え）

### 昇順（ASC: 小さい順）
```sql
SELECT * FROM users ORDER BY age ASC;
-- ASCは省略可能
SELECT * FROM users ORDER BY age;
```

### 降順（DESC: 大きい順）
```sql
SELECT * FROM users ORDER BY age DESC;
```

### 複数カラムで並び替え
```sql
SELECT * FROM users 
ORDER BY department ASC, age DESC;
```

---

## LIMIT（件数制限）

### 上位N件を取得
```sql
-- 上位10件
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
```

### OFFSET（スキップして取得）
```sql
-- 11件目から20件目を取得（ページネーション）
SELECT * FROM users 
ORDER BY id 
LIMIT 10 OFFSET 10;
```

---

## 集計関数

### COUNT（件数）
```sql
-- 全件数
SELECT COUNT(*) FROM users;

-- 特定条件の件数
SELECT COUNT(*) FROM users WHERE department = '営業部';

-- NULL以外の件数
SELECT COUNT(email) FROM users;

-- ユニークな値の件数
SELECT COUNT(DISTINCT department) FROM users;
```

### SUM（合計）
```sql
SELECT SUM(price) FROM orders;
```

### AVG（平均）
```sql
SELECT AVG(age) FROM users;
```

### MAX / MIN（最大・最小）
```sql
SELECT MAX(age) FROM users;
SELECT MIN(age) FROM users;
```

---

## GROUP BY（グループ化）

### 部署ごとの人数
```sql
SELECT department, COUNT(*) AS count
FROM users
GROUP BY department;
```

### 部署ごとの平均年齢
```sql
SELECT department, AVG(age) AS avg_age
FROM users
GROUP BY department
ORDER BY avg_age DESC;
```

### ユーザーごとの注文金額合計
```sql
SELECT user_id, SUM(price) AS total_price
FROM orders
GROUP BY user_id
ORDER BY total_price DESC;
```

### HAVING（集計後の絞り込み）
```sql
-- 人数が5人以上の部署だけ抽出
SELECT department, COUNT(*) AS count
FROM users
GROUP BY department
HAVING COUNT(*) >= 5;
```

**WHEREとHAVINGの違い:**
- WHERE: GROUP BY前の絞り込み（行に対する条件）
- HAVING: GROUP BY後の絞り込み（集計結果に対する条件）

```sql
-- WHEREとHAVINGの併用
SELECT department, AVG(age) AS avg_age
FROM users
WHERE age >= 20  -- GROUP BY前: 20歳未満を除外
GROUP BY department
HAVING AVG(age) >= 30  -- GROUP BY後: 平均年齢30歳以上の部署のみ
ORDER BY avg_age DESC;
```

---

## JOIN（結合）

### INNER JOIN（内部結合）
両方のテーブルに存在するデータのみ取得。

```sql
SELECT 
    users.name,
    users.department,
    orders.product_name,
    orders.price
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

### LEFT JOIN（左外部結合）
左側のテーブル（users）の全データを取得。<br>
右側のテーブル（orders）にデータがない場合はNULLで表示。

```sql
-- 注文していないユーザーも含めて全ユーザーを表示
SELECT 
    users.name,
    orders.product_name,
    orders.price
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

### 注文していないユーザーを抽出
```sql
SELECT users.name
FROM users
LEFT JOIN orders ON users.id = orders.user_id
WHERE orders.id IS NULL;
```

### 複数のJOIN
```sql
SELECT 
    users.name,
    orders.product_name,
    orders.price
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN departments ON users.department_id = departments.id;
```

---

## INSERT（挿入）

### 1件挿入
```sql
INSERT INTO users (name, email, age, department, created_at)
VALUES ('山田太郎', 'yamada@example.com', 30, '営業部', NOW());
```

### 複数件挿入
```sql
INSERT INTO users (name, email, age, department)
VALUES 
    ('田中一郎', 'tanaka@example.com', 25, '開発部'),
    ('佐藤花子', 'sato@example.com', 28, '管理部'),
    ('鈴木次郎', 'suzuki@example.com', 35, '営業部');
```

### 特定カラムのみ挿入（他はNULL）
```sql
INSERT INTO users (name, email)
VALUES ('高橋三郎', 'takahashi@example.com');
```

---

## UPDATE（更新）

### 1件更新（WHERE必須！）
```sql
UPDATE users
SET age = 31, department = '開発部'
WHERE id = 1;
```

### 複数件更新
```sql
UPDATE users
SET department = '新営業部'
WHERE department = '営業部';
```

### **注意: WHEREを忘れると全件更新されてしまう！**
```sql
-- ❌ 危険: 全ユーザーが同じ年齢になる
UPDATE users SET age = 30;

-- ✅ 正しい: 条件を指定
UPDATE users SET age = 30 WHERE id = 1;
```

---

## DELETE（削除）

### 1件削除（WHERE必須！）
```sql
DELETE FROM users WHERE id = 1;
```

### 条件に一致する複数件削除
```sql
DELETE FROM users WHERE age < 20;
```

### **注意: WHEREを忘れると全件削除されてしまう**
```sql
-- ❌ 超危険: 全データが消える
DELETE FROM users;

-- ✅ 正しい: 条件を指定
DELETE FROM users WHERE id = 1;
```

### 安全な削除確認手順
```sql
-- 1. まずSELECTで確認
SELECT * FROM users WHERE age < 20;

-- 2. 削除対象が正しければDELETE実行
DELETE FROM users WHERE age < 20;
```

---

## サブクエリ

### WHERE句でサブクエリ
```sql
-- 平均年齢以上のユーザーを取得
SELECT name, age
FROM users
WHERE age >= (SELECT AVG(age) FROM users);
```

### IN句でサブクエリ
```sql
-- 注文履歴があるユーザーを取得
SELECT name
FROM users
WHERE id IN (SELECT DISTINCT user_id FROM orders);
```

### FROM句でサブクエリ
```sql
-- 部署ごとの平均年齢が30歳以上の部署を取得
SELECT *
FROM (
    SELECT department, AVG(age) AS avg_age
    FROM users
    GROUP BY department
) AS dept_avg
WHERE avg_age >= 30;
```

---

## CASE文（条件分岐）

### 基本的なCASE
```sql
SELECT 
    name,
    age,
    CASE
        WHEN age < 25 THEN '若手'
        WHEN age < 35 THEN '中堅'
        ELSE 'ベテラン'
    END AS age_group
FROM users;
```

### 集計と組み合わせ
```sql
SELECT 
    department,
    COUNT(CASE WHEN age < 30 THEN 1 END) AS under_30,
    COUNT(CASE WHEN age >= 30 THEN 1 END) AS over_30
FROM users
GROUP BY department;
```

---

## DISTINCT（重複排除）

### 重複を除いて取得
```sql
-- 部署の一覧（重複なし）
SELECT DISTINCT department FROM users;
```

### 複数カラムの組み合わせで重複排除
```sql
SELECT DISTINCT department, age FROM users;
```

---

## 日付・時刻

### 現在日時
```sql
-- MySQL/PostgreSQL
SELECT NOW();
SELECT CURRENT_TIMESTAMP;

-- 現在日付のみ
SELECT CURDATE();  -- MySQL
SELECT CURRENT_DATE;  -- PostgreSQL
```

### 日付の比較
```sql
-- 今日の注文
SELECT * FROM orders 
WHERE order_date = CURDATE();

-- 過去7日間の注文
SELECT * FROM orders 
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 7 DAY);  -- MySQL

-- 特定期間
SELECT * FROM orders 
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

### 日付のフォーマット
```sql
-- MySQL
SELECT DATE_FORMAT(order_date, '%Y/%m/%d') FROM orders;

-- PostgreSQL
SELECT TO_CHAR(order_date, 'YYYY/MM/DD') FROM orders;
```

---

## 便利な関数

### 文字列操作
```sql
-- 連結
SELECT CONCAT(name, ' (', department, ')') FROM users;

-- 大文字・小文字変換
SELECT UPPER(name), LOWER(email) FROM users;

-- 部分文字列
SELECT SUBSTRING(email, 1, 5) FROM users;

-- 文字列長
SELECT name, LENGTH(name) FROM users;
```

### 数値操作
```sql
-- 四捨五入
SELECT ROUND(price * 1.1, 0) FROM orders;  -- 小数点以下0桁

-- 切り上げ・切り捨て
SELECT CEIL(price / 100) FROM orders;   -- 切り上げ
SELECT FLOOR(price / 100) FROM orders;  -- 切り捨て
```

### NULL処理
```sql
-- NULLを別の値に置き換え
-- MySQL
SELECT IFNULL(email, '未設定') FROM users;

-- PostgreSQL
SELECT COALESCE(email, '未設定') FROM users;
```

---

## トランザクション

### 基本的なトランザクション
```sql
-- トランザクション開始
BEGIN;  -- または START TRANSACTION;

-- 複数のSQL実行
UPDATE users SET age = 31 WHERE id = 1;
INSERT INTO orders (user_id, product_name, price) VALUES (1, '商品A', 1000);

-- 確定（成功時）
COMMIT;

-- 取り消し（失敗時）
ROLLBACK;
```

---

## 実務Tips

### SELECT前に実行件数を確認（**SELECT前に件数確認**で大量データ取得を防ぐ）
```sql
-- まず件数確認
SELECT COUNT(*) FROM users WHERE age < 20;

-- 問題なければ実行
SELECT * FROM users WHERE age < 20;
```

### UPDATE/DELETE前に必ずSELECTで確認（**UPDATE/DELETE前にSELECTで確認**で誤操作防止）
```sql
-- 1. SELECTで対象を確認
SELECT * FROM users WHERE department = '営業部';

-- 2. 問題なければUPDATE/DELETE
UPDATE users SET department = '新営業部' WHERE department = '営業部';
```

### インデックスを意識したWHERE句
```sql
-- ❌ 関数を使うとインデックスが効かない
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- ✅ 範囲指定でインデックスを活用
SELECT * FROM users 
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### LIMIT句の活用
```sql
-- 大量データの確認時はLIMITをつける
SELECT * FROM users LIMIT 100;
```

---

## よくあるミス

### 1. WHERE句の忘れ（全件更新・削除）
```sql
❌ DELETE FROM users;  -- 全件削除！
✅ DELETE FROM users WHERE id = 1;
```

### 2. GROUP BYに含まれないカラムをSELECT
```sql
❌ SELECT name, department, COUNT(*) FROM users GROUP BY department;
✅ SELECT department, COUNT(*) FROM users GROUP BY department;
```

### 3. JOINのON句を忘れる
```sql
❌ SELECT * FROM users JOIN orders;  -- クロス結合になる
✅ SELECT * FROM users JOIN orders ON users.id = orders.user_id;
```

### 4. ORDER BYの位置
```sql
❌ SELECT * FROM users ORDER BY age WHERE age > 20;  -- 構文エラー
✅ SELECT * FROM users WHERE age > 20 ORDER BY age;
```

**SQL句の正しい順番:**
```
SELECT
FROM
JOIN
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```
