# 第77回　記述統計と要約

!!! abstract "この回のゴール"
    - 中心（平均・中央値）とばらつき（標準偏差・分散）を求める
    - `summary()` で一気に要約する
    - グループごとの要約を出す
    - 所要時間の目安: 60分
    - 使うテーマ：**測定値の要約**

データ分析の第一歩は「データの姿をつかむ」こと。記述統計はそのための基本です。

`stats77.R` を作りましょう。

```r
library(tidyverse)
tensile <- c(25, 35, 45, 52, 60, 50)
```

---

## 1. 中心とばらつき

```r
mean(tensile)       # 平均
median(tensile)     # 中央値
sd(tensile)         # 標準偏差（標本、n-1）
var(tensile)        # 分散
range(tensile)      # 最小・最大
IQR(tensile)        # 四分位範囲
```

出力:

```text
[1] 44.5
[1] 47.5
[1] 12.62933
[1] 159.5
[1] 25 60
[1] 14
```

R では、これらの関数がすべて標準装備です（`import` 不要）。ここが統計に強い R の面目躍如です。

---

## 2. summary()：一気に要約

`summary()` は、最小・四分位数・中央値・平均・最大をまとめて出します。

```r
summary(tensile)
```

出力:

```text
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   25.0    37.5    47.5    44.5    51.5    60.0 
```

第31回の pandas `describe()` にあたります。データの分布の要点が一目で分かります。

---

## 3. グループごとの要約（dplyr）

第74回の group_by と組み合わせれば、カテゴリ別の記述統計が出せます。

```r
polymers <- tibble(
  category = c("thermoplastic", "thermoplastic", "thermoplastic", "thermoplastic", "thermoset", "thermoset"),
  tensile  = c(25, 35, 45, 52, 60, 50)
)

polymers %>%
  group_by(category) %>%
  summarise(
    n = n(),
    mean = round(mean(tensile), 1),
    sd = round(sd(tensile), 2),
    median = median(tensile),
    .groups = "drop"
  )
```

出力:

```text
# A tibble: 2 × 5
  category          n  mean    sd median
  <chr>         <int> <dbl> <dbl>  <dbl>
1 thermoplastic     4  39.2 11.8     47.5
2 thermoset         2  55    7.07    55  
```

!!! note "平均と中央値の違い"
    - **平均（mean）**：全部を足して割る。外れ値の影響を受けやすい。
    - **中央値（median）**：真ん中の値。外れ値に強い（第42回）。

    両方を見ると、データの偏りに気づけます。差が大きいときは、分布が歪んでいるサインです。

---

## 演習問題

**問1.** 測定値 `data <- c(12.1, 12.3, 12.0, 12.4, 12.2, 25.0)`（最後は外れ値）について、平均と中央値を計算し、どちらが外れ値の影響を受けやすいか確かめてください。

**問2.** 同じ `data` に `summary()` を使って、要約統計を表示してください。

**問3.** `density <- c(0.94, 0.905, 1.05, 1.38, 1.20, 1.30)` の標準偏差（`sd`）と四分位範囲（`IQR`）を計算してください。

---

## 解答

??? success "問1 の解答"
    ```r
    data <- c(12.1, 12.3, 12.0, 12.4, 12.2, 25.0)
    mean(data)      # [1] 14.33333
    median(data)    # [1] 12.25
    ```
    平均は外れ値25に引っ張られて14.3ですが、中央値は12.25と実態に近い。**中央値は外れ値に強い**ことが分かります。

??? success "問2 の解答"
    ```r
    summary(data)
    ```

    出力:
    ```text
       Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      12.00   12.12   12.25   14.33   12.38   25.00 
    ```

??? success "問3 の解答"
    ```r
    density <- c(0.94, 0.905, 1.05, 1.38, 1.20, 1.30)
    sd(density)     # [1] 0.1946386
    IQR(density)    # [1] 0.3075
    ```

---

## この回のまとめ

- `mean` `median` `sd` `var` `range` `IQR` が標準装備。
- `summary()` で最小・四分位・中央値・平均・最大を一気に。
- `group_by %>% summarise` でグループ別の記述統計。
- 平均は外れ値に弱く、中央値は強い。両方を見る。

### 次回予告

[第78回：正規分布と確率](lesson-78.md) では、統計の土台となる正規分布を学びます。
