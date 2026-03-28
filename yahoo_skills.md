# ヤフーショッピング LP作成ツール 仕様書

## 1. 概要

本ツールは、ヤフーショッピングの「PC用フリースペース1」欄に掲載するテキスト化ランディングページ（LP）を、テンプレート入力方式で簡易に作成するためのフレームワークである。

### 1.1 目的

- テキスト・画像URLを入力するだけで、HTML+CSSが自動生成される仕組みを構築する
- デザインの統一性を保ちつつ、商品ごとにコンテンツを差し替え可能にする
- ヤフーショッピングのHTMLタグ制限に準拠した安全なコードを出力する

### 1.2 掲載先の制約

| 項目 | 仕様 |
|------|------|
| 掲載先 | ストアクリエイターPro → PC用フリースペース1 |
| 文字数上限 | 全角15,000文字（30,000バイト）以内 |
| HTML仕様 | HTML4.01準拠 |
| CSSの扱い | `<style>` タグは使用不可。インラインCSS（`style=""` 属性）で記述する |

> **重要**: ヤフーショッピングでは `<style>` タグや `<script>` タグが使用できないため、すべてのスタイルは各要素の `style=""` 属性にインラインで記述する必要がある。

---

## 2. 使用可能HTMLタグ一覧（PC用）

ヤフーショッピング公式ドキュメントに基づく、PC用ストアで使用可能なHTMLタグは以下の通り。

### 2.1 本ツールで使用するタグ

| タグ | 用途 |
|------|------|
| `<div>` | レイアウトブロック、各コンポーネントのラッパー |
| `<img>` | 商品画像の表示 |
| `<table>` `<tbody>` `<tr>` `<th>` `<td>` | スペック表 |
| `<span>` | インラインテキスト装飾 |
| `<br>` | 改行 |
| `<b>` / `<strong>` | 太字 |
| `<a>` | リンク（href は `https` / `//` / `mailto` / `#` のみ） |
| `<p>` | 段落 |

### 2.2 使用不可の主なタグ

| タグ | 理由 |
|------|------|
| `<style>` | 一覧に含まれないため使用不可 |
| `<script>` | 一覧に含まれないため使用不可 |
| `<form>` `<input>` `<button>` | フォーム系は一覧にないため使用不可 |
| `<h1>` `<h2>` `<h3>` | PC版では `<h4>` `<h5>` `<h6>` のみ許可 |
| `<section>` `<article>` `<header>` `<footer>` | HTML5タグはPC版では使用不可 |

### 2.3 `<a>` タグの制限

href に指定可能な文字列は以下のいずれかで始まるもののみ。それ以外は空文字に補正される。

- `https`
- `//`
- `mailto`
- `#`（ページ内リンク）

---

## 3. CSSの取り扱い

### 3.1 ベースCSS（デザイン定義）

プロジェクト内の「CSSテンプレート」に定義された全スタイルを、LP構築のデザイン基準として使用する。ただし、ヤフーショッピングでは `<style>` タグが使えないため、**出力時にはすべてインラインCSS化する**。

### 3.2 インラインCSS変換ルール

ベースCSSの各クラスに対応するスタイルを、HTML出力時に `style=""` 属性として展開する。

**変換例:**

ベースCSS定義:
```css
.lp-title-main {
  font-family: "Kozuka Gothic Pr6N", sans-serif;
  font-weight: 900;
  font-size: 28pt;
  color: #000;
  line-height: 1.0;
  text-align: left;
}
```

出力HTML:
```html
<div style="font-family:'Kozuka Gothic Pr6N',sans-serif; font-weight:900; font-size:28pt; color:#000; line-height:1.0; text-align:left;">
  PRODUCT NAME
</div>
```

### 3.3 レスポンシブ対応について

- `@media` クエリはインラインCSSでは使用できない
- PC用フリースペースのため、**PC表示（幅800px）を基準**とした固定レイアウトで出力する
- `clamp()` 関数はインラインでも使用可能だが、対応ブラウザに注意が必要なため、**固定値で出力する**（ベースCSS内の `clamp()` の最大値を採用）

---

## 4. コンポーネント一覧

LPは以下のコンポーネントを組み合わせて構成する。各コンポーネントは独立したブロックとして上から順に配置される。

### 4.1 商品タイトルブロック

