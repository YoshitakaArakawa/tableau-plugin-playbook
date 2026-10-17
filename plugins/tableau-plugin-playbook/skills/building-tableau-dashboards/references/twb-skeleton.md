---
purpose: 公開データソースに接続した TWB を XML で組むときの土台と、公開で失敗しやすい点
note: SKILL.md の手順4〜6で読む。Tableau Cloud の描画で動作を確かめた書き方を集めたもの
---

# TWB の土台（公開データソースへの接続）

実際に公開まで通った TWB（Tableau のワークブックのファイル）から、土台に使う部分だけを抜き出したもの。`{...}` は置き換える。

## 先に読む: 公開で失敗しやすい点

- 最初に `download-workbook` を試し、使えれば既存のワークブックを雛形にする。使えなければこの土台から組む
- 接続先のデータソースは `list-datasources` で実名と contentUrl を確かめ、下の `SampleSuperStore` を置き換える
- ソートは `<computed-sort>` で書く。`<sort class='computed'>` は公開時の検証で落ちる
- 日付の月切り捨ては派生名 `Month-Trunc`、接頭辞 `tmn`。`TruncMonth` は公開時の検証で落ちる
- `SheetIdentifierTracking` を宣言したら、worksheet・window・dashboard ごとに `<simple-id uuid='{GUID}' />` を入れる
- ダッシュボードの `<datasources />` は空でも省略しない
- ゾーンは固定座標（tiled）で置く。横並び（layout-flow）のコンテナは崩れやすい
- ローカルの検証が通っても公開時に落ちることがある。公開できても `get-view-data` で各シートの `sheetStatus` を確かめる（ERROR のシートが残ることがある）

## ルートと宣言

```xml
<?xml version='1.0' encoding='utf-8' ?>
<workbook original-version='18.1' source-build='2026.2.0 (20262.26.0528.2205)' source-platform='mac' version='18.1' xmlns:user='http://www.tableausoftware.com/xml/user'>
  <document-format-change-manifest>
    <SheetIdentifierTracking />
    <SortTagCleanup />
    <WindowsPersistSimpleIdentifiers />
  </document-format-change-manifest>
  <preferences />
  <datasources> ... </datasources>
  <worksheets> ... </worksheets>
  <dashboards> ... </dashboards>
  <windows> ... </windows>
</workbook>
```

## 公開データソースの接続（sqlproxy）

```xml
<datasource caption='Sample SuperStore' inline='true' name='{DS_NAME}' version='18.1'>
  <repository-location derived-from='https://{SERVER}/t/{SITE_NAME}/datasources/SampleSuperStore?rev=1.0' id='SampleSuperStore' path='/t/{SITE_NAME}/datasources' revision='1.0' site='{SITE_NAME}' />
  <connection channel='https' class='sqlproxy' dbname='SampleSuperStore' directory='/dataserver' port='443' server='{SERVER}' username='' workgroup-auth-mode='prompt'>
    <relation name='sqlproxy' table='[sqlproxy]' type='table' />
  </connection>
  <aliases enabled='yes' />
  <!-- 使う列と計算フィールドの <column> 宣言 -->
</datasource>
```

`{DS_NAME}` は `sqlproxy.` に任意の英数字を続けた名前（例 `sqlproxy.0k3m9x2a7b4c1d8e5f6g0h2j4l6n`）。シート側の参照もこの名前に揃える。

## 列とインスタンス

- 基本列: `<column datatype='real' name='[Sales]' role='measure' type='quantitative' />`
- 計算フィールド: `<column caption='利益率' datatype='real' name='[Calculation_ProfitRatio]' role='measure' type='quantitative' default-format='p0.0%'><calculation class='tableau' formula='SUM([Profit]) / SUM([Sales])' /></column>`
- インスタンス名は `[{接頭辞}:{列}:{nk|qk}]`

| derivation | 接頭辞 | 末尾 | type |
|---|---|---|---|
| None | none | nk | nominal |
| Sum | sum | qk | quantitative |
| CountD | ctd | qk | quantitative |
| User（集計済みの計算） | usr | qk | quantitative |
| Month-Trunc | tmn | qk | quantitative |

## ソート

```xml
<computed-sort column='[{DS_NAME}].[none:Sub-Category:nk]' direction='DESC' using='[{DS_NAME}].[sum:Sales:qk]' />
```

## KPI シート（タイトルを隠し、ラベルで見出しと数値）

- マークは Text。`<customized-label><formatted-text>` の中で、見出しの run（fontsize 11、灰色）と数値の run（fontsize 24）を改行で並べる
- ダッシュボードのゾーンに `show-title='false'` を付ける
- 行と列のフィールドラベルを隠す: `<style-rule element='worksheet'><format attr='display-field-labels' scope='rows' value='false' /><format attr='display-field-labels' scope='cols' value='false' /></style-rule>`

## ダッシュボード

- `<size maxheight='900' maxwidth='1200' minheight='900' minwidth='1200' sizing-mode='fixed' />`
- 最上位を `type-v2='layout-basic'` のゾーンにし、子ゾーンを 0〜100000 の座標（x・y・w・h）で置く

## 見た目を整えるには

この土台だけでは、色・数値書式・表示名が Tableau の既定のままになる。

- 色・書式・表示名の書き方: `twb-pitfalls-and-formulas.md`
- 見た目の判断基準と定石: `look-and-feel.md`
