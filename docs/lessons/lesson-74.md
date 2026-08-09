# 第74回　dplyrで集計（group_by・summarise）

!!! abstract "この回のゴール"
    - `group_by` と `summarise` でグループ集計する
    - `n()`・`mean()`・`sd()` などで集計値を求める
    - レポート用の集計表を作る
    - 所要時間の目安: 60分
    - 使うテーマ：**高分子のカテゴリ別集計**

第36回（pandas の groupby）の R 版です。「カテゴリごとに平均を出して表にまとめる」——統計解析・帳票の基本です。

`stats74.R` を作りましょう。

```r
library(tidyverse)

polymers <- tibble(
  polymer  = c("PE", "PP", "PS", "PVC", "Epoxy", "Phenolic"),
  category = c("thermoplastic", "thermoplastic", "thermoplastic", "thermoplastic", "thermoset", "thermoset"),
  tensile  = c(25, 35, 45, 52, 60, 50),
  density  = c(0.94, 0.905, 1.05, 1.38, 1.20, 1.30)
)
```

---

## 1. group_by → summarise の流れ

`group_by(列)` でグループに分け、`summarise(...)` で各グループの集計値を計算します。

```r
polymers %>%
  group_by(category) %>%
  summarise(
    n = n(),                    # 件数
    mean_tensile = mean(tensile),
    .groups = "drop"
  )
```

出力:

```text
# A tibble: 2 × 3
  category          n mean_tensile
  <chr>         <int>        <dbl>
1 thermoplastic     4         39.2
2 thermoset         2         55  
```

カテゴリ別の件数と平均引張強さが、1つの表になりました。読み方は **「グループに分けて（group_by）→ 集計する（summarise）」**。第36回の `groupby(...).agg(...)` と同じ発想です。

!!! note "`n()` と `.groups=\"drop\"`"
    - `n()` は「そのグループの件数」を返す特別な関数。
    - `.groups = "drop"` は集計後にグループ化を解除する指定（付けないと注意メッセージが出ます）。

---

## 2. 複数の集計をまとめる

`summarise` の中に、いくつでも集計を並べられます。

```r
polymers %>%
  group_by(category) %>%
  summarise(
    n = n(),
    mean_tensile = round(mean(tensile), 1),
    sd_tensile = round(sd(tensile), 2),
    mean_density = round(mean(density), 3),
    .groups = "drop"
  )
```

出力:

```text
# A tibble: 2 × 5
  category          n mean_tensile sd_tensile mean_density
  <chr>         <int>        <dbl>      <dbl>        <dbl>
1 thermoplastic     4         39.2      11.8          1.07
2 thermoset         2         55         7.07         1.25
```

これは**そのままレポートに載せられる集計表**です。第4部の pandas と同じことが、R でもできます。

---

## 3. 集計表を保存する

作った表は CSV に保存できます（帳票・共有用）。

```r
summary_table <- polymers %>%
  group_by(category) %>%
  summarise(mean_tensile = mean(tensile), .groups = "drop")

write_csv(summary_table, "polymer_summary.csv")
```

`write_csv`（tidyverse）は、pandas の `to_csv` にあたります。

---

## 演習問題

**問1.** `polymers` をカテゴリ別にグループ化し、`summarise` で件数（`n()`）と**平均密度**（`mean(density)`）を表示してください。

**問2.** カテゴリ別に、引張強さの**最大値**（`max(tensile)`）と**最小値**（`min(tensile)`）をまとめた表を作ってください。

**問3.** 問2の表を `write_csv` で `tensile_range.csv` に保存してください。

---

## 解答

??? success "問1 の解答"
    ```r
    polymers %>%
      group_by(category) %>%
      summarise(n = n(), mean_density = round(mean(density), 3), .groups = "drop")
    ```

    出力:
    ```text
    # A tibble: 2 × 3
      category          n mean_density
      <chr>         <int>        <dbl>
    1 thermoplastic     4         1.07
    2 thermoset         2         1.25
    ```

??? success "問2 の解答"
    ```r
    polymers %>%
      group_by(category) %>%
      summarise(max_tensile = max(tensile), min_tensile = min(tensile), .groups = "drop")
    ```

    出力:
    ```text
    # A tibble: 2 × 3
      category      max_tensile min_tensile
      <chr>               <dbl>       <dbl>
    1 thermoplastic          52          25
    2 thermoset              60          50
    ```

??? success "問3 の解答"
    ```r
    result <- polymers %>%
      group_by(category) %>%
      summarise(max_tensile = max(tensile), min_tensile = min(tensile), .groups = "drop")
    write_csv(result, "tensile_range.csv")
    ```

---

## この回のまとめ

- `group_by(列) %>% summarise(...)` でグループ集計。
- `n()`（件数）・`mean`・`sd`・`max`・`min` などを並べて集計表に。
- `.groups = "drop"` で集計後のグループ化を解除。
- `write_csv()` で保存（帳票・共有）。

### 次回予告

[第75回：tidyrでデータ整形（pivot）](lesson-75.md) では、縦長・横長のデータ変換（ピボット）を学びます。
