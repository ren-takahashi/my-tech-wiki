# 認可（Authorization）とアクセス制御

## 認可（Authorization）とは

**認可（Authorization）** は、**「誰が何をできるか」を制御する仕組み**。<br>
認証（Authentication）で「誰であるか」を確認した後、その人が**特定の操作やリソースにアクセスする権限があるか**を判断するプロセス。<br>

**認証 vs 認可:**<br>
```
認証（Authentication）: 「あなたは誰ですか？」→ 本人確認
認可（Authorization）  : 「あなたは何ができますか？」→ 権限確認

例: ホテルの入館
- 認証: フロントで身分証を見せてチェックイン（本人確認）
- 認可: 自分の部屋の鍵カードで部屋に入る（アクセス権限の確認）
      → 他人の部屋には入れない
```

---

## パーミッション（Permission）

**パーミッション（Permission）** は、**個別の操作権限**のこと。<br>
「特定のリソースに対して、特定のアクションを実行できる権限」を表す最小単位。<br>

**構成要素:**<br>
```
パーミッション = リソース（対象） + アクション（操作）

例:
- users:read    → ユーザー情報を読み取る権限
- users:write   → ユーザー情報を編集する権限
- users:delete  → ユーザー情報を削除する権限
- posts:create  → 投稿を作成する権限
- posts:publish → 投稿を公開する権限
```

**具体例（ブログシステム）:**<br>
```
パーミッション一覧:
- articles:read     : 記事を閲覧する
- articles:create   : 記事を作成する
- articles:edit     : 記事を編集する
- articles:delete   : 記事を削除する
- articles:publish  : 記事を公開する
- comments:read     : コメントを閲覧する
- comments:moderate : コメントを承認・削除する
- users:manage      : ユーザーを管理する
- settings:manage   : システム設定を変更する
```

**パーミッションの粒度:**<br>
- **粗い粒度**: `blog:manage`（ブログ全体の管理権限）<br>
- **細かい粒度**: `articles:edit`, `articles:delete`（個別の操作ごと）<br>

粒度が細かいほど柔軟な制御が可能だが、管理が複雑になる。<br>
システムの要件に応じて適切なレベルを選択する。<br>

---

## ロール（Role）

**ロール（Role）** は、**パーミッションの詰め合わせセット**。<br>
複数のパーミッションをまとめて定義し、それに名前を付けたもの。<br>

**なぜロールが必要か:**<br>
```
パーミッションだけの場合:
- 新しい編集者に10個のパーミッションを1つずつ付与 → 手間がかかる
- 編集者の権限を変更したい → 全員のパーミッションを変更 → 煩雑

ロールを使う場合:
- 「編集者」ロールに10個のパーミッションを定義
- 新しい編集者には「編集者」ロールを1つ付与 → 簡単
- 編集者の権限を変更 → 「編集者」ロールの定義を変更 → 全員に反映
```

**具体例（ブログシステム）:**<br>
```
【読者（Reader）ロール】
- articles:read
- comments:read

【ライター（Writer）ロール】
- articles:read
- articles:create
- articles:edit     ← 自分の記事のみ編集可能（実装による）
- comments:read

【編集者（Editor）ロール】
- articles:read
- articles:create
- articles:edit     ← 全ての記事を編集可能
- articles:delete
- articles:publish
- comments:read
- comments:moderate

【管理者（Admin）ロール】
- すべてのパーミッション
  (または特別な admin フラグで全権限を付与)
```

**ロールの階層:**<br>
一部のシステムでは、ロールの継承や階層構造をサポートする：<br>
```
Admin（管理者）
  ├─ すべてのEditorの権限を含む
  └─ + users:manage, settings:manage

Editor（編集者）
  ├─ すべてのWriterの権限を含む
  └─ + articles:delete, articles:publish, comments:moderate

Writer（ライター）
  ├─ すべてのReaderの権限を含む
  └─ + articles:create, articles:edit

Reader（読者）
  └─ articles:read, comments:read
```

---

## パーミッション vs ロール：まとめ

| 項目 | パーミッション（Permission） | ロール（Role） |
|------|--------------------------|--------------|
| **定義** | 個別の操作権限 | パーミッションの詰め合わせ |
| **粒度** | 細かい（最小単位） | 粗い（複数の権限の集合） |
| **例** | `users:read`, `posts:create` | `Editor`, `Admin` |
| **付与対象** | 通常はロールに付与 | ユーザーに付与 |
| **用途** | 細かい制御が必要な場合 | 役割に応じた権限管理 |
| **管理** | 数が多いと煩雑 | 役割で整理できて管理しやすい |

