---
purpose: TWB の数値書式・色の割り当て・表示名・マークとシェルフ・テキスト・ダッシュボード・計算式の書き方と落とし穴
note: SKILL.md の手順4〜6で読む。Tableau Cloud の描画で動作を確かめた書き方を集めたもの
---

# TWB の書式・色・表示名と落とし穴

TWB を XML で組むときの、数値書式・色・表示名の書き方と、検証を通っても Tableau で失敗するもの・見た目が崩れるもの。土台と接続は `twb-skeleton.md`、見た目の判断と定石は `look-and-feel.md` を参照する。

出典: [wow-challenge-creator](https://github.com/YoshitakaArakawa/wow-challenge-creator)（MIT）の create-workbook スキルの参照資料を、汎用のダッシュボード向けに再構成したもの。

`[ds]` はデータソース名（例 `sqlproxy.xxxx`）の省略。`look-and-feel.md` でも同じ意味で使う。

## 数値書式

列の `default-format` 属性に書く。動作を確かめた書式:

| 用途 | 書式 |
|---|---|
| 通貨 | `c"$"#,##0;-"$"#,##0` |
| 符号付き通貨 | `*+$#,##0;-$#,##0` |
| 符号付き % | `*+0.0%;-0.0%;0.0%` |
| % | `p0.0%` |
| 日付 | `*MMM d, yyyy` |

- 千単位の K 表記（`c"$"#,##0.0,K`）は効かず、値が 1000 倍に見える。使わない
- 符号付き通貨で `"$"` を引用符で囲むと、`+` と `$` の間で改行されることがある
- 文字列を含む書式（例 `0" 件"`）は Tableau Cloud の描画で無視される。文言を付けたいときは文字列を返す計算にする

## 色の割り当て

- 離散フィールドの色の割り当ては、データソース直下の `<style>`（`<layout>` の直後）に書く。ワークシートの `<style>` に書いた割り当ては無視される
- 同じデータソースに、そのフィールドの `<column-instance>` も宣言する

```xml
<column-instance column='[Segment]' derivation='None' name='[none:Segment:nk]' pivot='key' type='nominal' />
<layout ... />
<style>
  <style-rule element='mark'>
    <encoding attr='color' field='[none:Segment:nk]' type='palette'>
      <map to='#4e79a7'><bucket>&quot;Consumer&quot;</bucket></map>
      <map to='#9a9fa6'><bucket>&quot;Corporate&quot;</bucket></map>
      <map to='#cdd1d6'><bucket>&quot;Home Office&quot;</bucket></map>
    </encoding>
  </style-rule>
</style>
```

- 文字列の値は `<bucket>&quot;値&quot;</bucket>` のように `&quot;` で囲む。真偽値は `<bucket>true</bucket>`
- 色の値を兼ねる表示用の文字列は、文言を変えると割り当てが外れる。式と `<bucket>` を同時に直す
- 連続値（利益など）の発散配色は `look-and-feel.md` の「発散型パレットの中心」

## 表示名とヘッダー

- 画面に出る名前は、列の `caption` で日本語にする（例 `<column caption='売上' ... name='[Sales]' />`）
- 行・列のフィールドラベルを消す: `twb-skeleton.md` の KPI シートの節
- 離散ピルのヘッダーを隠す: `<style-rule element='label'>` に `<format attr='display' field='[ds].[none:X:nk]' value='false' />`（`element='header'` の `display` は無視される）
- 連続軸を隠す: `element='axis'` に `scope` 付きで書く
- 見出しの文字の書式: `<style-rule element='header'>` に `field` 指定で書く（`<format attr='color' field='[ds].[none:X:nk]' value='…' />`）

## マークとシェルフ

- 二重軸は `<rows>([A] + [B])</rows>` とし、`pane id='1'` を A、`pane id='2'` を B に対応させる。A が背面に描かれる
- ペインと軸の対応は、行の二重軸なら `y-axis-name`、列の二重軸なら `x-axis-name`
- 同じメジャーを 2 回置く二重軸は区別できない。2 本目は同じ値の別名の計算フィールドにする
- 軸の同期は、ワークシートの `<style-rule element='axis'>` に、B のフィールドを指す `<encoding attr='space' class='0' field='[ds].[B]' field-type='quantitative' fold='true' scope='rows' synchronized='true' type='space' />` を置く
- 線グラフで、点ごとに値が変わるディメンションを詳細に置かない（線が点に切れる）。ツールチップ用なら `ATTR` でツールチップに置く
- 色で分けた線の重なり順は、色の凡例の並び順で決まる（先頭が最前面）
- マークのサイズ（`<format attr='size'>`）は小数で指定できる
- ビュー内の手動ソートは `<sort class='manual' column='…' direction='ASC'><dictionary><bucket>&quot;A&quot;</bucket>…</dictionary></sort>` を `<filter>` の後・`<aggregation>` の前に置く（計算によるソートは `twb-skeleton.md` の `<computed-sort>`）

## テキスト（formatted-text）

- 空白だけの `<run>` は捨てられる。空白は隣の文字を含む run に入れる
- 値が無いときは NULL でなく空文字 `''` を返す（ラベルに不要な表示を出さない）
- ■ █ などの塗りつぶし記号は Tableau Cloud の描画で崩れる。▲ ▼ · — は正しく出る
- テキストのマークは折り返さない。1 行がシートの幅を超えるとラベル全体が `#####` になる。長い文は式の中で `CHAR(10)` で改行する

## ダッシュボード

- シートにスクロールバーが出る、または `#####` になるときは、ダッシュボードの `<window>` に `<zoom type='entire-view' />`（Fit）があるか確かめる
- それでもテキストのシートが `#####` になるのは、多くは表示の高さ不足（次に幅）。ゾーンかダッシュボードの高さを足す
- 固定サイズのゾーンの高さとマージンの合計が、ダッシュボードの高さに収まるようにする。溢れた分は見えなくなる
- ゾーンの `fixed-size` は、外側のパディング（`margin-top`／`margin-bottom`）を除いた中身の高さで書く
- 複数のゾーンを 1 枚の色の面に載せるには、コンテナと子ゾーンの `zone-style` に `background-color` を書き、中のシートの `<style-rule element='table'>` にも同じ色を書く

## 計算式の注意

- アポストロフィを含む文字列は二重引用符で囲む（`"last year's"`）
- 式がエラーのフィールドを 1 つでも参照するシートは、丸ごと空白になる。空白のシートを見たら参照先の式を疑う
- formula 属性の中の改行は `&#13;&#10;` で書く。生の改行は空白になり、`//` コメントが後ろの式を飲み込む
- 週の切り捨ては開始曜日を明示する: `DATETRUNC('week', [Order Date], 'sunday')`
- 直近年の値のように「どのシートでも同じ値」を使うときは、表計算ではなく FIXED LOD で持つ（例 `IF YEAR([Order Date]) = { MAX(YEAR([Order Date])) } THEN [Sales] END`）
- formula 属性に直接書くときは `<` を `&lt;`、`>` を `&gt;`、`&` を `&amp;`、`'` を `&apos;`（属性を `'` で囲む場合）、`"` を `&quot;` にする
