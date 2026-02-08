# インデックス（Index）とパフォーマンス最適化

## インデックスとは？

データベースの検索速度を劇的に向上させる仕組み。<br>
本の「索引（さくいん）」と同じで、目的のデータを素早く見つけるための**目次**のようなもの。<br>

**インデックスなし:**<br>
100万件のテーブルから特定のユーザーを探す → 全行を1件ずつチェック（フルスキャン）<br>
→ 数秒〜数十秒かかる<br>

**インデックスあり:**<br>
インデックスを使って一発で該当行にジャンプ<br>
→ 数ミリ秒で完了<br>

---

## インデックスの基本

### インデックスが効く場面

以下の箇所で使われるカラムにインデックスを張ると効果的：<br>

1. **WHERE 句**（検索条件）<br>
2. **JOIN**（テーブル結合）<br>
3. **ORDER BY**（並び替え）<br>
4. **GROUP BY**（集計）<br>

**例:**<br>
```sql
-- このクエリが遅い場合
SELECT * FROM users WHERE email = 'example@example.com';

-- email カラムにインデックスを張る
CREATE INDEX idx_users_email ON users(email);

-- 劇的に速くなる
```

---

### インデックスのコスト

**メリット:**<br>
- SELECT（検索）が高速化<br>

**デメリット:**<br>
- **INSERT / UPDATE / DELETE が遅くなる**（インデックスも更新が必要）<br>
- ディスク容量を消費する<br>

**実務の鉄則:**<br>
「全てのカラムにインデックス」は NG。<br>
本当に必要なカラムだけに絞る。<br>

---

## インデックスの種類

### 1. 単一カラムインデックス

1つのカラムに対するインデックス。<br>

```sql
-- email カラムにインデックス
CREATE INDEX idx_users_email ON users(email);
```

**使われる場面:**<br>
```sql
SELECT * FROM users WHERE email = 'test@example.com';
```

---

### 2. 複合インデックス（Composite Index）

複数のカラムをまとめたインデックス。<br>
**カラムの順序が非常に重要**。<br>

```sql
-- (last_name, first_name) の順で複合インデックス
CREATE INDEX idx_users_name ON users(last_name, first_name);
```

**使われる場面（○）:**<br>
```sql
-- ○ last_name だけの検索 → 使われる
SELECT * FROM users WHERE last_name = '田中';

-- ○ last_name + first_name の検索 → 使われる
SELECT * FROM users WHERE last_name = '田中' AND first_name = '太郎';
```

**使われない場面（×）:**<br>
```sql
-- × first_name だけの検索 → インデックスが使われない
SELECT * FROM users WHERE first_name = '太郎';
```

**なぜ？**<br>
複合インデックスは**左から順に使われる**（左端一致の法則）。<br>
電話帳の「あいうえお順」で探すのと同じで、「名前だけ」では探せない。<br>

**複合インデックスの順序の決め方:**<br>
1. **絞り込み能力が高い（カーディナリティが高い）カラムを左に**<br>
   - 例: `(prefecture, city)` より `(city, prefecture)` の方が良い（cityの方が種類が多い）<br>
2. **WHERE句でよく使われる順**<br>

---

### 3. ユニークインデックス（UNIQUE Index）

重複を許さないインデックス。<br>

```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- または制約として定義
ALTER TABLE users ADD CONSTRAINT uk_email UNIQUE (email);
```

**効果:**<br>
- 検索が速くなる<br>
- データの一意性が保証される（同じメールアドレスを登録できない）<br>

---

### 4. 部分インデックス（Partial Index）

条件付きでインデックスを作成（PostgreSQL など）。<br>

```sql
-- status が 'active' のユーザーだけインデックス化
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';
```

**メリット:**<br>
- インデックスサイズが小さくなる<br>
- 更新コストが下がる<br>

**使い所:**<br>
特定の条件でしか検索しない場合（論理削除で `deleted_at IS NULL` など）<br>

---

## 実行計画（EXPLAIN）の読み方

クエリがどう実行されるかを確認するツール。<br>
**インデックスが使われているか**をチェックするために必須。<br>

### 基本的な使い方

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

**PostgreSQL の出力例:**<br>
```
Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=...)
  Index Cond: (email = 'test@example.com'::text)
```

