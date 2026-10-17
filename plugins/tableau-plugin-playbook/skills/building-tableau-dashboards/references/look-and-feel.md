---
purpose: Tableau のダッシュボードを人に見せられる見た目にするための判断基準と、少ない手順で効く定石
note: SKILL.md の手順4〜6で読む。Tableau Cloud の描画で動作を確かめた書き方を集めたもの
---

# 見た目を整える判断基準と軽量テクニック

Tableau のダッシュボードを「人に見せられる見た目」にするための判断基準と、少ない手順で効く定石。XML の基本的な落とし穴・書式・色の割り当ては `twb-pitfalls-and-formulas.md` を参照する。

出典: [wow-challenge-creator](https://github.com/YoshitakaArakawa/wow-challenge-creator)（MIT）の create-workbook スキルの参照資料を、汎用のダッシュボード向けに再構成したもの。元資料の情報源は Flerlage Twins、VizWiz、Tableau Help、日本語コミュニティの記事など。

## 目次

- 判断基準（明るさの 3 段・色の使い方・見せ場は 1 か所・線とマーク）
- 定石の一覧
- 実証済みの XML
- 扱わないもの

## 判断基準

### 明るさの 3 段

「メリハリがない」原因は、彩度より明るさの段階の無さであることが多い。要素を役割で 3 段に分け、明るさで差を付ける。

| 段 | 要素 | 色の例 |
|---|---|---|
| 背景 | 比較の棒、前年の線、帯、区切り線 | 棒 `#cdd1d6`、前年の線 `#d6d9de`、帯 `#f1f4f7` |
| 主役 | 主系列の線や棒、KPI の数値 | 線 `#676f7a`、数値 `#1d1f24` |
| 信号 | 意味を持つ色（良い・悪い・注目） | `#2063b0` / `#a6c0e1` / `#e4a59f` / `#a83634` |

- 背景を一段明るくすると、主役や信号の彩度を上げなくても目立つ。全体を濃くする前に試す
- 平均線などの補助線は主役より一段淡くする（例 `#9a9fa6`）

### 色の使い方

- 色は意味に使う。比較の対象はグレー、主役か注目点だけに色を付ける
- 赤は悪い値（赤字・マイナス・基準割れ）にだけ使う。カテゴリの色分けに赤を入れない
- カテゴリの色分けが主題のときは、3〜4 色までに絞り、赤を避けた落ち着いた色を選ぶ
- グレーは暖色系か寒色系の 1 系統にそろえる
- 正負のある量（利益など）は、0 を中心にした発散配色にする（下の「発散型パレットの中心」）
- 色の意味は記号（▲ ▼）や言葉と一緒に出す

### 見せ場は 1 か所

- ダッシュボードの問い（例「どこが儲かっていないか」）に答える部分を見せ場にする。見せ場にだけ淡い背景のパネルを付け、他は静かにする
- 見せ場の中では、強い色が注目すべきものと一致するようにする
- 区切り線や凡例は、内容の事実（まとまりの境目、線の意味）を表すときだけ置く

### 線とマーク

- 主役の線は濃く細く。太い中間色の線は強すぎて見える
- 比較の線は主役の後ろに描く（色の凡例で主役を先頭に並べる）
- 強調点の円には線と同じ色の枠を付ける。大きさは控えめにする
- 線や帯の意味は、見出しの下に 1 行の凡例で書く

## 定石の一覧

見た目の課題（ラベルの重なり、色の多さ、強調点が分からない等）に当たったら、計算や工程を足す前にここから選ぶ。「実証済み」は Tableau Cloud の描画で効くことを確かめた書き方が下にある。ここに無い技（参照線・二重軸など）は、元資料の wow-challenge-creator を参照。

| 技 | 解決すること | 作り方の要点 |
|---|---|---|
| 強調色＋グレー | 色が多すぎて焦点がぼける | 主役だけ色、比較対象はグレー |
| ノイズ除去 | グリッド・ゼロ線・不要なヘッダーが邪魔 | 線をオフ、ヘッダーを隠す、棒の軸は 0 起点 |
| BAN／KPI カード | 最重要の数値を即座に読ませる | テキストマークで大きな数値、比較を小さく併記 |
| ラベルの表示条件 | 全点ラベルが騒がしい | 最新・線端・最小／最大だけにラベル（実証済み） |
| ラベルの配置・向き | 重なり・はみ出し・縦書き化 | 配置と向きを指定（実証済み） |
| 正負で分ける数値書式（▲▼、+/−） | 増減の向きを計算なしで示す | 書式を「正;負;ゼロ」で書き分ける |
| `MIN(1.0)` のダミー軸で色付きタイル | セルを色で埋める、KPI の枠を作る | 列に `MIN(1.0)` の棒、軸を 0〜1 に固定して隠す（実証済み） |
| 空白 1 個の区切り線 | まとまりの境目を示す | 背景色付きの空白ゾーンにパディング（実証済み） |
| 発散型パレットの中心を基準値に | 良い／悪いの境目を正しく示す | 色の中心を 0 や平均にする（実証済み） |
| マークの枠線 | タイルの境目・強調点の輪郭がぼける | 枠線の色を指定（実証済み） |

四角マークで作るハイライト表は、生成した TWB ではセルが埋まりきらないことがある。色付きタイルは `MIN(1.0)` のダミー軸で作る。

## 実証済みの XML

`[ds]` はデータソース名の省略（`twb-pitfalls-and-formulas.md` と同じ）。

### 発散型パレットの中心

ワークシートの `<style>` の `element='mark'` に書く。

```xml
<style-rule element='mark'>
  <encoding attr='color' center='0.0' field='[ds].[sum:Profit:qk]' palette='orange_blue_diverging_10_0' type='interpolated' />
</style-rule>
```

`num-steps='N'` で段階色、`reverse='true'` で反転。

### ラベルの表示条件

ペインの `element='mark'` に書く。`mark-labels-mode` は `all` / `most-recent` / `line-ends` / `range`（最小／最大）/ `selection` / `highlight`。

```xml
<style-rule element='mark'>
  <format attr='mark-labels-show' value='true' />
  <format attr='mark-labels-mode' value='line-ends' />
  <format attr='mark-labels-line-first' value='false' />
  <format attr='mark-labels-line-last' value='true' />
</style-rule>
```

### ラベルの配置・向き

ペインの `<style>` の `element='cell'` に書く（`element='mark'` では効かない）。`text-orientation` の `0` が横書き。

```xml
<style>
  <style-rule element='cell'>
    <format attr='text-orientation' value='0' />
    <format attr='text-align' value='left' />
    <format attr='vertical-align' value='top' />
  </style-rule>
  <style-rule element='mark'>
    <format attr='mark-labels-show' value='true' />
  </style-rule>
</style>
```

ラベルはペインの境界で切られる。

### `MIN(1.0)` のダミー軸で色付きタイル

```xml
<!-- データソースの計算フィールド -->
<column caption='Tile Length' datatype='real' name='[Calculation_900]' role='measure' type='quantitative'>
  <calculation class='tableau' formula='MIN(1.0)' />
</column>

<!-- ワークシートの datasource-dependencies -->
<column-instance column='[Calculation_900]' derivation='User' name='[usr:Calculation_900:qk]' pivot='key' type='quantitative' />

<!-- 列 = 指標 / ダミー軸 -->
<cols>([ds].[none:Metric:nk] / [ds].[usr:Calculation_900:qk])</cols>

<!-- ワークシートの style：軸を 0〜1 に固定して隠す -->
<style-rule element='axis'>
  <encoding attr='space' class='0' field='[ds].[usr:Calculation_900:qk]' field-type='quantitative'
      min='0' max='1' range-type='fixed' scope='cols' type='space' />
  <format attr='display' class='0' field='[ds].[usr:Calculation_900:qk]' scope='cols' value='false' />
  <format attr='title' class='0' field='[ds].[usr:Calculation_900:qk]' scope='cols' value='' />
</style-rule>
<style-rule element='cell'><format attr='text-align' value='center' /></style-rule>

<!-- ペイン：棒を最大の太さに -->
<mark class='Bar' />
<mark-sizing mark-sizing-setting='marks-scaling-off' />
<style><style-rule element='mark'><format attr='size' value='10' /></style-rule></style>
```

### マークの枠線

```xml
<style-rule element='mark'>
  <format attr='has-stroke' value='true' />
  <format attr='stroke-color' value='#ffffff' />
</style-rule>
```

### 空白 1 個の区切り線

線の太さは `fixed-size`、上下の余白は `margin-top`／`margin-bottom`、長さは `margin-left`／`margin-right` で決まる。

```xml
<zone id='105' type-v2='empty' fixed-size='2' is-fixed='true' x='2857' y='14368' w='94286' h='2874'>
  <zone-style>
    <format attr='border-style' value='none' />
    <format attr='background-color' value='#caced3' />
    <format attr='margin-top' value='14' />
    <format attr='margin-bottom' value='14' />
    <format attr='margin-left' value='150' />
    <format attr='margin-right' value='150' />
  </zone-style>
</zone>
```

## 扱わないもの

計算や工程が多く、少ない手順で整える範囲を超えるもの: Sankey、radial、waffle、donut、waterfall、funnel、radar などの自作チャート、データの水増し、ダークモード、シートの切り替え。