**基本的な流れ:**<br>
```
1. パーミッションを定義（users:read, posts:create など）
2. ロールを作成し、パーミッションを割り当て（Editor = 編集に必要な権限セット）
3. ユーザーにロールを付与（山田さんに「Editor」ロールを付与）
4. アクセス時に判定（山田さんはEditorロールを持つ → posts:create権限あり → 投稿作成OK）
```

**現場での使い分け:**<br>
- **一般的なケース**: ロールベースで管理（Editor、Admin など）<br>
- **特殊なケース**: 個別にパーミッションを追加（「特定のユーザーだけ billing:view を付与」など）<br>
- **ハイブリッド**: ロール + 個別パーミッションの組み合わせ<br>

---

## RBAC（Role-Based Access Control）

**RBAC（Role-Based Access Control）** は、**ロールベースでアクセス制御を行う方式**。<br>
最も一般的な認可モデルの1つ。<br>

**RBACの基本原則:**<br>
```
ユーザー → ロール → パーミッション → リソース

1. ユーザーはロールを持つ
2. ロールはパーミッションを持つ
3. パーミッションはリソースへのアクセス権を定義
4. ユーザーがリソースにアクセスする際、ロール経由で権限を確認
```

**RBACの利点:**<br>
- **管理が簡単**: ユーザーごとに個別の権限を設定しなくてよい<br>
- **変更に強い**: ロールの定義を変更すれば、該当する全ユーザーに反映<br>
- **監査しやすい**: 誰がどのロールを持っているか追跡できる<br>
- **理解しやすい**: 「編集者」「管理者」など直感的な役割名<br>

**RBACの実装例（データベース設計）:**<br>
```sql
-- ユーザーテーブル
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);

-- ロールテーブル
CREATE TABLE roles (
  id INT PRIMARY KEY,
  name VARCHAR(50),  -- 例: 'editor', 'admin'
  description TEXT
);

-- パーミッションテーブル
CREATE TABLE permissions (
  id INT PRIMARY KEY,
  name VARCHAR(100),  -- 例: 'articles:edit', 'users:delete'
  description TEXT
);

-- ユーザーとロールの関連（多対多）
CREATE TABLE user_roles (
  user_id INT,
  role_id INT,
  PRIMARY KEY (user_id, role_id)
);

-- ロールとパーミッションの関連（多対多）
CREATE TABLE role_permissions (
  role_id INT,
  permission_id INT,
  PRIMARY KEY (role_id, permission_id)
);
```

**権限チェックの例（疑似コード）:**<br>
```javascript
function canUserPerformAction(userId, permission) {
  // 1. ユーザーのロールを取得
  const userRoles = getUserRoles(userId);
  
  // 2. 各ロールが持つパーミッションを取得
  const permissions = [];
  for (const role of userRoles) {
    permissions.push(...getRolePermissions(role.id));
  }
  
  // 3. 必要なパーミッションが含まれているか確認
  return permissions.includes(permission);
}

// 使用例
if (canUserPerformAction(userId, 'articles:delete')) {
  deleteArticle(articleId);
} else {
  throw new Error('権限がありません');
}
```

---

## その他のアクセス制御モデル

RBACの他にも、様々なアクセス制御モデルがある：<br>

### ABAC（Attribute-Based Access Control）
**属性ベースのアクセス制御**。<br>
ユーザー、リソース、環境などの**属性**を組み合わせて、動的に権限を判断する。<br>

```
例: 
「部門が営業で、かつ、勤務時間中で、かつ、機密レベル3以下の資料なら閲覧可能」

RBACとの違い:
- RBAC: 固定的なロールで判定（山田さんは「営業部長」ロール → 閲覧可）
- ABAC: 属性の組み合わせで柔軟に判定（時間帯、場所、リソースの属性なども考慮）
```

### ACL（Access Control List）
**アクセス制御リスト**。<br>
リソースごとに「誰がアクセスできるか」のリストを持つ方式。<br>

```
例（ファイルのACL）:
article_123.md
  - user_1: read, write
  - user_2: read
  - user_3: read, write, delete
```