ページ最上部に表示する商品情報エリア。

| 入力項目 | 説明 | 必須 |
|----------|------|------|
| 商品名（英語） | メインタイトル。太字大文字 | ○ |
| 適合車種 | 右寄せで表示 | △ |
| 商品名（日本語） | 小さめのグレー文字 | △ |
| キャッチコピー | 訴求テキスト | ○ |

**出力HTML構造:**
```html
<div style="width:100%; max-width:800px; margin:0 auto; padding:0; background-color:#fff; text-align:left; box-sizing:border-box;">
  <div style="display:flex; justify-content:space-between; align-items:flex-end; flex-wrap:wrap;">
    <div style="font-weight:900; font-size:28pt; color:#000; line-height:1.0; text-align:left;">
      {{商品名_英語}}
    </div>
    <div style="font-weight:900; font-size:14pt; color:#000; line-height:1.0; text-align:right;">
      {{適合車種}}
    </div>
  </div>
  <div style="font-weight:600; font-size:9pt; color:#666; margin-top:0; margin-bottom:15px; text-align:left; line-height:1.4;">
    {{商品名_日本語}}
  </div>
  <div style="font-weight:600; font-size:13pt; color:#000; margin-top:-5px; margin-bottom:15px; text-align:left; line-height:1.4;">
    {{キャッチコピー}}
  </div>
</div>
```

### 4.2 画像ブロック

3種類のサイズをサポートする。

| サイズ名 | 幅 x 高さ | 用途 |
|----------|-----------|------|
| メイン画像 | 800 x 400px | メインビジュアル、セクション画像 |
| 横長画像 | 800 x 200px | 取り付けイメージなど |
| 2カラム用画像 | 388 x 240px | 2カラムレイアウト内 |

| 入力項目 | 説明 | 必須 |
|----------|------|------|
| 画像URL | 楽天キャビネットまたはヤフートリプルのURL | ○ |
| alt テキスト | 画像の説明 | ○ |
| サイズ種別 | 上記3種から選択 | ○ |

**出力HTML構造（メイン画像の例）:**
```html
<div style="display:block; position:relative; overflow:hidden; width:800px; height:400px; margin:0 auto 40px; object-fit:cover;">
  <img src="{{画像URL}}" alt="{{alt}}" style="width:100%; height:100%; object-fit:cover;">
</div>
```

### 4.3 帯見出しブロック

セクション区切りに使用する全幅の帯タイトル。上下に線あり。

| 入力項目 | 説明 | 必須 |
|----------|------|------|
| 帯テキスト | セクションタイトル（例: 「製品の特長」） | ○ |

**出力HTML構造:**
```html
<div style="font-weight:300; font-size:15pt; color:#000; border-top:1px solid #000; border-bottom:1px solid #000; padding:12px 0; margin:60px auto 30px; text-align:center; width:800px; max-width:100%; box-sizing:border-box; line-height:1.6;">
  {{帯テキスト}}
</div>
```

### 4.4 見出し＋本文ブロック

見出しと説明文のセット。

| 入力項目 | 説明 | 必須 |
|----------|------|------|
| 見出しテキスト | セクション見出し | ○ |
| 本文テキスト | 説明文（改行は `<br>` で処理） | ○ |

**出力HTML構造:**
```html
<div style="font-weight:600; font-size:17pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.4; text-align:left;">
  {{見出し}}
</div>
<div style="font-weight:300; font-size:12pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.6; text-align:left;">
  {{本文}}
</div>
```

### 4.5 2カラムレイアウトブロック

画像＋見出し＋本文を左右2列で表示する。

| 入力項目（左カラム） | 説明 | 必須 |
|----------------------|------|------|
| 画像URL | 388x240px 画像 | ○ |
| alt テキスト | 画像の説明 | ○ |
| 見出し | カラム見出し | ○ |
| 本文 | カラム説明文 | ○ |

※右カラムも同様の入力項目。

