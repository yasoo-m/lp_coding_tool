# 楽天用テキスト化ランディングページ作成ツール 仕様書

**バージョン:** 1.0  
**作成日:** 2026年3月27日  
**最終更新:** 2026年3月27日

---

## 1. ツール概要

### 1.1 目的

楽天市場の商品ページ（PC用販売説明文 / スマートフォン用商品説明文）に掲載するテキスト化ランディングページ（LP）を、テンプレートに沿ってテキスト入力するだけで HTML+CSS を自動生成するツール。

### 1.2 対象プラットフォーム

| 出力先 | 入力欄 | HTMLレベル | 文字数上限 |
|--------|--------|------------|------------|
| 楽天 PC | PC用販売説明文 | レベル2 | 全角5,120文字 |
| 楽天 スマホ | スマートフォン用商品説明文 | 制限タグのみ | 全角5,120文字 |

### 1.3 基本方針

- ベースCSSはプロジェクト内の `CSSテンプレート` を使用する（`<style>` タグで埋め込み）
- コンテンツ部分は **6種類のブロックパーツ** を組み合わせて構築する
- 画像は外部URL（楽天キャビネット / Yahoo!ショッピング トリプル）を指定する
- PC版はCSS埋め込みでリッチなレイアウトを実現、スマホ版は許可タグのみで構成する

---

## 2. 楽天HTMLタグ制限

### 2.1 PC用販売説明文（HTMLレベル2）

**使用可能なタグ（主要なもの）：**

| タグ | 用途 | 属性制限 |
|------|------|----------|
| `<div>` | ブロック要素 | 一部属性制限あり |
| `<span>` | インライン要素 | 一部属性制限あり |
| `<table>` `<tr>` `<th>` `<td>` | テーブル | 一部属性制限あり |
| `<img>` | 画像表示 | 一部属性制限あり |
| `<br>` | 改行 | 一部属性制限あり |
| `<p>` | 段落 | 一部属性制限あり |
| `<font>` | フォント指定 | 一部属性制限あり |
| `<b>` | 太字 | — |
| `<style>` | CSS記述 | 外部CSS読み込み不可 |
| `<a>` | リンク | 一部属性制限あり |

**使用不可なタグ（主要なもの）：**
`<script>`（動画貼付以外）、`<form>`、`<input>`、`<iframe>`（一部制限付き）、`<object>`、`<embed>`、`<!DOCTYPE>`、`<body>`、`<head>` 等

**重要な制約：**
- `<style>` タグは使用可能だが、**外部CSSファイルの読み込み（`<link rel="stylesheet">`）は不可**
- CSSはすべて `<style>` タグ内にインラインで記述する

### 2.2 スマートフォン用商品説明文

**使用可能なタグ（ホワイトリスト方式・これ以外は不可）：**

| タグ | 用途 | 使用可能な属性 |
|------|------|----------------|
| `<a>` | リンク | href, target |
| `<img>` | 画像表示（**上限20枚**） | alt, border, height, src, width |
| `<table>` | テーブル | align（※left/right不可）, bgcolor, border, bordercolor, cellpadding, cellspacing, frame, height, rules, width |
| `<tr>` | テーブル行 | align, bgcolor, bordercolor, height, valign |
| `<th>` | 見出しセル | align, axis, bgcolor, bordercolor, colspan, headers, height, rowspan, valign, width |
| `<td>` | データセル | align, axis, bgcolor, bordercolor, colspan, headers, height, rowspan, valign, width |
| `<br>` | 改行 | — |
| `<p>` | 段落 | align |
| `<font>` | フォント | color, size |
| `<b>` | 太字 | — |
| `<center>` | 中央揃え | — |
| `<hr>` | 水平線 | color, noshade, size |
| `<!-- -->` | コメントアウト | — |

**重要な制約：**
- `<style>` タグは**使用不可** → CSSによるスタイリングは不可
- `<div>` `<span>` は**使用不可** → テーブルレイアウトで代替する
- 画像は **1ページあたり20枚まで**、1枚あたり **200KB程度推奨**
- `<table align="left">` / `<table align="right">` は画面崩れの原因となるため使用禁止

---