**MySQL の出力例:**<br>
```
+----+-------------+-------+------+------------------+---------+-------+
| id | select_type | table | type | possible_keys    | key     | rows  |
+----+-------------+-------+------+------------------+---------+-------+
|  1 | SIMPLE      | users | ref  | idx_users_email  | idx_... | 1     |
+----+-------------+-------+------+------------------+---------+-------+
```

---

### 重要な項目の見方

#### 1. type（MySQL）/ Scan Type（PostgreSQL）

クエリの実行方法。**上から順に高速:**<br>

| type | 意味 | 速度 |
|------|------|------|
| **const** | 主キーまたはユニークキーで1件取得 | ★★★★★ 最速 |
| **eq_ref** | JOINで主キー・ユニークキーを使用 | ★★★★☆ 高速 |
| **ref** | 非ユニークインデックスを使用 | ★★★☆☆ 普通 |
| **range** | インデックスで範囲検索（BETWEEN, >, < など） | ★★☆☆☆ やや遅い |
| **index** | インデックス全体をスキャン | ★☆☆☆☆ 遅い |
| **ALL** | テーブル全体をスキャン（フルスキャン） | ☆☆☆☆☆ 非常に遅い |

**ALL（フルスキャン）が出たら要注意！**<br>
インデックスが使われていないので、大量データでは致命的に遅い。<br>

---

#### 2. rows（推定行数）

クエリが読み込む行数の推定値。<br>

```
rows=1          → 速い
rows=100        → まあまあ
rows=100000     → 遅い（インデックス追加を検討）
```

---

#### 3. key（使用されたインデックス）

実際に使われたインデックス名。<br>

```
key: idx_users_email  → インデックスが使われている ○
key: NULL             → インデックスが使われていない ×
```

---

### EXPLAIN ANALYZE（実際の実行結果を見る）

EXPLAINは「予測」だが、EXPLAIN ANALYZEは「実測値」を返す。<br>

**PostgreSQL:**<br>
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

**出力例:**<br>
```
Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=...)
                                           (actual time=0.025..0.026 rows=1 loops=1)
Planning Time: 0.089 ms
Execution Time: 0.053 ms
```

**Execution Time（実行時間）** を見て、遅ければ改善が必要。<br>

---

## インデックスを張るべきカラムの見分け方

### 1. WHERE 句で頻繁に使うカラム

```sql
-- よく実行されるクエリ
SELECT * FROM orders WHERE user_id = 123;

-- user_id にインデックスを張る
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

### 2. JOIN で使うカラム

```sql
-- orders と users を結合
SELECT * FROM orders 
JOIN users ON orders.user_id = users.id;

-- orders.user_id にインデックス（users.id は主キーなので既にある）
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

### 3. ORDER BY / GROUP BY で使うカラム

```sql
-- 作成日時で並び替え
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10;

-- created_at にインデックス
CREATE INDEX idx_posts_created_at ON posts(created_at);
```

---

### 4. カーディナリティが高いカラム

**カーディナリティ = 値の種類の多さ**<br>

| カラム | カーディナリティ | インデックス効果 |
|--------|----------------|----------------|
| email（ユーザー100万人） | 高い（100万通り） | ★★★★★ 効果大 |
| prefecture（都道府県） | 低い（47通り） | ★★☆☆☆ 効果小 |
| gender（性別） | 非常に低い（2〜3通り） | ☆☆☆☆☆ ほぼ無意味 |

**カーディナリティが低い**カラムは、インデックスを張っても効果が薄い。<br>

---

## インデックスが使われない（効かない）ケース

### 1. 関数を使った検索

```sql
-- × インデックスが効かない
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';

-- ○ インデックスが効く
SELECT * FROM users WHERE email = 'test@example.com';
```

**対策:**<br>
- 関数を使わずに済む設計にする<br>
- または**関数インデックス**を作成（PostgreSQLなど）<br>
```sql
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
```

---

### 2. LIKE の前方一致以外

```sql
-- ○ 前方一致はインデックスが効く
SELECT * FROM users WHERE name LIKE '田中%';

-- × 中間一致・後方一致は効かない
SELECT * FROM users WHERE name LIKE '%田中%';
SELECT * FROM users WHERE name LIKE '%田中';
```

