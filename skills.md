# テキスト化ランディングページ作成ツール 仕様書

## 1. ツール概要

本ツールは、ECモール（楽天市場・Yahoo!ショッピング）の商品ページにおいて、テキストベースのランディングページ（LP）を効率的に作成するためのフレームワークである。あらかじめ定義されたHTMLブロックとCSSテンプレートを組み合わせ、テキストと画像URLを入力するだけで、統一されたデザインのLPを出力する。

---

## 2. 対象プラットフォーム

| プラットフォーム | 画像ホスティング | HTML設置場所 |
|---|---|---|
| 楽天市場 | 楽天キャビネット（R-Cabinet） | 商品説明文（PC・スマホ） |
| Yahoo!ショッピング | トリプル（Triple） | 商品説明文・フリースペース |

---

## 3. 設計方針

- **テンプレート駆動**: 7種類のHTMLブロックを組み合わせてページを構成する
- **CSS一元管理**: 共通CSSテンプレート1ファイルでデザインを制御する
- **入力の最小化**: ユーザーが入力するのは「テキスト」と「画像URL」のみ
- **レスポンシブ対応**: PC（幅800px）とスマホ（幅768px以下）の両方に対応済み

---

## 4. ページ構成（上から下への標準レイアウト）

```
┌─────────────────────────────────┐
│  商品タイトルエリア               │ ← ブロック①
│  （英語名 / 車種 / 日本語名 / コピー）│
├─────────────────────────────────┤
│  メイン画像（800×400）            │ ← ブロック②
├─────────────────────────────────┤
│  ═══ 帯見出し「製品の特長」═══     │ ← ブロック③
├─────────────────────────────────┤
│  見出し（大）＋本文              │ ← ブロック④
├─────────────────────────────────┤
│  見出し（小）＋本文              │ ← ブロック⑤
├─────────────────────────────────┤
│  2カラム（画像＋テキスト×2列）    │ ← ブロック⑦
├─────────────────────────────────┤
│  ═══ 帯見出し「スペック」═══      │ ← ブロック③
├─────────────────────────────────┤
│  スペック表                      │ ← ブロック⑥
├─────────────────────────────────┤
│  ═══ 帯見出し「注意事項」═══      │ ← ブロック③
├─────────────────────────────────┤
│  見出し（小）＋本文              │ ← ブロック⑤
└─────────────────────────────────┘
```

> 上記は標準構成の一例。ブロックの順序・繰り返し・省略は自由に行える。

---

## 5. HTMLブロック定義

### ブロック① 商品タイトル

**用途**: ページ最上部の商品名・適合車種・キャッチコピーを表示する。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 商品名（英語） | ブランド名やシリーズ名（英語表記） | `AERO GUARD PLUS` |
| 適合車種 | 対応する車種名 | `TOYOTA GR86 / SUBARU BRZ (ZN8/ZD8)` |
| 商品名（日本語） | 商品の日本語正式名称 | `エアロガード プラス フロントリップスポイラー` |
| キャッチコピー | 訴求文（1〜2行程度） | `空力性能と美しさを両立する、次世代エアロパーツ` |

**HTMLテンプレート**:

```html
<div class="lp-wrapper">
    <div class="lp-header-top" style="display: flex; justify-content: space-between; align-items: flex-end; flex-wrap: wrap;">
        <div class="lp-title-main">{{商品名_英語}}</div>
        <div class="lp-title-carmodel">{{適合車種}}</div>
    </div>
    <div class="lp-title-jp">{{商品名_日本語}}</div>
    <div class="lp-catch">{{キャッチコピー}}</div>
</div>
```

---

### ブロック② 画像

**用途**: 任意のサイズの画像を表示する。メイン画像、サブ画像など用途に応じて使い分ける。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 画像URL | 楽天キャビネットまたはトリプルのURL | `https://image.rakuten.co.jp/xxxxx/cabinet/img001.jpg` |
| alt属性 | 画像の代替テキスト | `フロントリップスポイラー装着イメージ` |
| サイズクラス | 画像サイズ指定用のCSSクラス | `img-800-400`（下表参照） |