## 3. 画像管理

### 3.1 画像ホスティング

| プラットフォーム | サービス | URL形式 |
|------------------|----------|---------|
| 楽天市場 | R-Cabinet（楽天キャビネット） | `https://image.rakuten.co.jp/ショップID/cabinet/フォルダ名/ファイル名` |
| Yahoo!ショッピング | トリプル | `https://shopping.c.yimg.jp/lib/ストアID/ファイル名` |

### 3.2 画像サイズ規格

| 用途 | CSSクラス | PCサイズ(px) | アスペクト比 |
|------|-----------|-------------|-------------|
| メイン画像 | `img-800-400` | 800 × 400 | 2:1 |
| 横長画像（取付イメージ等） | `img-800-200` | 800 × 200 | 4:1 |
| 2カラム用画像 | `img-388-240` | 388 × 240 | 約1.6:1 |

### 3.3 画像入力方式

ユーザーは画像URLを直接入力する。ツールはURLを受け取り、`<img src="URL">` として HTML に埋め込む。

---

## 4. ブロックパーツ構成

LPは以下の **6種類のブロックパーツ** を自由に組み合わせて構築する。

### 4.1 商品タイトルブロック

**役割：** ページ最上部に配置する商品名・適合車種・キャッチコピーのヘッダー領域

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| 商品名（英語） | 英字メインのメインタイトル | ○ |
| 適合車種 | 対応車種名 | — |
| 商品名（日本語） | 日本語の商品名 | ○ |
| キャッチコピー | 訴求テキスト | ○ |

**PC版 HTMLテンプレート：**

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

**スマホ版 HTMLテンプレート：**

```html
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr>
        <td>
            <font size="5"><b>{{商品名_英語}}</b></font>
        </td>
    </tr>
    <tr>
        <td align="right">
            <font size="2"><b>{{適合車種}}</b></font>
        </td>
    </tr>
    <tr>
        <td>
            <font size="1" color="#666666">{{商品名_日本語}}</font>
        </td>
    </tr>
    <tr>
        <td>
            <font size="3"><b>{{キャッチコピー}}</b></font>
        </td>
    </tr>
</table>
```

---

### 4.2 画像ブロック

**役割：** フル幅の画像を1枚表示する

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| 画像URL | 楽天キャビネットまたはトリプルのURL | ○ |
| 画像サイズ | メイン(800×400) / 横長(800×200) から選択 | ○ |
| alt属性 | 画像の説明テキスト | ○ |

**PC版 HTMLテンプレート：**

```html
<!-- メイン画像(800x400)の場合 -->
<div class="lp-img-wrap img-800-400">
    <img src="{{画像URL}}" alt="{{alt属性}}">
</div>

<!-- 横長画像(800x200)の場合 -->
<div class="lp-img-wrap img-800-200">
    <img src="{{画像URL}}" alt="{{alt属性}}">
</div>
```

**スマホ版 HTMLテンプレート：**

```html
<img src="{{画像URL}}" alt="{{alt属性}}" width="100%" border="0">
<br>
```

---

### 4.3 帯見出しブロック

**役割：** セクション区切りとなる帯状の見出し（上下に罫線）

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| 帯テキスト | セクション名（例：「製品の特長」「スペック」） | ○ |

**PC版 HTMLテンプレート：**

```html
<div class="lp-band-title">{{帯テキスト}}</div>
```

**スマホ版 HTMLテンプレート：**

```html
<hr size="1" color="#000000" noshade>
<center><font size="3">{{帯テキスト}}</font></center>
<hr size="1" color="#000000" noshade>
<br>
```

---

### 4.4 見出し＋本文ブロック

**役割：** 大見出しまたは中見出しと、本文テキストの組み合わせ

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| 見出しサイズ | 大(lg) / 中(md) から選択 | ○ |
| 見出しテキスト | 見出し文字列 | ○ |
| 本文テキスト | 説明文（複数行可） | ○ |

**PC版 HTMLテンプレート：**

```html
<!-- 大見出しの場合 -->
<div class="text-heading-lg">{{見出しテキスト}}</div>
<div class="text-body">
    {{本文テキスト}}
</div>

<!-- 中見出しの場合 -->
<div class="text-heading-md">{{見出しテキスト}}</div>
<div class="text-body">
    {{本文テキスト}}
</div>
```