**出力HTML構造:**
```html
<div style="display:flex; justify-content:space-evenly; width:800px; max-width:100%; gap:0; margin:0 auto 30px;">
  <!-- 左カラム -->
  <div style="width:357px; max-width:48%; text-align:left;">
    <div style="display:block; position:relative; overflow:hidden; width:388px; height:240px; margin:0 auto 10px; object-fit:cover;">
      <img src="{{左_画像URL}}" alt="{{左_alt}}" style="width:100%; height:100%; object-fit:cover;">
    </div>
    <div style="font-weight:700; font-size:13pt; color:#000; margin:0 auto 10px; line-height:1.4; text-align:left;">
      {{左_見出し}}
    </div>
    <div style="font-weight:300; font-size:12pt; color:#000; margin:0 auto 10px; line-height:1.6; text-align:left;">
      {{左_本文}}
    </div>
  </div>
  <!-- 右カラム -->
  <div style="width:357px; max-width:48%; text-align:left;">
    <div style="display:block; position:relative; overflow:hidden; width:388px; height:240px; margin:0 auto 10px; object-fit:cover;">
      <img src="{{右_画像URL}}" alt="{{右_alt}}" style="width:100%; height:100%; object-fit:cover;">
    </div>
    <div style="font-weight:700; font-size:13pt; color:#000; margin:0 auto 10px; line-height:1.4; text-align:left;">
      {{右_見出し}}
    </div>
    <div style="font-weight:300; font-size:12pt; color:#000; margin:0 auto 10px; line-height:1.6; text-align:left;">
      {{右_本文}}
    </div>
  </div>
</div>
```

### 4.6 スペック表ブロック

商品仕様を表形式で表示。

| 入力項目 | 説明 | 必須 |
|----------|------|------|
| 項目名（左列） | スペックの項目名 | ○ |
| 値（右列） | スペックの値 | ○ |

※行数は可変。必要な分だけ入力する。

**出力HTML構造:**
```html
<table style="width:100%; max-width:800px; border-collapse:collapse; color:#000; margin:0 auto 30px;">
  <tbody>
    <tr style="background:#CCC;">
      <th style="font-weight:300; font-size:12pt; padding:16px; text-align:center; width:30%; vertical-align:middle;">
        {{項目名1}}
      </th>
      <td style="font-weight:300; font-size:11pt; padding:16px; vertical-align:middle; text-align:left;">
        {{値1}}
      </td>
    </tr>
    <tr>
      <th style="font-weight:300; font-size:12pt; padding:16px; text-align:center; width:30%; vertical-align:middle;">
        {{項目名2}}
      </th>
      <td style="font-weight:300; font-size:11pt; padding:16px; vertical-align:middle; text-align:left;">
        {{値2}}
      </td>
    </tr>
    <!-- 奇数行(1,3,5...)に background:#CCC を付与 -->
  </tbody>
</table>
```

---

## 5. 画像の取り扱い

### 5.1 画像ホスティング先

| サービス | URL形式 | 用途 |
|----------|---------|------|
| 楽天キャビネット | `https://image.rakuten.co.jp/ストアID/cabinet/フォルダ/ファイル名` | 楽天と共用の画像 |
| ヤフートリプル | `https://shopping.geocities.jp/ストアID/フォルダ/ファイル名` | ヤフー専用画像 |

### 5.2 画像仕様

| 項目 | 推奨仕様 |
|------|----------|
| 形式 | JPEG / PNG |
| メイン画像 | 800 x 400px |
| 横長画像 | 800 x 200px |
| 2カラム用画像 | 388 x 240px |

### 5.3 入力方法

ユーザーは画像URLを直接入力する。ツール側ではURLの妥当性チェック（`https://` で始まるか）を行う。

---

## 6. LP構成テンプレート（標準パターン）

一般的な商品LPは以下の順序でコンポーネントを配置する。

```
┌──────────────────────────────────────────┐
│  [1] 商品タイトルブロック                    │
│      商品名（英語）/ 適合車種 / 日本語名 / コピー  │
├──────────────────────────────────────────┤
│  [2] メイン画像 (800x400)                  │
├──────────────────────────────────────────┤
│  [3] 帯見出し「製品の特長」                   │
├──────────────────────────────────────────┤
│  [4] 見出し＋本文（特長説明①）               │
├──────────────────────────────────────────┤
│  [5] 画像 (800x400)                       │
├──────────────────────────────────────────┤
│  [6] 2カラム（特長説明② ＋ ③）              │
├──────────────────────────────────────────┤
│  [7] 帯見出し「スペック」                    │
├──────────────────────────────────────────┤
│  [8] スペック表                            │
├──────────────────────────────────────────┤
│  [9] 帯見出し「注意事項」                    │
├──────────────────────────────────────────┤
│  [10] 見出し＋本文（注意事項説明）             │
└──────────────────────────────────────────┘
```

