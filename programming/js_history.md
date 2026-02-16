# JavaScriptの進化の流れ

## ① 誕生期（1995〜2005）
**「ブラウザにちょっとした動きを付ける言語」**

### 1995年
JavaScript誕生（Netscape）<br>
目的：ブラウザ上の簡単な動作<br>

**当時の用途:**<br>
- フォーム入力チェック<br>
- クリック時の挙動<br>
- 簡単なDOM操作<br>

**この時代の特徴:**<br>
- ブラウザごとに挙動が違う<br>
- 開発がとても大変<br>
- 規模の大きいアプリは作らない<br>

**つまり:** JS = 補助スクリプト<br>

---

## ② jQuery時代（2006〜2015）
**「ブラウザ差異を吸収する救世主」**

### 2006年
jQuery登場<br>
これが革命的でした。<br>

**例えば当時の問題:**<br>
```javascript
document.getElementById("id")
// ブラウザごとに書き方が違う問題がありました
```

**jQueryの解決策:**<br>
```javascript
$("#id")
```

**jQueryが提供したもの:**<br>
- Ajaxが簡単<br>
- DOM操作が簡単<br>
- アニメーションが簡単<br>
- イベント処理が簡単<br>

**2008〜2014あたりはほぼ標準でした。**<br>

**この時代の主な用途:**<br>
- WordPress<br>
- コーポレートサイト<br>
- LP（ランディングページ）<br>
- CMS<br>

ほぼ全部 jQuery。<br>
2019年でもまだ現場では普通に使われていました。<br>

---

## ③ JavaScript進化（ES6革命）（2015〜）
**ここが重要な転換点。**

### 2015年
**ECMAScript2015（ES6）**<br>

**追加されたもの:**<br>
- `let` / `const`（変数宣言）<br>
- アロー関数（`() => {}`）<br>
- `class`（クラス構文）<br>
- `import` / `export`（モジュール）<br>
- `Promise`（非同期処理）<br>

**それ以前:**<br>
- スケールする言語ではなかった<br>
- 設計しづらかった<br>

**ES6以降:**<br>
- 大規模開発可能な言語になった<br>

**ここでJSが「まともな言語」になりました。**<br>

---

## ④ Node.jsの登場（2009〜現在）
**「JSがサーバーでも動くようになった」**

### 2009年
Node.js誕生<br>

これは概念的に重要。<br>

**それまで:**<br>
- JS = ブラウザ<br>
- PHP / Java / Ruby = サーバー<br>

**Node.js以降:**<br>
JSがサーバーで動く<br>

**つまり以下が可能に:**<br>
- API開発<br>
- CLIツール<br>
- ビルドツール<br>
- Webサーバー<br>

全部JSで可能になりました。<br>

**npmの誕生も大きい。**<br>
現在のフロント開発はほぼ全てNodeベース。<br>

---

## ⑤ SPAフレームワーク時代（2013〜現在）
**ここでReactやVueが登場します。**

### Angular（2010）
最初の大規模フレームワーク<br>

### React（2013）
Facebook製<br>
**ここが大きな分岐点。**<br>

**思想:**<br>
- UI = 状態から生成<br>
- 仮想DOM<br>
- コンポーネント指向<br>

**「DOM操作を直接しない」世界に。**<br>

### Vue（2014）
Reactより簡単<br>
日本では人気が高い<br>

---

## ⑥ jQueryが衰退した理由

### 理由1: ブラウザ差異が減った
モダンブラウザ統一により、以下が普通に使えるようになった。<br>
```javascript
document.querySelector()
```

### 理由2: フレームワークがDOM操作を隠した
React/VueではDOMを直接触らない。<br>

### 理由3: ES6でJSが進化した
jQueryの「便利さ」が不要になった。<br>

---

## ⑦ TypeScriptの登場（2012→2018以降普及）
TypeScriptが普及したのは比較的最近。<br>

### 2018〜現在
**React + TypeScript が主流。**<br>

**理由:**<br>
- 大規模開発に必要<br>
- IDE補完が強い<br>
- バグが減る<br>

現場でTypeScriptは標準になりつつあります。<br>

---

## 2019年（業界参入時期）の状況
**2019年はちょうど過渡期。**<br>

**現場状況:**<br>
- **jQuery** → まだ多い<br>
- **React** → 急増中<br>
- **Vue** → 人気<br>
- **TypeScript** → 使う会社が増え始める<br>

**つまり:**<br>
「jQuery世代の最後のタイミング」に入った時期。<br>

---

## 2026年現在の状況（実務ベース）

### フロントエンド
- **React**（最多）<br>
- **TypeScript**（ほぼ必須）<br>
- **Next.js**（主流）<br>

### バックエンド
- Node.js<br>
- Python<br>
- Go<br>
- Java<br>
- PHP（Laravel中心）<br>

### jQuery
**保守案件のみ**<br>
新規開発ではほぼ使われない。<br>

---

## まとめ：JavaScriptの進化

```
1995年   JavaScript誕生 → 補助スクリプト
2006年   jQuery登場 → ブラウザ差異を吸収
2009年   Node.js → サーバーサイドでもJS
2013年   React登場 → SPA時代へ
2015年   ES6 → まともな言語に進化
2018年〜 TypeScript普及 → 大規模開発の標準
2026年   React + TypeScript + Next.js が主流
```
JavaScriptは「補助スクリプト」から「アプリケーション開発の中心言語」に進化した。<br>
その過程でjQuery、React、TypeScriptという大きな転換点があった。<br>
