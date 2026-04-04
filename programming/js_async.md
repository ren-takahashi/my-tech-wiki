# JavaScriptの非同期処理

## 1. 全体像

JavaScriptには2つの軸がある：

* **イベント**：いつ起きるかわからない（クリックなど）
* **非同期**：いつ終わるかわからない（API通信など）

ここでは非同期について掘り下げていく。

---

## 2. 非同期とは何か

> **「結果が後で返ってくる処理」**

### 同期との違い

**同期:**

```javascript
const a = 1 + 2;
console.log(a); // 3
```

その場で結果が確定

**非同期:**

```javascript
const result = fetch("/api");
console.log(result); // Promiseだけが返る（「5. Promiseとは何か」を参照）
```

APIの結果を待たないので、resultの中身はまだデータが入っていない状態の「箱」だけが返ってきてしまう。
※ JSは非同期がデフォルトなので、待つときだけ明示する必要がある（後述）。

---

## 3. なぜ非同期が必要か

**理由:**

> **処理を止めないため**

もし全部同期なら：

```javascript
const data = 重い処理(); // 数秒ここで止まる
console.log("次");
```

数秒間すべて停止してしまう。

実際のJS：

```javascript
const data = fetch("/api"); // ここはすぐ次へ進む（非同期だから）
console.log("次");
```

すぐ次の処理へ進む

---

## 4. ここが重要：「待たない」がデフォルト

実際のコードではAPI実行処理にawaitをつけることが多い。  

> 「awaitが必要なのはなぜ？」

→ **「待たないのがデフォルトだから、待つときだけ明示するために await を使う」**

### デフォルトの挙動

```javascript
const data = fetch("/api"); // 待たない
```
Promiseが返るだけ。

### 待つ場合

```javascript
const data = await fetch("/api");
```
ここだけ待つ。

---

## 5. Promiseとは何か（最重要ポイント）

### 定義

> **Promise =「未来に値が入る箱」**

### 例

```javascript
const result = fetch("/api");
```

この `result` は：

```text
「まだデータはないが、後で入る箱」
```

### 状態

* pending（待機中）
* fulfilled（成功）
* rejected（失敗）

### イメージ

```text
今：空箱
後：データが入る
```

---

## 6. 「Promiseが返る」とは

```javascript
const result = getData();
```

意味：

> **データではなく「Promise（箱）」が返ってきている**

### よくある誤解

```javascript
console.log(result);
```

出るのは：

```text
Promise { <pending> }
```
※ まだ中身がないため。

---

## 7. 中身の取り出し方

### 方法① then

```javascript
getData().then((data) => {
  console.log(data);
});
```

### 方法② await（主流）

```javascript
const data = await getData();
```

### awaitの意味

```javascript
const data = await getData();
```

**「Promiseが解決するまで待って、中身を取り出す」**

---

## 8. awaitの重要な誤解

### ❌ 同期になる

違う

### ⭕ 非同期のまま、書き方だけ順番になる

正しい

### 内部イメージ

```javascript
console.log("A");

const data = await fetch("/api");

console.log("B");
```

実際：

```text
A
↓
fetch開始
↓（一旦抜ける）
↓
データ到着
↓
B
```

---

## 9. Promiseの正体（少し深く）

```javascript
new Promise((resolve, reject) => {
  resolve("OK");
});
```
* resolve → 成功して値を入れる
* reject → 失敗として値を入れる

### 重要

> **Promise = 非同期処理そのものではない**

> **Promise = 非同期処理の結果を表すもの**

---

## 10. コールバックとの関係

昔：

```javascript
getData((data) => {
  console.log(data);
});
```

今：

```javascript
const data = await getData();
```

本質は同じ：

> **「終わったら続きをやる」**

---

## 11. 非同期の本質まとめ

非同期は必ずこうなる：

```text
① 処理開始
② 待たずに次へ
③ 後で結果が来る
```

---

## 12. PHPとの違い（重要）

### PHP

