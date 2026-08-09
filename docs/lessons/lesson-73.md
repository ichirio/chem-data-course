# 第73回　tidyverse入門：dplyrでデータ整形

!!! abstract "この回のゴール"
    - **tidyverse** と **パイプ `%>%`** を知る
    - dplyr の5つの動詞（filter, select, mutate, arrange）を使う
    - データ整形を読みやすく書く
    - 所要時間の目安: 60分
    - 使うテーマ：**高分子（ポリマー）**の物性

**tidyverse** は、データ分析を劇的に楽にする R のパッケージ群です。中でも **dplyr** はデータ整形の定番。pandas（第4部）に相当します。

!!! info "準備：tidyverse"
    第1回で入れていなければ、R で次を一度だけ実行します（少し時間がかかります）。
    ```r
    install.packages("tidyverse")
    ```

`stats73.R` を作りましょう。

```r
library(tidyverse)

polymers <- tibble(
  polymer  = c("PE", "PP", "PS", "PVC", "Epoxy", "Phenolic"),
  category = c("thermoplastic", "thermoplastic", "thermoplastic", "thermoplastic", "thermoset", "thermoset"),
  tensile  = c(25, 35, 45, 52, 60, 50),
  density  = c(0.94, 0.905, 1.05, 1.38, 1.20, 1.30)
)
print(polymers)
```

出力:

```text
# A tibble: 6 × 4
  polymer  category      tensile density
  <chr>    <chr>           <dbl>   <dbl>
1 PE       thermoplastic      25   0.94 
2 PP       thermoplastic      35   0.905
3 PS       thermoplastic      45   1.05 
4 PVC      thermoplastic      52   1.38 
5 Epoxy    thermoset          60   1.2  
6 Phenolic thermoset          50   1.3  
```

`tibble` は tidyverse 版のデータフレームです。列の型（`<chr>`＝文字、`<dbl>`＝小数）も表示され、見やすくなっています。

---

## 1. パイプ `%>%`：処理を左から右へつなぐ

tidyverse では、**パイプ `%>%`** で「データを次の処理へ流す」ように書きます。「〜して、それから〜」と読めます。

```r
polymers %>% filter(tensile > 45)
```

これは「polymers を、それから tensile > 45 で絞る」という意味。読む順と処理の順が一致します。

---

## 2. dplyr の動詞

=== "filter（行を絞る）"

    ```r
    polymers %>% filter(tensile > 45)
    ```

    出力:
    ```text
    # A tibble: 3 × 4
      polymer  category      tensile density
      <chr>    <chr>           <dbl>   <dbl>
    1 PVC      thermoplastic      52    1.38
    2 Epoxy    thermoset          60    1.2 
    3 Phenolic thermoset          50    1.3 
    ```

=== "mutate + select（列を追加・選ぶ）"

    ```r
    polymers %>%
      mutate(spec_strength = round(tensile / density, 1)) %>%
      select(polymer, spec_strength)
    ```

    出力:
    ```text
    # A tibble: 6 × 2
      polymer  spec_strength
      <chr>            <dbl>
    1 PE                26.6
    2 PP                38.7
    3 PS                42.9
    4 PVC               37.7
    5 Epoxy             50  
    6 Phenolic          38.5
    ```

=== "arrange（並べ替え）"

    ```r
    polymers %>% arrange(desc(tensile))
    ```

    出力:
    ```text
    # A tibble: 6 × 4
      polymer  category      tensile density
      <chr>    <chr>           <dbl>   <dbl>
    1 Epoxy    thermoset          60   1.2  
    2 PVC      thermoplastic      52   1.38 
    3 Phenolic thermoset          50   1.3  
    4 PS       thermoplastic      45   1.05 
    5 PP       thermoplastic      35   0.905
    6 PE       thermoplastic      25   0.94 
    ```

| dplyr 動詞 | 意味 | pandas 相当 |
|---|---|---|
| `filter()` | 行を絞る | `df[条件]` |
| `select()` | 列を選ぶ | `df[["列"]]` |
| `mutate()` | 列を追加・計算 | `df["新列"]=...` |
| `arrange()` | 並べ替え | `sort_values` |

!!! tip "パイプでつなげる"
    `%>%` で動詞をいくつでもつなげられます。「絞って → 列を足して → 並べ替える」を1つの流れで書けるのが tidyverse の魅力です。R 4.1以降は `|>`（ネイティブパイプ）も使えます。

---

## 演習問題

**問1.** `polymers` から、`density` が 1.0 より大きいポリマーだけを `filter` で取り出してください。

**問2.** `mutate` で「密度を kg/m³ に直した列 `density_kg`（× 1000）」を追加し、`select` で `polymer` と `density_kg` だけを表示してください。

**問3.** `polymers` を `density` の**小さい順**に `arrange` で並べ替えてください（降順は `desc()`、昇順はそのまま）。

---

## 解答

??? success "問1 の解答"
    ```r
    polymers %>% filter(density > 1.0)
    ```

    出力:
    ```text
    # A tibble: 4 × 4
      polymer  category      tensile density
      <chr>    <chr>           <dbl>   <dbl>
    1 PS       thermoplastic      45    1.05
    2 PVC      thermoplastic      52    1.38
    3 Epoxy    thermoset          60    1.2 
    4 Phenolic thermoset          50    1.3 
    ```

??? success "問2 の解答"
    ```r
    polymers %>%
      mutate(density_kg = density * 1000) %>%
      select(polymer, density_kg)
    ```

    出力:
    ```text
    # A tibble: 6 × 2
      polymer  density_kg
      <chr>         <dbl>
    1 PE              940
    2 PP              905
    3 PS             1050
    4 PVC            1380
    5 Epoxy          1200
    6 Phenolic       1300
    ```

??? success "問3 の解答"
    ```r
    polymers %>% arrange(density)
    ```
    密度の小さい順（PP → PE → PS → …）に並びます。`desc()` を付けなければ昇順です。

---

## この回のまとめ

- tidyverse はデータ分析パッケージ群。dplyr がデータ整形の主役。
- **パイプ `%>%`** で処理を左から右へつなぐ（読む順＝処理順）。
- 4動詞：`filter`（行）・`select`（列）・`mutate`（追加）・`arrange`（並べ替え）。
- tibble は型が見えるデータフレーム。

### 次回予告

[第74回：dplyrで集計（group_by・summarise）](lesson-74.md) では、カテゴリ別の集計を学びます。第36回の groupby の R 版です。
