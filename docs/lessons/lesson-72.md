# 第72回　Rの基本文法（ベクトル・データフレーム）

!!! abstract "この回のゴール"
    - R の中心となる**ベクトル**を作り、計算する
    - **データフレーム**（表）を作る
    - 列や要素を取り出す
    - 所要時間の目安: 60分
    - 使うテーマ：**測定値・元素データ**

`stats72.R` を作りましょう（または R の対話モードで）。

---

## 1. ベクトル：値の並び

R では `c()`（combine）で複数の値をまとめた**ベクトル**を作ります。R の計算は、このベクトルが基本単位です。

```r
yields <- c(88.5, 90.1, 89.3, 91.0)   # 収率のベクトル

print(yields)
mean(yields)      # 平均
sd(yields)        # 標準偏差（Rは既定で標本＝n-1で割る）
length(yields)    # 要素数
```

出力:

```text
[1] 88.5 90.1 89.3 91.0
[1] 89.725
[1] 1.072016
[1] 4
```

!!! note "R のベクトルは一括計算"
    NumPy の配列（第21回）と同じく、R のベクトルも一括計算できます。
    ```r
    temps_C <- c(0, 25, 100)
    temps_C + 273.15    # [1] 273.15 298.15 373.15
    ```
    R では**標準偏差 `sd()` が既定で標本（n−1）**です（Python の NumPy は既定が n。第25回参照）。

---

## 2. データフレーム：表を作る

**データフレーム**は、列を組み合わせた表です（pandas の DataFrame にあたる）。`data.frame()` で作ります。

```r
elements <- data.frame(
  symbol = c("H", "C", "N", "O"),
  mass   = c(1.008, 12.011, 14.007, 15.999)
)

print(elements)
```

出力:

```text
  symbol   mass
1      H  1.008
2      C 12.011
3      N 14.007
4      O 15.999
```

左端の 1,2,3,4 は行番号です。列名（symbol, mass）が付いた表になりました。

---

## 3. 列・要素を取り出す

列は `$` で取り出します（R 独特の書き方）。

```r
elements$symbol        # symbol 列
elements$mass          # mass 列
mean(elements$mass)    # mass 列の平均

elements[1, ]          # 1行目（全列）
elements[, "mass"]     # mass 列（[行, 列] の書き方）
```

出力（一部）:

```text
[1] "H" "C" "N" "O"
[1]  1.008 12.011 14.007 15.999
[1] 10.75625
```

!!! tip "`$` で列を指す"
    `データフレーム$列名` が、R で列を取り出す基本形です。`elements$mass` は「elements の mass 列」。pandas の `df["mass"]` にあたります。

---

## 演習問題

**問1.** 測定値のベクトル `data <- c(12.1, 12.3, 12.0, 12.4, 12.2)` を作り、平均・標準偏差・要素数を表示してください。

**問2.** 3つのポリマーについて、`polymer`（名前）と `tensile`（引張強さ）の2列を持つデータフレームを作って表示してください。例：PE 25、PP 35、PS 45。

**問3.** 問2のデータフレームで、`tensile` 列の平均を `$` を使って計算してください。

---

## 解答

??? success "問1 の解答"
    ```r
    data <- c(12.1, 12.3, 12.0, 12.4, 12.2)
    mean(data)      # [1] 12.2
    sd(data)        # [1] 0.1581139
    length(data)    # [1] 5
    ```

??? success "問2 の解答"
    ```r
    polymers <- data.frame(
      polymer = c("PE", "PP", "PS"),
      tensile = c(25, 35, 45)
    )
    print(polymers)
    ```

    出力:
    ```text
      polymer tensile
    1      PE      25
    2      PP      35
    3      PS      45
    ```

??? success "問3 の解答"
    ```r
    mean(polymers$tensile)     # [1] 35
    ```

---

## この回のまとめ

- `c(...)` でベクトルを作る。`mean` `sd` `length` などで集計。
- R の `sd()` は既定で標本標準偏差（n−1）。
- `data.frame(...)` で表を作る。`$列名` で列を取り出す。
- ベクトルは NumPy 配列、データフレームは pandas DataFrame にあたる。

### 次回予告

[第73回：tidyverse入門（dplyr）](lesson-73.md) では、データ整形を劇的に楽にする tidyverse を導入します。