このテンプレートは標準パターンであり、コンポーネントの追加・削除・順序変更は自由に行える。

---

## 7. 入力フォーマット

### 7.1 入力データ形式

ユーザーはテキストベースで以下の情報を入力する。

```
■ 商品タイトル
商品名（英語）: CARBON FIBER HOOD
適合車種: for TOYOTA GR86 / SUBARU BRZ (ZN8/ZD8)
商品名（日本語）: カーボンファイバーフード
キャッチコピー: 軽量化と空力性能を両立するドライカーボン製ボンネット

■ メイン画像
URL: https://shopping.geocities.jp/example/img/main.jpg
alt: カーボンファイバーフード メイン画像

■ セクション: 製品の特長
--- 見出し+本文 ---
見出し: 圧倒的な軽量化を実現
本文: 純正スチールフード比で約60%の軽量化を実現。ドライカーボン（CFRP）製法により、高い剛性を維持しながらも驚異的な軽さを実現しました。

--- 画像 ---
URL: https://shopping.geocities.jp/example/img/detail1.jpg
alt: カーボン繊維のクローズアップ
サイズ: 800x400

--- 2カラム ---
左画像URL: https://shopping.geocities.jp/example/img/feature1.jpg
左alt: エアロダイナミクス
左見出し: 空力特性を最適化
左本文: 風洞実験に基づく設計で、高速走行時のダウンフォースを向上。

右画像URL: https://shopping.geocities.jp/example/img/feature2.jpg
右alt: フィッティング
右見出し: 高精度フィッティング
右本文: 純正ボンネットのラインに沿った設計で、隙間なくフィット。

■ セクション: スペック
素材: ドライカーボン（CFRP）
重量: 約4.8kg（純正比 -7.2kg）
適合車種: TOYOTA GR86 (ZN8) / SUBARU BRZ (ZD8)
年式: 2022年～

■ セクション: 注意事項
--- 見出し+本文 ---
見出し: ご購入前にご確認ください
本文: 本製品は競技用パーツです。公道走行時は関連法規をご確認ください。
```

### 7.2 入力変換フロー

```
ユーザーがテキスト入力
       ↓
各コンポーネントに分解
       ↓
コンポーネントごとにHTML生成
（インラインCSS付き）
       ↓
全コンポーネントを結合
       ↓
全体を lp-wrapper の div で囲む
       ↓
文字数チェック（30,000バイト以内）
       ↓
最終HTML出力
```

---

## 8. 出力仕様

### 8.1 出力形式

- HTMLコードのテキスト（コピー＆ペースト用）
- ヤフーショッピングのフリースペース欄にそのまま貼り付け可能な状態

### 8.2 出力ルール

| ルール | 詳細 |
|--------|------|
| `<style>` タグ不使用 | すべてインラインCSS |
| `class` 属性不使用 | ヤフー側で無視される可能性があるため |
| 最外層のラッパー | `<div style="width:100%; max-width:800px; margin:0 auto; ...">` で囲む |
| 画像 | `<img>` タグで絶対URLを指定 |
| 文字コード | UTF-8 |
| 空白・改行 | 出力コードは適度にインデントし可読性を保つ |
| バイト数 | 30,000バイト以内 |

### 8.3 プレビュー

出力HTMLの確認用として、ベースCSS付きのプレビューHTMLファイルも同時生成する。これはヤフーショッピングへの投稿用ではなく、ローカルでデザインを確認するためのもの。

---

## 9. インラインCSS変換対応表

ベースCSSの各クラスに対応するインラインスタイル値の一覧。ツール内部でこの対応表を使用してHTMLを生成する。

