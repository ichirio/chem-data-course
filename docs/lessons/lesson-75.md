# 第75回　tidyrでデータ整形（pivot）

!!! abstract "この回のゴール"
    - 「横長」と「縦長」のデータの違いを知る
    - `pivot_longer` で横長 → 縦長に変換する
    - `pivot_wider` で縦長 → 横長に変換する
    - 所要時間の目安: 60分
    - 使うテーマ：**触媒 × 温度の収率**

分析やグラフ化には「縦長（tidy）」の形が向き、人が見るには「横長」が向きます。両者を行き来するのが **pivot** です。

`stats75.R` を作りましょう。

```r
library(tidyverse)

# 横長データ（触媒×温度の収率）
wide <- tibble(
  catalyst = c("Pd", "Pt", "Ni"),
  t50  = c(62, 55, 40),
  t80  = c(78, 70, 58),
  t110 = c(85, 80, 66)
)
print(wide)
```

出力:

```text
# A tibble: 3 × 4
  catalyst   t50   t80  t110
  <chr>    <dbl> <dbl> <dbl>
1 Pd          62    78    85
2 Pt          55    70    80
3 Ni          40    58    66
```

---

## 1. pivot_longer：横長 → 縦長

温度の列（t50, t80, t110）を、1つの「温度」列と「収率」列にまとめます。

```r
long <- wide %>%
  pivot_longer(cols = starts_with("t"), names_to = "temp", values_to = "yield")
print(long)
```

出力（先頭6行）:

```text
# A tibble: 9 × 3
  catalyst temp  yield
  <chr>    <chr> <dbl>
1 Pd       t50      62
2 Pd       t80      78
3 Pd       t110     85
4 Pt       t50      55
5 Pt       t80      70
6 Pt       t110     80
```

各行が「1つの条件（触媒×温度）＝1つの測定」になりました。この**縦長（tidy）**の形は、dplyr の集計や ggplot2 のグラフ化に最適です。

!!! note "なぜ縦長が良いのか"
    ggplot2（次回）や dplyr は、「1行＝1観測」の縦長データを前提に設計されています。横長のままだと `group_by` や `aes` で扱いにくいのです。**分析の前に pivot_longer で縦長にする**——これは tidyverse の定石です。

---

## 2. pivot_wider：縦長 → 横長

逆に、縦長を横長（人が見やすい表）に戻します。

```r
back <- long %>%
  pivot_wider(names_from = "temp", values_from = "yield")
print(back)
```

出力:

```text
# A tibble: 3 × 4
  catalyst   t50   t80  t110
  <chr>    <dbl> <dbl> <dbl>
1 Pd          62    78    85
2 Pt          55    70    80
3 Ni          40    58    66
```

元の横長に戻りました。**レポートの表は横長、分析は縦長**、と使い分けます（pandas の pivot_table・melt にあたります）。

---

## 演習問題

**問1.** 本文の `wide` を `pivot_longer` で縦長にし、全9行を表示してください（`print(long, n = 9)` で全行表示）。

**問2.** 縦長にした `long` を dplyr で集計し、**触媒ごとの平均収率**を求めてください（`group_by(catalyst) %>% summarise(...)`）。縦長だと集計が簡単なことを体験しましょう。

**問3.** `long` を `pivot_wider` で横長に戻し、元の `wide` と同じ形になることを確認してください。

---

## 解答

??? success "問1 の解答"
    ```r
    long <- wide %>% pivot_longer(cols = starts_with("t"), names_to = "temp", values_to = "yield")
    print(long, n = 9)
    ```
    9行すべて（Pd/Pt/Ni × t50/t80/t110）が表示されます。

??? success "問2 の解答"
    ```r
    long %>%
      group_by(catalyst) %>%
      summarise(mean_yield = mean(yield), .groups = "drop")
    ```

    出力:
    ```text
    # A tibble: 3 × 2
      catalyst mean_yield
      <chr>         <dbl>
    1 Ni             54.7
    2 Pd             75  
    3 Pt             68.3
    ```
    縦長にしておくと、`group_by` 一発で集計できます。

??? success "問3 の解答"
    ```r
    long %>% pivot_wider(names_from = "temp", values_from = "yield")
    ```
    元の `wide`（触媒×温度の横長表）に戻ります。

---

## この回のまとめ

- 「縦長（tidy）＝1行1観測」は分析・グラフ向き、「横長」は閲覧向き。
- `pivot_longer(cols=, names_to=, values_to=)` で横長 → 縦長。
- `pivot_wider(names_from=, values_from=)` で縦長 → 横長。
- 分析前に縦長化するのが tidyverse の定石。

### 次回予告

[第76回：ggplot2入門](lesson-76.md) では、R の強力な作図ライブラリ ggplot2 で、美しい統計グラフを描きます。