**画像サイズクラス一覧**:

| クラス名 | PC表示サイズ | 用途 | スマホ時の挙動 |
|---|---|---|---|
| `img-800-400` | 800×400px | メイン画像・大画像 | 幅100%、高さ自動（2:1比率維持） |
| `img-800-200` | 800×200px | 横長バナー・取付イメージ | 幅100%、高さ自動（4:1比率維持） |
| `img-388-240` | 388×240px | 2カラム用画像 | 幅100%、高さ自動（比率維持） |

**HTMLテンプレート**:

```html
<div class="lp-img-wrap {{サイズクラス}}">
    <img src="{{画像URL}}" alt="{{alt属性}}">
</div>
```

---

### ブロック③ 帯見出し

**用途**: セクション区切りとして上下に罫線のある帯状の見出しを表示する。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 帯テキスト | セクション名 | `製品の特長` / `スペック` / `注意事項` |

**HTMLテンプレート**:

```html
<div class="lp-band-title">{{帯テキスト}}</div>
```

---

### ブロック④ 見出し（大）＋本文

**用途**: セクション内の大見出しと説明文を表示する。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 見出しテキスト | 大見出し | `高剛性ウレタン素材を採用` |
| 本文テキスト | 説明文（複数行可） | `独自開発の高剛性ウレタンにより...` |

**HTMLテンプレート**:

```html
<div class="text-heading-lg">{{見出しテキスト}}</div>
<div class="text-body">
    {{本文テキスト}}
</div>
```

---

### ブロック⑤ 見出し（小）＋本文

**用途**: セクション内の小見出しと説明文を表示する。ブロック④より一段小さい見出し。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 見出しテキスト | 小見出し | `取り付け方法について` |
| 本文テキスト | 説明文（複数行可） | `付属の専用ボルトを使用して...` |

**HTMLテンプレート**:

```html
<div class="text-heading-md">{{見出しテキスト}}</div>
<div class="text-body">
    {{本文テキスト}}
</div>
```

---

### ブロック⑥ スペック表

**用途**: 商品仕様をテーブル形式で表示する。奇数行にストライプ背景あり。

**入力項目**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 項目名（左列） | スペック項目名 | `素材` |
| 値（右列） | スペック値 | `高剛性ウレタン（PUR）` |

> 行数は任意。必要な行数分だけ `<tr>` を追加する。

**HTMLテンプレート**:

```html
<table class="lp-spec-table">
    <tbody>
        <tr>
            <th>{{項目名_1}}</th>
            <td>{{値_1}}</td>
        </tr>
        <tr>
            <th>{{項目名_2}}</th>
            <td>{{値_2}}</td>
        </tr>
        <!-- 必要な行数分繰り返す -->
    </tbody>
</table>
```

---

### ブロック⑦ 2カラムレイアウト

**用途**: 画像＋見出し＋本文を横2列で並べて表示する。スマホでは縦積みになる。

**入力項目（左カラム・右カラムそれぞれ）**:

| 入力項目 | 説明 | 入力例 |
|---|---|---|
| 画像URL | 楽天キャビネットまたはトリプルのURL | `https://image.rakuten.co.jp/xxxxx/cabinet/img002.jpg` |
| alt属性 | 画像の代替テキスト | `正面からの装着イメージ` |
| 見出しテキスト | 小見出し | `フロントビュー` |
| 本文テキスト | 説明文 | `低重心デザインが...` |

**HTMLテンプレート**:

```html
<div class="lp-grid-2">
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="{{画像URL_左}}" alt="{{alt属性_左}}">
        </div>
        <div class="text-heading-md">{{見出し_左}}</div>
        <div class="text-body">
            {{本文_左}}
        </div>
    </div>
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="{{画像URL_右}}" alt="{{alt属性_右}}">
        </div>
        <div class="text-heading-md">{{見出し_右}}</div>
        <div class="text-body">
            {{本文_右}}
        </div>
    </div>
</div>
```