**対策:**<br>
全文検索インデックス（Full-Text Search）を使う。<br>

---

### 3. OR 条件

```sql
-- × インデックスが効きにくい
SELECT * FROM users WHERE email = 'test@example.com' OR phone = '090-1234-5678';

-- ○ IN や UNION を検討
SELECT * FROM users WHERE email IN ('test@example.com', 'another@example.com');
```

---

### 4. 型の不一致

```sql
-- × email は VARCHAR 型なのに数値で検索
SELECT * FROM users WHERE email = 12345;  -- 暗黙の型変換が起きてインデックス無効

-- ○ 文字列として検索
SELECT * FROM users WHERE email = '12345';
```

---

## カバリングインデックス（Covering Index）

**SELECT で取得するカラムが全てインデックスに含まれている**状態。<br>
テーブル本体を読まずに、インデックスだけで結果を返せるため超高速。<br>

**例:**<br>
```sql
-- このクエリが頻繁に実行される
SELECT id, email FROM users WHERE email = 'test@example.com';

-- email だけでなく id も含む複合インデックス
CREATE INDEX idx_users_email_id ON users(email, id);
```

**効果:**<br>
PostgreSQL: `Index Only Scan`（インデックスのみスキャン）<br>
MySQL: `Using index`（インデックスのみ使用）<br>

テーブル本体にアクセスしないため、さらに高速化する。<br>

---

## 実務でのパフォーマンスチューニング手順

### 1. 遅いクエリを特定する

**スロークエリログを有効化:**<br>

**MySQL:**<br>
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 1秒以上かかるクエリを記録
```

**PostgreSQL:**<br>
```sql
ALTER SYSTEM SET log_min_duration_statement = 1000;  -- 1000ms = 1秒
SELECT pg_reload_conf();
```

---

### 2. EXPLAIN で実行計画を確認

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
```

**チェックポイント:**<br>
- `type` が `ALL`（フルスキャン）になっていないか<br>
- `rows` が数万〜数十万になっていないか<br>
- `key` が `NULL`（インデックス未使用）になっていないか<br>

---

### 3. インデックスを追加

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

---

### 4. 再度 EXPLAIN で効果を確認

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
```

`type` が `ref` や `eq_ref` に変わり、`rows` が減っていれば成功。<br>

---

### 5. 実測で速度を確認

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;
```

`Execution Time` が改善されていることを確認。<br>

---

## 実務での注意点

### 1. インデックスを張りすぎない

**問題:**<br>
- INSERT / UPDATE / DELETE が遅くなる<br>
- ディスク容量を圧迫<br>

**目安:**<br>
1テーブルあたり**5〜10個以内**に抑える。<br>

---

### 2. 使われないインデックスを削除する

**PostgreSQL で未使用インデックスを確認:**<br>
```sql
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan  -- インデックスが使われた回数
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY schemaname, tablename;
```

`idx_scan = 0`（一度も使われていない）インデックスは削除を検討。<br>

```sql
DROP INDEX idx_unused_index;
```

---

### 3. 定期的にインデックスを再構築

長期間使っていると、インデックスが断片化して効率が落ちる。<br>

**PostgreSQL:**<br>
```sql
REINDEX TABLE users;
```

**MySQL:**<br>
```sql
OPTIMIZE TABLE users;
```

---

## まとめ

| 項目 | ポイント |
|------|---------|
| **基本** | WHERE、JOIN、ORDER BY で使うカラムにインデックス |
| **複合インデックス** | カラムの順序が重要（左端一致の法則） |
| **EXPLAIN** | type=ALL（フルスキャン）は要改善 |
| **カーディナリティ** | 値の種類が多いカラムほど効果大 |
| **効かないケース** | 関数、LIKE中間一致、OR、型不一致に注意 |
| **カバリングインデックス** | SELECT するカラムを全て含むと超高速 |
| **実務の鉄則** | スロークエリログ → EXPLAIN → インデックス追加 → 再計測 |

インデックスは「クエリが遅い」問題の9割を解決する最重要テーマ。<br>
EXPLAIN を使いこなし、適切なインデックスを設計することが実務では必須。<br>