**スマホ版 HTMLテンプレート：**

```html
<!-- 大見出しの場合 -->
<font size="4"><b>{{見出しテキスト}}</b></font>
<br>
<font size="2">{{本文テキスト}}</font>
<br><br>

<!-- 中見出しの場合 -->
<font size="3"><b>{{見出しテキスト}}</b></font>
<br>
<font size="2">{{本文テキスト}}</font>
<br><br>
```

---

### 4.5 2カラムブロック

**役割：** 画像＋見出し＋本文を横2列に並べるレイアウト

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| 左カラム画像URL | 画像URL | ○ |
| 左カラムalt | 画像の説明 | ○ |
| 左カラム見出し | 中見出しテキスト | ○ |
| 左カラム本文 | 説明文 | ○ |
| 右カラム画像URL | 画像URL | ○ |
| 右カラムalt | 画像の説明 | ○ |
| 右カラム見出し | 中見出しテキスト | ○ |
| 右カラム本文 | 説明文 | ○ |

**PC版 HTMLテンプレート：**

```html
<div class="lp-grid-2">
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="{{左カラム画像URL}}" alt="{{左カラムalt}}">
        </div>
        <div class="text-heading-md">{{左カラム見出し}}</div>
        <div class="text-body">{{左カラム本文}}</div>
    </div>
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="{{右カラム画像URL}}" alt="{{右カラムalt}}">
        </div>
        <div class="text-heading-md">{{右カラム見出し}}</div>
        <div class="text-body">{{右カラム本文}}</div>
    </div>
</div>
```

**スマホ版 HTMLテンプレート：**

```html
<!-- スマホでは縦積みで表示 -->
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr>
        <td>
            <img src="{{左カラム画像URL}}" alt="{{左カラムalt}}" width="100%" border="0">
            <br>
            <font size="3"><b>{{左カラム見出し}}</b></font>
            <br>
            <font size="2">{{左カラム本文}}</font>
        </td>
    </tr>
</table>
<br>
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr>
        <td>
            <img src="{{右カラム画像URL}}" alt="{{右カラムalt}}" width="100%" border="0">
            <br>
            <font size="3"><b>{{右カラム見出し}}</b></font>
            <br>
            <font size="2">{{右カラム本文}}</font>
        </td>
    </tr>
</table>
<br>
```

---

### 4.6 スペック表ブロック

**役割：** 商品仕様をストライプ表で表示

**入力項目：**

| 項目 | 入力内容 | 必須 |
|------|----------|------|
| スペック行（複数） | 項目名（左セル） + 値（右セル） のペアを複数行 | ○ |

**PC版 HTMLテンプレート：**

```html
<table class="lp-spec-table">
    <tbody>
        <tr>
            <th>{{項目名1}}</th>
            <td>{{値1}}</td>
        </tr>
        <tr>
            <th>{{項目名2}}</th>
            <td>{{値2}}</td>
        </tr>
        <!-- 行を必要数繰り返す -->
    </tbody>
</table>
```

**スマホ版 HTMLテンプレート：**

```html
<table width="100%" cellpadding="8" cellspacing="0" border="0">
    <tr bgcolor="#CCCCCC">
        <th width="30%" align="center"><font size="2">{{項目名1}}</font></th>
        <td><font size="2">{{値1}}</font></td>
    </tr>
    <tr>
        <th width="30%" align="center"><font size="2">{{項目名2}}</font></th>
        <td><font size="2">{{値2}}</font></td>
    </tr>
    <!-- 奇数行にbgcolor="#CCCCCC"を付与してストライプにする -->
</table>
```

---

## 5. LP構成フレームワーク（推奨構成順序）

標準的なLPは以下の順序でブロックを配置する。必要に応じてブロックの追加・省略・順序変更が可能。