---

## 6. CSS仕様

### 6.1 基本設定

| 項目 | 値 |
|---|---|
| 最大幅 | 800px |
| 配置 | 中央寄せ（`margin: 0 auto`） |
| 背景色 | #FFFFFF |
| レスポンシブ切替 | 768px以下でスマホレイアウトに切替 |

### 6.2 フォント設定

| 要素 | フォント | ウェイト | サイズ（PC） |
|---|---|---|---|
| 英語タイトル | Kozuka Gothic Pr6N | 900（Black） | clamp(22px, 4.7vw, 28pt) |
| 適合車種 | Kozuka Gothic Pr6N | 900（Black） | clamp(11px, 2.4vw, 14pt) |
| 日本語タイトル | Hiragino Kaku Gothic ProN | 600（Semi Bold） | clamp(10px, 1.5vw, 9pt) |
| キャッチコピー | Hiragino Kaku Gothic ProN | 600（Semi Bold） | clamp(13px, 2.2vw, 13pt) |
| 帯見出し | Hiragino Kaku Gothic ProN | 300（Light） | clamp(14px, 2.5vw, 15pt) |
| 大見出し | Hiragino Kaku Gothic ProN | 600（Semi Bold） | clamp(16px, 2.5vw, 17pt) |
| 中見出し | Hiragino Kaku Gothic ProN | 600（Semi Bold） | clamp(15px, 2.5vw, 16pt) |
| 本文 | Hiragino Kaku Gothic ProN | 300（Light） | clamp(12px, 2.0vw, 12pt) |
| スペック表 TH | Hiragino Kaku Gothic ProN | 300（Light） | clamp(11px, 2.0vw, 12pt) |
| スペック表 TD | Hiragino Kaku Gothic ProN | 300（Light） | clamp(11px, 2.0vw, 11pt) |

### 6.3 CSSファイル

CSSはプロジェクト内の「CSSテンプレート」をそのまま使用する。編集不要。完全なCSSソースは本仕様書末尾の「付録A」を参照。

---

## 7. 画像ホスティングルール

### 7.1 楽天キャビネット（R-Cabinet）

| 項目 | 仕様 |
|---|---|
| URL形式 | `https://image.rakuten.co.jp/{ショップID}/cabinet/{フォルダ}/{ファイル名}` |
| 推奨フォーマット | JPG / PNG |
| 推奨画像サイズ | メイン: 800×400px / 2カラム: 388×240px / バナー: 800×200px |
| 注意事項 | ファイル名は半角英数字・ハイフン・アンダースコアのみ使用 |

### 7.2 Yahoo!ショッピング トリプル（Triple）

| 項目 | 仕様 |
|---|---|
| URL形式 | `https://shopping.c.yimg.jp/lib/{ストアID}/{パス}/{ファイル名}` |
| 推奨フォーマット | JPG / PNG |
| 推奨画像サイズ | 楽天と同一サイズを推奨 |
| 注意事項 | トリプルのストレージ容量に注意（プランにより上限あり） |

---

## 8. LP作成ワークフロー

### Step 1: 情報整理

以下の入力シートに情報を記入する。

```
【商品タイトル】
  商品名（英語）: 
  適合車種: 
  商品名（日本語）: 
  キャッチコピー: 

【メイン画像】
  画像URL: 
  alt: 

【セクション1: 製品の特長】
  帯テキスト: 製品の特長
  
  大見出し: 
  本文: 
  
  --- 2カラム ---
  左画像URL: 
  左alt: 
  左見出し: 
  左本文: 
  右画像URL: 
  右alt: 
  右見出し: 
  右本文: 

【セクション2: スペック】
  帯テキスト: スペック
  
  項目1: _____ / 値1: _____
  項目2: _____ / 値2: _____
  項目3: _____ / 値3: _____

【セクション3: 注意事項】
  帯テキスト: 注意事項
  
  小見出し: 
  本文: 
```

### Step 2: HTMLブロックの組み立て