ファイルシステムのパーミッション（`chmod`、NTFS権限など）がACLの例。<br>

### ポリシーベース
**複雑なルールをポリシーとして記述**する方式。<br>
例: AWS IAMポリシー、Kubernetes RBAC、OPA（Open Policy Agent）など。<br>

```json
// AWS IAMポリシーの例
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "192.0.2.0/24"
        }
      }
    }
  ]
}
```

---

## 関連する用語

### プリンシパル（Principal）
**権限を持つ主体**のこと。<br>
ユーザー、サービスアカウント、APIキーなど、アクセスを要求する側。<br>

```
例:
- ユーザー「山田太郎」
- サービスアカウント「payment-service」
- APIキー「12345-abcde」
```

### リソース（Resource）
**保護される対象**のこと。<br>
ファイル、データベースのレコード、API エンドポイントなど。<br>

```
例:
- ファイル「/docs/secret.pdf」
- データベースの「users」テーブル
- APIエンドポイント「/api/admin/settings」
```

### アクション（Action）
**リソースに対する操作**のこと。<br>
読み取り（Read）、書き込み（Write）、削除（Delete）など。<br>

```
例:
- Create（作成）
- Read（読み取り）
- Update（更新）
- Delete（削除）
→ CRUD操作
```

### スコープ（Scope）
**権限の適用範囲**のこと。<br>
OAuth 2.0 などで、アクセストークンが持つ権限の範囲を表す。<br>

```
例（OAuth 2.0のスコープ）:
- read:user    → ユーザー情報の読み取り
- write:repo   → リポジトリへの書き込み
- admin:org    → 組織の管理
```

---

## 実装時の注意点

### 1. 最小権限の原則（Principle of Least Privilege）
ユーザーには**必要最小限の権限のみを付与**する。<br>
デフォルトは権限なし、必要に応じて追加する。<br>

```
悪い例: 全員に「Admin」ロールを付与
良い例: 通常は「User」ロール、管理業務が必要な人にのみ「Admin」を付与
```

### 2. デフォルト拒否（Default Deny）
明示的に許可されていない操作は、**すべて拒否**する。<br>

```javascript
// 悪い例: 権限がない場合に何もしない（暗黙の許可）
if (hasPermission(user, 'articles:delete')) {
  // 削除処理
} else {
  // 何もしない → 処理が続行されてしまう可能性
}

// 良い例: デフォルトで拒否
if (!hasPermission(user, 'articles:delete')) {
  throw new UnauthorizedError('削除権限がありません');
}
deleteArticle(articleId);
```

### 3. 権限チェックの場所
**サーバーサイドで必ず権限をチェック**する。<br>
クライアント側（フロントエンド）のチェックは、UIの表示制御のみに使用。<br>

```
クライアント側: ボタンの表示/非表示を制御（UX向上）
サーバー側: 実際の操作前に権限を検証（セキュリティ担保）
```

### 4. 監査ログ
権限の変更やアクセス試行を**ログに記録**する。<br>
セキュリティインシデントの調査や、コンプライアンス対応に必要。<br>

```
ログの例:
- 2026-02-18 10:30:15 - User:yamada - Action:articles:delete - Result:SUCCESS
- 2026-02-18 10:31:42 - User:tanaka - Action:users:manage - Result:DENIED
```

---

## まとめ

```
認可の階層構造:
ユーザー（主体）
  ↓ 付与
ロール（役割）
  ↓ 含む
パーミッション（個別の権限）
  ↓ 制御
リソース（保護対象）

単語の整理:
- パーミッション = 個別の操作権限（articles:read）
- ロール = パーミッションの詰め合わせセット（Editor）
- 認可 = 「誰が何をできるか」を制御する仕組み
- RBAC = ロールベースでアクセス制御を行う方式
```

**現場でよくある呼び方:**<br>
- 「権限」= パーミッションを指すことが多い<br>
- 「ロール」= 役割（Editor、Adminなど）<br>
- 「ロールを付与する」= ユーザーに役割を割り当てる<br>
- 「権限がない」= 必要なパーミッションを持っていない<br>

**理解のポイント:**<br>
単に「権限」と言っても、**個別の操作権限（パーミッション）**なのか、**役割としてまとめられた権限セット（ロール）**なのかで意味が異なる。<br>
文脈に応じて使い分けることが重要。<br>