```
┌─────────────────────────────────────┐
│  ① 商品タイトルブロック              │
├─────────────────────────────────────┤
│  ② 画像ブロック（メイン画像）        │
├─────────────────────────────────────┤
│  ③ 帯見出し「製品の特長」            │
├─────────────────────────────────────┤
│  ④ 見出し＋本文ブロック（特長説明）  │
├─────────────────────────────────────┤
│  ⑤ 2カラムブロック（特長詳細）       │
├─────────────────────────────────────┤
│  ⑥ 画像ブロック（取付イメージ等）    │
├─────────────────────────────────────┤
│  ⑦ 帯見出し「スペック」              │
├─────────────────────────────────────┤
│  ⑧ スペック表ブロック                │
├─────────────────────────────────────┤
│  ⑨ 帯見出し「注意事項」              │
├─────────────────────────────────────┤
│  ⑩ 見出し＋本文ブロック（注意事項）  │
└─────────────────────────────────────┘
```

---

## 6. 出力仕様

### 6.1 PC版出力

PC用販売説明文として、`<style>` タグ（CSS全文）＋ HTML本文 を**1つの文字列**として出力する。

**出力構造：**

```html
<style>
/* ===== CSSテンプレート全文をここに埋め込む ===== */
.lp-wrapper { ... }
.lp-title-main { ... }
/* ... 以下、ベースCSS全体 ... */
</style>

<div class="lp-wrapper">
    <!-- ブロックパーツをここに順番に配置 -->
</div>
```

**制約チェック項目：**
- 全角5,120文字以内であること
- HTMLレベル2で禁止されているタグを使用していないこと
- `<style>` タグ内で外部CSSファイルを `@import` や `<link>` で読み込んでいないこと

### 6.2 スマホ版出力

スマートフォン用商品説明文として、許可タグのみで構成した HTMLを出力する。

**制約チェック項目：**
- 全角5,120文字以内であること
- 使用タグがホワイトリスト（a, img, table, tr, th, td, br, p, font, b, center, hr）に含まれていること
- `<img>` タグの使用数が **20個以下** であること
- `<div>`, `<span>`, `<style>` を使用していないこと
- `<table align="left">` / `<table align="right">` を使用していないこと

---

## 7. ツール入出力フロー

### 7.1 入力フロー

```
ユーザー入力
    │
    ├── 1. 全体情報入力
    │       ├── 商品名（英語）
    │       ├── 商品名（日本語）
    │       ├── 適合車種
    │       └── キャッチコピー
    │
    ├── 2. ブロック構成を選択
    │       └── 使用するブロックの種類・順序を指定
    │
    ├── 3. 各ブロックの内容入力
    │       ├── テキスト入力（見出し、本文、スペック等）
    │       └── 画像URL入力
    │
    └── 4. 生成実行
```

### 7.2 出力フロー

```
生成処理
    │
    ├── テンプレートにテキストをバインド
    │
    ├── PC版HTML生成
    │       ├── CSSテンプレート埋め込み
    │       ├── 各ブロックHTML結合
    │       └── 文字数チェック（5,120文字以内）
    │
    ├── スマホ版HTML生成
    │       ├── 許可タグのみでテンプレート変換
    │       ├── 各ブロックHTML結合
    │       ├── 画像数チェック（20枚以内）
    │       └── 文字数チェック（5,120文字以内）
    │
    └── 出力
            ├── PC版HTML（コピー用テキスト）
            └── スマホ版HTML（コピー用テキスト）
```

---

## 8. CSSベース設計

### 8.1 デザイン基本仕様

| 項目 | 値 |
|------|-----|
| 最大幅 | 800px |
| 背景色 | #FFFFFF |
| 文字色（基本） | #000000 |
| 日本語タイトル色 | #666666 |
| フォント（英語タイトル） | Kozuka Gothic Pr6N, sans-serif |
| フォント（日本語） | Hiragino Kaku Gothic ProN, Hiragino Sans, Yu Gothic, sans-serif |
| スペック表ストライプ色 | #CCCCCC |
| レスポンシブ切替 | 768px以下 |

### 8.2 CSS全文

プロジェクト内の `CSSテンプレート` に記載された `<style>` 内の全CSSを使用する。ツールはこのCSSを変更せず、そのまま出力HTMLの先頭に埋め込む。

---

## 9. 運用上の注意事項