| クラス名 | インラインCSS |
|----------|--------------|
| `.lp-wrapper` | `width:100%; max-width:800px; margin:0 auto; padding:0; border:none; background-color:#fff; text-align:left; overflow-x:hidden; box-sizing:border-box;` |
| `.lp-title-main` | `font-weight:900; font-size:28pt; color:#000; line-height:1.0; margin-bottom:0; padding-bottom:0; text-align:left;` |
| `.lp-title-carmodel` | `font-weight:900; font-size:14pt; color:#000; line-height:1.0; margin-bottom:0; padding-bottom:0; text-align:right;` |
| `.lp-title-jp` | `font-weight:600; font-size:9pt; color:#666; margin-top:0; margin-bottom:15px; text-align:left; display:block; line-height:1.4;` |
| `.lp-catch` | `font-weight:600; font-size:13pt; color:#000; margin-top:-5px; margin-bottom:15px; text-align:left; display:block; line-height:1.4;` |
| `.lp-band-title` | `font-weight:300; font-size:15pt; color:#000; border-top:1px solid #000; border-bottom:1px solid #000; padding:12px 0; margin:60px auto 30px; text-align:center; width:800px; max-width:100%; box-sizing:border-box; line-height:1.6;` |
| `.text-heading-lg` | `font-weight:600; font-size:17pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.4; text-align:left;` |
| `.text-heading-md` | `font-weight:600; font-size:16pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.4; text-align:left;` |
| `.text-body` | `font-weight:300; font-size:12pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.6; text-align:left;` |
| `.lp-grid-2` | `display:flex; justify-content:space-evenly; width:800px; max-width:100%; gap:0; margin:0 auto 30px;` |
| `.lp-col` | `width:357px; max-width:48%; text-align:left;` |
| `.lp-col .text-heading-md` | `font-weight:700; font-size:13pt;` |
| `.img-800-400` | `width:800px; height:400px; margin:0 auto 40px; object-fit:cover; display:block;` |
| `.img-800-200` | `width:800px; height:200px; margin:0 auto 5px; object-fit:cover; display:block;` |
| `.img-388-240` | `width:388px; height:240px; margin:0 auto 10px; object-fit:cover; display:block;` |
| `.lp-spec-table` | `width:100%; max-width:800px; border-collapse:collapse; color:#000; margin:0 auto 30px;` |
| `.lp-spec-table th` | `font-weight:300; font-size:12pt; padding:16px; text-align:center; width:30%; vertical-align:middle;` |
| `.lp-spec-table td` | `font-weight:300; font-size:11pt; padding:16px; vertical-align:middle; text-align:left;` |

---

## 10. 注意事項・制約

### 10.1 フォントについて

ベースCSSでは `"Kozuka Gothic Pr6N"` や `"Hiragino Kaku Gothic ProN"` を指定しているが、ヤフーショッピングの表示環境では閲覧者の端末依存となる。インラインCSS内で `font-family` を指定しても、閲覧環境によっては異なるフォントで表示される点に留意する。

### 10.2 `flex` レイアウトの互換性

- `display:flex` はPC版の主要ブラウザでは対応済み
- ヤフーショッピングのPC用フリースペースで `flex` が無効化されていないかは、実機確認が必要
- **フォールバック**: `flex` が使えない場合、`<table>` ベースのレイアウトに切り替える代替版も用意する

### 10.3 バイト数の管理

- インラインCSSは各要素に記述するため、コンポーネント数が増えるとバイト数が急増する
- 目安: コンポーネント10個程度で約8,000〜12,000バイト
- 画像URL の長さもバイト数に含まれるため、短いファイル名を推奨
- 出力時にバイト数を計算し、超過する場合は警告を出す

### 10.4 iframeタグについて

iframeタグは使用可能だが、Yahoo! JAPAN内のサービスURL（`http`/`https`）のみ許可されている。外部サイトへのリンクは削除対象となるため、本ツールでは使用しない。

---

## 11. ファイル構成（プロジェクト内）

| ファイル名 | 役割 |
|-----------|------|
| `CSSテンプレート` | ベースとなるCSS定義。デザインの基準値 |
| `商品タイトル` | タイトルブロックのHTMLテンプレート |
| `画像` | 画像ブロックのHTMLテンプレート |
| `帯` | 帯見出しのHTMLテンプレート（4種のサンプル） |
| `見出し` | 見出し＋本文ブロックのHTMLテンプレート |
| `スペック` | スペック表のHTMLテンプレート |
| `2カラム` | 2カラムレイアウトのHTMLテンプレート |
| `PC用フリースペース1` | 投稿先の仕様メモ |
| `ヤフーショッピング使用可能HTMLタグ一覧.xlsx` | 公式の許可タグ一覧 |