Step 1の情報をもとに、ブロック①〜⑦を上から順に組み合わせてHTMLを生成する。

### Step 3: CSS統合と出力

CSSテンプレートの `<style>` タグ内に全CSSを配置し、`<body>` 内にStep 2で作成したHTMLを挿入して、完成版HTMLファイルを出力する。

### Step 4: プラットフォームへの反映

- **楽天市場**: 商品編集画面の「PC用商品説明文」にHTMLを貼り付け
- **Yahoo!ショッピング**: 商品編集画面の「フリースペース」または「商品説明」にHTMLを貼り付け

---

## 9. 出力ファイル構成

最終出力は**単一HTMLファイル**とする。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Product LP</title>
<style>
/* === CSSテンプレート全文をここに配置 === */
</style>
</head>
<body>

<!-- ブロック①〜⑦を組み合わせたHTMLをここに配置 -->

</body>
</html>
```

---

## 10. 注意事項・制約

### 10.1 ECモール固有の制約

- 楽天市場・Yahoo!ショッピングともに、使用できるHTMLタグ・CSS属性に制限がある場合がある。JavaScript は使用不可。
- `<style>` タグの使用可否はモールの仕様変更により変わる可能性がある。インラインスタイルへの変換が必要になる場合に備えること。
- 外部CSSファイルの読み込み（`<link>`タグ）は使用不可。

### 10.2 画像について

- 画像は必ず楽天キャビネットまたはトリプルにアップロード済みのURLを使用する。外部画像サービスのURLは使用不可。
- 画像は事前に推奨サイズにリサイズ・トリミングしてからアップロードすること。
- alt属性は必ず記述する（アクセシビリティおよびSEO対策）。

### 10.3 テキストについて

- 本文中で改行が必要な場合は `<br>` タグを使用する。
- 特殊文字（`&`, `<`, `>`, `"` 等）はHTMLエンティティに変換する。
- 機種依存文字の使用は避ける。

---

## 付録A: CSSテンプレート全文