### 9.1 文字数管理

PC版・スマホ版ともに全角5,120文字の上限がある。CSS部分もこの文字数に含まれるため、PC版ではCSS（約4,500文字）を含めた残り文字数（約600文字分のHTML）で構成する必要がある。コンテンツ量が多い場合は以下の対策を検討する。

- CSSのコメントを除去して圧縮する（minify）
- 不要なCSS定義を削除する（使わないブロックのスタイル除去）
- テキスト量を削減する

### 9.2 画像に関する注意

- 楽天キャビネットの画像URLは `https://image.rakuten.co.jp/` で始まること
- Yahoo!トリプルの画像URLは `https://shopping.c.yimg.jp/lib/` で始まること
- スマホ版では画像20枚制限を厳守する
- 画像1枚あたり200KB以内を推奨（ページ読込速度に影響）

### 9.3 クロスプラットフォーム対応

同じ商品情報から PC版とスマホ版の2種類のHTMLを同時に生成する。コンテンツ（テキスト・画像URL）は共通だが、HTML構造はプラットフォームごとに完全に異なる。

---

## 10. 将来の拡張案

| 拡張項目 | 内容 |
|----------|------|
| プレビュー機能 | 生成したHTMLをブラウザ上でリアルタイムプレビュー |
| CSS minify | 自動でCSSを圧縮し文字数を節約 |
| 文字数カウンター | リアルタイムで残り文字数を表示 |
| 画像枚数カウンター | スマホ版の画像使用枚数をリアルタイム表示 |
| テンプレート保存 | 商品カテゴリごとにブロック構成パターンを保存 |
| バリデーション | 禁止タグの使用を自動検出してエラー表示 |
| Yahoo!ショッピング対応 | Yahoo!ストアクリエイターPro向け出力にも対応 |

---

## 付録A：ブロック入力テンプレート（テキスト記入シート）

ツールへの入力は以下の形式で行う。

```
===== LP構成シート =====

【商品タイトル】
商品名（英語）: 
適合車種: 
商品名（日本語）: 
キャッチコピー: 

---

【ブロック1: 画像】
画像URL: 
サイズ: メイン / 横長
alt: 

---

【ブロック2: 帯見出し】
テキスト: 

---

【ブロック3: 見出し＋本文】
見出しサイズ: 大 / 中
見出し: 
本文: 

---

【ブロック4: 2カラム】
左画像URL: 
左alt: 
左見出し: 
左本文: 
右画像URL: 
右alt: 
右見出し: 
右本文: 

---

【ブロック5: スペック表】
項目1: 値1
項目2: 値2
項目3: 値3

---

【ブロック6: 帯見出し】
テキスト: 

---

【ブロック7: 見出し＋本文】
見出しサイズ: 大 / 中
見出し: 
本文: 

===== ここまで =====
```

---

## 付録B：出力サンプル（PC版）

```html
<style>
.lp-wrapper{width:100%;max-width:800px;margin:0 auto;padding:0;border:none;background-color:#fff;text-align:left;overflow-x:hidden;box-sizing:border-box}
/* ... CSS省略（ベースCSS全文を挿入） ... */
</style>

<div class="lp-wrapper">
    <div class="lp-header-top" style="display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;">
        <div class="lp-title-main">PREMIUM FLOOR MAT</div>
        <div class="lp-title-carmodel">TOYOTA ALPHARD 40系</div>
    </div>
    <div class="lp-title-jp">プレミアムフロアマット</div>
    <div class="lp-catch">最高級素材を使用した、40系アルファード専用設計のフロアマット</div>
</div>

<div class="lp-img-wrap img-800-400">
    <img src="https://image.rakuten.co.jp/shopid/cabinet/mat/main.jpg" alt="プレミアムフロアマット メイン画像">
</div>

<div class="lp-band-title">製品の特長</div>

<div class="text-heading-lg">車種専用設計で完璧なフィッティング</div>
<div class="text-body">
    40系アルファード/ヴェルファイアのフロア形状を3Dスキャンし、ミリ単位で型取りした専用設計。純正マットのように、ズレや浮きのないフィット感を実現します。
</div>

<div class="lp-grid-2">
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="https://image.rakuten.co.jp/shopid/cabinet/mat/feature01.jpg" alt="高級素材">
        </div>
        <div class="text-heading-md">高級素材を採用</div>
        <div class="text-body">厳選された国産素材を使用。</div>
    </div>
    <div class="lp-col">
        <div class="lp-img-wrap img-388-240">
            <img src="https://image.rakuten.co.jp/shopid/cabinet/mat/feature02.jpg" alt="滑り止め加工">
        </div>
        <div class="text-heading-md">裏面滑り止め加工</div>
        <div class="text-body">特殊ゴム加工で安全性を確保。</div>
    </div>
</div>

<div class="lp-band-title">スペック</div>

<table class="lp-spec-table">
    <tbody>
        <tr><th>素材</th><td>ナイロン100%（表面）/ 合成ゴム（裏面）</td></tr>
        <tr><th>適合車種</th><td>トヨタ アルファード/ヴェルファイア 40系</td></tr>
        <tr><th>セット内容</th><td>フロント2枚 + リア3枚（計5枚）</td></tr>
        <tr><th>カラー</th><td>ブラック / ベージュ / グレー</td></tr>
    </tbody>
</table>
```