---

## 12. 運用フロー

```
[STEP 1] 画像準備
  → 楽天キャビネット or ヤフートリプルに画像アップロード
  → アップロード後のURLを控える

[STEP 2] テキスト入力
  → 第7章の入力フォーマットに従い、商品情報を入力

[STEP 3] HTML生成
  → ツールが入力を解析し、インラインCSS付きHTMLを生成

[STEP 4] プレビュー確認
  → プレビューHTMLで表示を確認

[STEP 5] 貼り付け
  → 生成されたHTMLをストアクリエイターProの
    「PC用フリースペース1」欄にコピー＆ペースト

[STEP 6] 公開確認
  → ストアページで実際の表示を確認
```

---

## 付録A: 出力サンプル（完全版）

以下は全コンポーネントを使用した出力HTMLのサンプル。

```html
<div style="width:100%; max-width:800px; margin:0 auto; padding:0; border:none; background-color:#fff; text-align:left; overflow-x:hidden; box-sizing:border-box;">

  <!-- 商品タイトル -->
  <div style="display:flex; justify-content:space-between; align-items:flex-end; flex-wrap:wrap;">
    <div style="font-weight:900; font-size:28pt; color:#000; line-height:1.0; text-align:left;">CARBON FIBER HOOD</div>
    <div style="font-weight:900; font-size:14pt; color:#000; line-height:1.0; text-align:right;">for GR86 / BRZ</div>
  </div>
  <div style="font-weight:600; font-size:9pt; color:#666; margin-top:0; margin-bottom:15px; text-align:left; line-height:1.4;">カーボンファイバーフード</div>
  <div style="font-weight:600; font-size:13pt; color:#000; margin-top:-5px; margin-bottom:15px; text-align:left; line-height:1.4;">軽量化と空力性能を両立するドライカーボン製ボンネット</div>

  <!-- メイン画像 -->
  <div style="display:block; position:relative; overflow:hidden; width:800px; height:400px; margin:0 auto 40px;">
    <img src="https://shopping.geocities.jp/example/img/main.jpg" alt="メイン画像" style="width:100%; height:100%; object-fit:cover;">
  </div>

  <!-- 帯: 製品の特長 -->
  <div style="font-weight:300; font-size:15pt; color:#000; border-top:1px solid #000; border-bottom:1px solid #000; padding:12px 0; margin:60px auto 30px; text-align:center; width:800px; max-width:100%; box-sizing:border-box; line-height:1.6;">製品の特長</div>

  <!-- 見出し+本文 -->
  <div style="font-weight:600; font-size:17pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.4; text-align:left;">圧倒的な軽量化を実現</div>
  <div style="font-weight:300; font-size:12pt; color:#000; width:800px; max-width:100%; margin:0 auto 10px; line-height:1.6; text-align:left;">純正スチールフード比で約60%の軽量化を実現。ドライカーボン（CFRP）製法により、高い剛性を維持しながらも驚異的な軽さを実現しました。</div>

  <!-- 帯: スペック -->
  <div style="font-weight:300; font-size:15pt; color:#000; border-top:1px solid #000; border-bottom:1px solid #000; padding:12px 0; margin:60px auto 30px; text-align:center; width:800px; max-width:100%; box-sizing:border-box; line-height:1.6;">スペック</div>

  <!-- スペック表 -->
  <table style="width:100%; max-width:800px; border-collapse:collapse; color:#000; margin:0 auto 30px;">
    <tbody>
      <tr style="background:#CCC;">
        <th style="font-weight:300; font-size:12pt; padding:16px; text-align:center; width:30%; vertical-align:middle;">素材</th>
        <td style="font-weight:300; font-size:11pt; padding:16px; vertical-align:middle; text-align:left;">ドライカーボン（CFRP）</td>
      </tr>
      <tr>
        <th style="font-weight:300; font-size:12pt; padding:16px; text-align:center; width:30%; vertical-align:middle;">重量</th>
        <td style="font-weight:300; font-size:11pt; padding:16px; vertical-align:middle; text-align:left;">約4.8kg（純正比 -7.2kg）</td>
      </tr>
    </tbody>
  </table>

</div>
```

---

*本仕様書バージョン: 1.0*
*作成日: 2026年3月27日*