```css
/* =================================================================
   1. 全体の設定 (Wrapper)
================================================================= */
.lp-wrapper {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 0;
  border: none;
  background-color: #fff;
  text-align: left;
  overflow-x: hidden;
  box-sizing: border-box;
}

/* =================================================================
   2. 商品タイトルエリア (Header Typography)
================================================================= */
.lp-title-main {
  font-family: "Kozuka Gothic Pr6N", sans-serif;
  font-weight: 900;
  font-size: clamp(22px, 4.7vw, 28pt);
  color: #000;
  line-height: 1.0;
  margin-bottom: 0;
  padding-bottom: 0;
  text-align: left;
}

.lp-title-carmodel {
  font-family: "Kozuka Gothic Pr6N", sans-serif;
  font-weight: 900;
  font-size: clamp(11px, 2.4vw, 14pt);
  color: #000;
  line-height: 1.0;
  margin-bottom: 0;
  padding-bottom: 0;
  text-align: right;
}

.lp-title-jp {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 600;
  font-size: clamp(10px, 1.5vw, 9pt);
  color: #666666;
  margin-top: 0;
  margin-bottom: 15px;
  text-align: left;
  display: block;
  line-height: 1.4;
}

.lp-catch {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 600;
  font-size: clamp(13px, 2.2vw, 13pt);
  color: #000;
  margin-top: -5px;
  margin-bottom: 15px;
  text-align: left;
  display: block;
  line-height: 1.4;
}

/* =================================================================
   3. 画像設定 (Images)
================================================================= */
.lp-img-wrap {
  display: block;
  position: relative;
  overflow: hidden;
  margin-bottom: 0px;
}

.lp-img-wrap img { width: 100%; height: 100%; object-fit: cover; }

.img-800-400 {
  width: 800px;
  height: 400px;
  margin: 0 auto 40px;
  padding-bottom: 0;
  object-fit: cover;
  display: block;
}

.img-800-200 {
  width: 800px;
  height: 200px;
  margin: 0 auto 5px;
  padding-bottom: 0;
  object-fit: cover;
  display: block;
}

.img-388-240 {
  width: 388px;
  height: 240px;
  margin: 0 auto 10px;
  padding-bottom: 0;
  object-fit: cover;
  display: block;
}

@media screen and (max-width: 768px) {
  .img-800-400 {
    width: 100%;
    height: auto;
    aspect-ratio: 800 / 400;
    margin: 0 auto 40px;
    padding-bottom: 0;
    display: block;
  }
  .img-800-197 {
    width: 100%;
    height: auto;
    aspect-ratio: 800 / 197;
    margin: 0 auto 5px;
    padding-bottom: 0;
    display: block;
  }
  .img-388-240 {
    width: 100%;
    height: auto;
    aspect-ratio: 388 / 240;
    margin: 0 auto 10px;
    padding-bottom: 0;
    display: block;
  }
}

/* =================================================================
   4. 帯見出し (Band Title)
================================================================= */
.lp-band-title {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 300;
  font-size: clamp(14px, 2.5vw, 15pt);
  color: #000;
  border-top: 1px solid #000;
  border-bottom: 1px solid #000;
  padding: 12px 0;
  margin: 60px auto 30px;
  text-align: center;
  width: 800px;
  max-width: 100%;
  box-sizing: border-box;
  line-height: 1.6;
}

/* =================================================================
   5. テキスト・見出し (Typography)
================================================================= */
.text-heading-lg {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 600;
  width: 800px;
  max-width: 100%;
  font-size: clamp(16px, 2.5vw, 17pt);
  color: #000;
  margin: 0 auto 10px;
  line-height: 1.4;
  text-align: left;
}

.text-heading-md {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 600;
  width: 800px;
  max-width: 100%;
  font-size: clamp(15px, 2.5vw, 16pt);
  color: #000;
  margin: 0 auto 10px;
  line-height: 1.4;
  text-align: left;
}

.text-body {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 300;
  width: 800px;
  max-width: 100%;
  font-size: clamp(12px, 2.0vw, 12pt);
  color: #000;
  margin: 0 auto 10px;
  line-height: 1.6;
  text-align: left;
}

/* =================================================================
   6. 2カラムレイアウト (Grid System)
================================================================= */
.lp-grid-2 {
  display: flex;
  justify-content: space-evenly;
  width: 800px;
  max-width: 100%;
  gap: 0;
  margin: 0 auto 30px;
}

.lp-col {
  width: 357px;
  max-width: 48%;
  text-align: left;
}

.lp-col .text-heading-md {
  font-weight: 700;
  font-size: clamp(13px, 2.2vw, 13pt);
}

@media screen and (max-width: 768px) {
  .lp-grid-2 {
    display: block;
  }
  .lp-col {
    width: 100%;
    max-width: 100%;
    margin-bottom: 40px;
  }
  .lp-col:last-child {
    margin-bottom: 0;
  }
}

/* =================================================================
   7. スペック表 (Specification Table)
================================================================= */
.lp-spec-table {
  width: 100%;
  max-width: 800px;
  border-collapse: collapse;
  color: #000;
  margin: 0 auto 30px;
}

.lp-spec-table tr:nth-child(odd) {
  background: #CCC;
}

.lp-spec-table th {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 300;
  font-size: clamp(11px, 2.0vw, 12pt);
  padding: 16px;
  text-align: center;
  width: 30%;
  vertical-align: middle;
}

.lp-spec-table td {
  font-family: "Hiragino Kaku Gothic ProN", "Hiragino Sans", "Yu Gothic", sans-serif;
  font-weight: 300;
  font-size: clamp(11px, 2.0vw, 11pt);
  padding: 16px;
  vertical-align: middle;
  text-align: left;
}

@media screen and (max-width: 768px) {
  .lp-spec-table th {
    padding: 12px;
    width: 28%;
  }
  .lp-spec-table td {
    padding: 12px;
  }
}
```

---

## 付録B: 完成サンプル（組み立て済みHTML）