---

## 付録C：出力サンプル（スマホ版）

```html
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr><td><font size="5"><b>PREMIUM FLOOR MAT</b></font></td></tr>
    <tr><td align="right"><font size="2"><b>TOYOTA ALPHARD 40系</b></font></td></tr>
    <tr><td><font size="1" color="#666666">プレミアムフロアマット</font></td></tr>
    <tr><td><font size="3"><b>最高級素材を使用した、40系アルファード専用設計のフロアマット</b></font></td></tr>
</table>
<br>
<img src="https://image.rakuten.co.jp/shopid/cabinet/mat/main.jpg" alt="プレミアムフロアマット メイン画像" width="100%" border="0">
<br>
<hr size="1" color="#000000" noshade>
<center><font size="3">製品の特長</font></center>
<hr size="1" color="#000000" noshade>
<br>
<font size="4"><b>車種専用設計で完璧なフィッティング</b></font>
<br>
<font size="2">40系アルファード/ヴェルファイアのフロア形状を3Dスキャンし、ミリ単位で型取りした専用設計。純正マットのように、ズレや浮きのないフィット感を実現します。</font>
<br><br>
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr><td>
        <img src="https://image.rakuten.co.jp/shopid/cabinet/mat/feature01.jpg" alt="高級素材" width="100%" border="0">
        <br><font size="3"><b>高級素材を採用</b></font>
        <br><font size="2">厳選された国産素材を使用。</font>
    </td></tr>
</table>
<br>
<table width="100%" cellpadding="0" cellspacing="0" border="0">
    <tr><td>
        <img src="https://image.rakuten.co.jp/shopid/cabinet/mat/feature02.jpg" alt="滑り止め加工" width="100%" border="0">
        <br><font size="3"><b>裏面滑り止め加工</b></font>
        <br><font size="2">特殊ゴム加工で安全性を確保。</font>
    </td></tr>
</table>
<br>
<hr size="1" color="#000000" noshade>
<center><font size="3">スペック</font></center>
<hr size="1" color="#000000" noshade>
<br>
<table width="100%" cellpadding="8" cellspacing="0" border="0">
    <tr bgcolor="#CCCCCC">
        <th width="30%" align="center"><font size="2">素材</font></th>
        <td><font size="2">ナイロン100%（表面）/ 合成ゴム（裏面）</font></td>
    </tr>
    <tr>
        <th width="30%" align="center"><font size="2">適合車種</font></th>
        <td><font size="2">トヨタ アルファード/ヴェルファイア 40系</font></td>
    </tr>
    <tr bgcolor="#CCCCCC">
        <th width="30%" align="center"><font size="2">セット内容</font></th>
        <td><font size="2">フロント2枚 + リア3枚（計5枚）</font></td>
    </tr>
    <tr>
        <th width="30%" align="center"><font size="2">カラー</font></th>
        <td><font size="2">ブラック / ベージュ / グレー</font></td>
    </tr>
</table>
```
