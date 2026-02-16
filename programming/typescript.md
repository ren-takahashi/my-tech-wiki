# TypeScript 要点まとめ

## TypeScriptとは
TypeScriptはJavaScriptに型システムを追加した言語。<br>
JavaScriptのスーパーセットであり、JavaScriptコードはそのままTypeScriptとしても成立する。<br>

**関係性:**<br>
```
JavaScript ⊂ TypeScript
```

---

## 実行モデル
TypeScriptは直接実行されない。<br>
必ずJavaScriptに変換（コンパイル）されてから実行される。<br>

**構造:**<br>
```
TypeScript（開発言語）
↓ コンパイル
JavaScript（実行言語）
↓
ブラウザ / Node.js
```

**重要**: 型情報はコンパイル時に消える。<br>

---

## TypeScriptの役割
TypeScriptは主に「開発時の安全性」を提供する。<br>

- 型チェック<br>
- IDE補完の強化<br>
- バグの早期検出<br>
- リファクタリングの安全性向上<br>
- チーム開発の安定化<br>

**実行速度や実行結果には影響しない。**<br>

---

## JavaScriptとの違い
**JavaScript:**<br>
動的型付け言語（実行時に型エラーが発生）<br>

**TypeScript:**<br>
静的型チェックを持つ（実行前にエラー検出）<br>

**例:**<br>

```javascript
// JavaScript:
function add(a, b) {
  return a + b
}

// TypeScript:
function add(a: number, b: number): number {
  return a + b
}
```

---

## 主要な型の紹介
TypeScriptで最低限押さえておくべき基本的な型。<br>

### プリミティブ型（基本型）
```typescript
// 文字列
let name: string = "太郎";

// 数値
let age: number = 25;

// 真偽値
let isActive: boolean = true;
```

### 特殊な型
```typescript
// any: どんな型でも許可（型チェックを無効化）
let anything: any = "文字列";
anything = 123;  // エラーにならない
anything = true; // エラーにならない
// ※ anyは型安全性を失うため、極力使わない

// unknown: 安全なany（使用前に型チェックが必要）
let value: unknown = "何か";
// value.toUpperCase(); ← エラー（型チェックなしで使えない）
if (typeof value === "string") {
  value.toUpperCase(); // ← OK（型チェック後なら使える）
}

// void: 戻り値なし（関数用）
function log(message: string): void {
  console.log(message);
  // return はなし
}

// null と undefined
let empty: null = null;
let notDefined: undefined = undefined;
```

### 配列とオブジェクト
```typescript
// 配列
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["太郎", "花子"];

// オブジェクト
let user: { name: string; age: number } = {
  name: "太郎",
  age: 25
};
```

### ユニオン型（複数の型を許可）
```typescript
let id: string | number;
id = "ABC123";  // OK
id = 123;        // OK
// id = true;    // エラー
```

**実務でよく使うのは:**<br>
- `string`, `number`, `boolean`（基本）<br>
- `void`（関数の戻り値なし）<br>
- `any`（型不明時の逃げ道、ただし乱用禁止）<br>
- `unknown`（anyより安全な「型不明」）<br>
- ユニオン型（`string | number`のように複数型を許可）<br>

---

## ReactとTypeScript
ReactはTypeScriptなしでも開発可能。<br>
ただし実務ではReact + TypeScriptが主流。<br>

**理由:**<br>
- propsの型管理が重要<br>
- コンポーネント間のデータ契約が明確になる<br>
- 大規模開発で壊れにくい<br>

**関係性:**<br>
- **React** = UIを作る仕組み<br>
- **TypeScript** = 安全に作る仕組み<br>

---

## 型定義ファイル（.d.ts と @types）
TypeScriptでJavaScriptライブラリを使う際に必要になる仕組み。<br>

### 問題：JavaScriptライブラリには型情報がない
```typescript
// React（元々はJavaScript製）を使う場合
import React from 'react';

// TypeScriptは「Reactって何？型は？」がわからない
// → エラーになる
```

**JavaScriptで書かれたライブラリには型情報がない。**<br>
TypeScriptから使おうとすると、型が分からずエラーになる。<br>

### 解決策：型定義ファイル（.d.ts）
型定義ファイル = 「このライブラリの型情報」を記述したファイル。<br>

```typescript
// react.d.ts（型定義ファイルの例）
export function useState<T>(initialState: T): [T, (newState: T) => void];
// ↑ 「useStateという関数があって、こういう型です」と定義
```

拡張子が `.d.ts` のファイルが型定義ファイル。<br>
**実際のコードではなく、型情報だけを記述する。**<br>

### @types パッケージ
**@types = npmで配布されている型定義ファイル集。**<br>

```bash
# Reactの型定義をインストール
npm install --save-dev @types/react

# Node.jsの型定義をインストール
npm install --save-dev @types/node

# jQueryの型定義をインストール
npm install --save-dev @types/jquery
```

**仕組み:**<br>
```
1. React本体（JavaScript製）をインストール
   npm install react

2. Reactの型定義をインストール
   npm install --save-dev @types/react

3. TypeScriptが自動的に型定義を読み込む
   → Reactをエラーなく使える！
```

> **注意**: React v18以降（2022年3月〜）は型定義が本体に同梱されているため、実際には`@types/react`のインストールは不要。<br>
> ただし、Node.js（`@types/node`）、Express（`@types/express`）などは現在（2026年2月 現在）でも型定義パッケージが必要。<br>

### いつ必要？
**JavaScriptで書かれたライブラリを使う時。**<br>

- `@types/react` - React<br>
- `@types/node` - Node.js<br>
- `@types/express` - Express<br>
- `@types/lodash` - Lodash<br>

**最初からTypeScriptで書かれたライブラリは不要。**<br>
（例: Nest.js、Prismaなど）<br>

### よくあるエラーと対処法
```typescript
// エラー: Could not find a declaration file for module 'some-library'
import something from 'some-library';

// 対処法1: @typesパッケージをインストール
npm install --save-dev @types/some-library

// 対処法2: @typesがない場合は自分で型定義を作る
// src/types/some-library.d.ts
declare module 'some-library' {
  export function someFunction(): void;
}

// 対処法3: 型チェックを諦める（非推奨）
declare module 'some-library'; // anyとして扱われる
```

### package.jsonでの確認
```json
{
  "dependencies": {
    "react": "^18.0.0"  // ← ライブラリ本体
  },
  "devDependencies": {
    "@types/react": "^18.0.0"  // ← 型定義（開発時のみ必要）
  }
}
```

**ポイント:**<br>
- `@types/*` は開発時のみ必要（本番環境では不要）<br>
- TypeScriptコンパイル後のJavaScriptには型情報が含まれない<br>
- だから `devDependencies` に入れる<br>

**実務での流れ:**<br>
1. ライブラリをインストール: `npm install react`<br>
2. TypeScriptでimportするとエラー<br>
3. 「型定義がない」と気づく<br>
4. `npm install --save-dev @types/react` で解決<br>

---

## TypeScriptの本質
TypeScriptは「JavaScriptを置き換える言語」ではない。<br>
JavaScriptを安全に書くための開発用言語。<br>

**言語というより:**<br>
- 型検証システム<br>
- 開発支援ツール<br>
- コンパイラ（tsc）<br>

の集合体に近い。<br>

---

## 重要な理解ポイント
- TypeScriptはJavaScriptではない（独立した言語）<br>
- ただしJavaScriptを完全に含む<br>
- 実行されるのは常にJavaScript<br>
- 型は実行時には存在しない<br>