```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Product LP</title>
<style>
/* 付録AのCSS全文をここに貼り付け */
</style>
</head>
<body>

<!-- ブロック① 商品タイトル -->
<div class="lp-wrapper">
    <div class="lp-header-top" style="display: flex; justify-content: space-between; align-items: flex-end; flex-wrap: wrap;">
        <div class="lp-title-main">AERO GUARD PLUS</div>
        <div class="lp-title-carmodel">TOYOTA GR86 / SUBARU BRZ (ZN8/ZD8)</div>
    </div>
    <div class="lp-title-jp">エアロガード プラス フロントリップスポイラー</div>
    <div class="lp-catch">空力性能と美しさを両立する、次世代エアロパーツ</div>
</div>

<!-- ブロック② メイン画像 -->
<div class="lp-img-wrap img-800-400">
    <img src="https://image.rakuten.co.jp/example/cabinet/main001.jpg" alt="AERO GUARD PLUS メイン画像">
</div>

<!-- ブロック③ 帯見出し -->
<div class="lp-band-title">製品の特長</div>

<!-- ブロック④ 大見出し＋本文 -->
<div class="text-heading-lg">高剛性ウレタン素材を採用</div>
<div class="text-body">
    独自開発の高剛性ウレタンを採用し、軽量ながら優れた耐久性を実現しました。<br>
    走行時の空力負荷にも耐える設計で、長期間にわたり美しいフォルムを維持します。
</div>

<!-- ブロック⑦ 2カラム -->
<div class="lp-grid-2">
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="https://image.rakuten.co.jp/example/cabinet/front001.jpg" alt="フロントビュー">
        </div>
        <div class="text-heading-md">フロントビュー</div>
        <div class="text-body">
            低重心デザインがスポーティな印象を強調します。
        </div>
    </div>
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="https://image.rakuten.co.jp/example/cabinet/side001.jpg" alt="サイドビュー">
        </div>
        <div class="text-heading-md">サイドビュー</div>
        <div class="text-body">
            ボディラインと一体化する流麗なフォルムです。
        </div>
    </div>
</div>

<!-- ブロック③ 帯見出し -->
<div class="lp-band-title">スペック</div>

<!-- ブロック⑥ スペック表 -->
<table class="lp-spec-table">
    <tbody>
        <tr>
            <th>素材</th>
            <td>高剛性ウレタン（PUR）</td>
        </tr>
        <tr>
            <th>カラー</th>
            <td>未塗装（素地）</td>
        </tr>
        <tr>
            <th>適合車種</th>
            <td>TOYOTA GR86 (ZN8) / SUBARU BRZ (ZD8) 2021年〜</td>
        </tr>
    </tbody>
</table>

<!-- ブロック③ 帯見出し -->
<div class="lp-band-title">注意事項</div>

<!-- ブロック⑤ 小見出し＋本文 -->
<div class="text-heading-md">取り付けについて</div>
<div class="text-body">
    本製品の取り付けには専門知識が必要です。<br>
    必ず認定整備工場での取り付けを推奨いたします。
</div>

</body>
</html>
```

---

## 付録C: クイックリファレンスカード

| やりたいこと | 使うブロック | クラス名 |
|---|---|---|
| 商品名を入れたい | ブロック① | `lp-title-main`, `lp-title-carmodel`, `lp-title-jp`, `lp-catch` |
| 大きい画像を入れたい | ブロック② | `lp-img-wrap img-800-400` |
| 横長バナーを入れたい | ブロック② | `lp-img-wrap img-800-200` |
| セクション区切りを入れたい | ブロック③ | `lp-band-title` |
| 大きな見出しで説明したい | ブロック④ | `text-heading-lg` + `text-body` |
| 小さな見出しで説明したい | ブロック⑤ | `text-heading-md` + `text-body` |
| 仕様表を入れたい | ブロック⑥ | `lp-spec-table` |
| 2列で比較・並列表示したい | ブロック⑦ | `lp-grid-2` > `lp-col` |