```php
$data = getData();
```

自動で待つ

### JS

```javascript
const data = getData();
```

待たない（Promise）

だから

```javascript
const data = await getData();
```

が必要

---

## 13. 一旦これだけ覚えればOK

* Promise = 未来の値の箱
* await = 完了待ちするための記述
* デフォルトは待たない

---

## 14. 一行まとめ

> **JavaScriptは「待たない設計」なので、「待つときだけ await で明示する」**

---

## 15. Promiseチェーン vs async/await

### Promiseチェーン（.then）

```javascript
fetch("/api/user")
  .then(res => res.json())
  .then(user => fetch(`/api/posts?userId=${user.id}`))
  .then(res => res.json())
  .then(posts => console.log(posts))
  .catch(err => console.error(err));
```

### async / await

```javascript
async function main() {
  try {
    const res1 = await fetch("/api/user");
    const user = await res1.json();

    const res2 = await fetch(`/api/posts?userId=${user.id}`);
    const posts = await res2.json();

    console.log(posts);
  } catch (err) {
    console.error(err);
  }
}
```

### 違い（本質）

| 観点 | Promiseチェーン | async/await |
| --- | --- | --- |
| 書き方 | 関数をつなぐ | 上から順に書く |
| 可読性 | ネスト/分岐で崩れる | 読みやすい |
| エラーハンドリング | `.catch()` | `try/catch` |

### 本質

> **やっていることは同じ（Promiseをつないでいる）**

違うのは「書き方」だけ

### いつ使うか

**async/await（基本これ）:**

* 順番がある処理
* 通常の業務コード

**デフォルト選択**

**Promiseチェーン:**

* 短い処理
* 関数的に書きたい場合
* map + 非同期など

サブ用途

---

## 16. 非同期の「2パターン」

ここがかなり重要

### パターン①：順番が必要（直列）

```javascript
const user = await getUser();
const posts = await getPosts(user.id);
```

BはAに依存

### パターン②：独立している（並列可能）

```javascript
const user = await getUser();
const news = await getNews();
```

本当は同時にできる

---

## 17. Promise.all（並列処理）

ここで登場

### 基本形

```javascript
const [user, news] = await Promise.all([
  getUser(),
  getNews()
]);
```

### 何が起きているか

```text
① getUser() 開始
② getNews() 開始
③ 両方同時に進む
④ 両方終わったら次へ
```

### 直列との違い

**❌ 直列（遅い）:**

```javascript
const user = await getUser(); // 1秒
const news = await getNews(); // 1秒
```

合計：2秒

**⭕ 並列（速い）:**

```javascript
const [user, news] = await Promise.all([
  getUser(),
  getNews()
]);
```

合計：1秒

---

## 18. Promise.allの注意点

### ① 1つでも失敗すると全部失敗

```javascript
Promise.all([
  success(),
  fail(),
]);
```

全体が reject

### ② 順番は維持される

```javascript
const [a, b] = await Promise.all([A(), B()]);
```

A, B の順で入る（完了順ではない）

---

## 19. よくある実務パターン

### API複数取得

```javascript
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);
```

### map + 非同期（重要）

```javascript
const results = await Promise.all(
  ids.map(id => fetchUser(id))
);
```

これは頻出する

---

## 20. NGパターン（よくあるミス）

```javascript
ids.map(async (id) => {
  return await fetchUser(id);
});
```

これだけだとダメ（awaitしてない）

### 正しくは

```javascript
const users = await Promise.all(
  ids.map(id => fetchUser(id))
);
```

---

## 21. 判断基準（最重要）

### これで決める

**Q1：依存関係ある？**

* YES → await（直列）
* NO → Promise.all（並列）

### まとめ図

```text
依存あり → awaitで順番
依存なし → Promise.allで同時
```

---

## 22. 一段抽象化

* await = 「順番制御」
* Promise.all = 「同時実行」

---

## 23. 一行で核心

> **非同期は「順番にやるか」「同時にやるか」**
