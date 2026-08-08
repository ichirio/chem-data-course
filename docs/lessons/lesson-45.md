# 第45回　まとめ演習：実験ノートCSVを分析する

!!! abstract "この回のゴール"
    - 第4部で学んだことを**1つの流れ**として通す
    - 読み込み → 確認 → 前処理 → 集計 → 保存 の一気通貫を体験する
    - 「分析の型」を自分のものにする
    - 所要時間の目安: 60分（第4部の総仕上げ）
    - 使うデータ：**触媒反応の実験ノート**（欠損あり）

`lesson45.py` を作りましょう。今回は実験記録を模した、少し現実的なデータを扱います。

```python
import pandas as pd

exp = pd.DataFrame({
    "run":       [1, 2, 3, 4, 5, 6, 7, 8],
    "catalyst":  ["Pd", "Pd", "Pt", "Pt", "Ni", "Ni", "Pd", "Pt"],
    "temp_C":    [80, 110, 80, 110, 80, 110, 80, 110],
    "yield_pct": [78, 85, 70, 80, 58, 66, None, 82],   # run 7 は未測定
})
```

（実際の分析では、この表は `pd.read_csv("experiment.csv")` で読み込むのが普通です。今回は自作します。）

---

## ステップ1　まず全体を確認する（第31・35回）

```python
print(exp)
print("形:", exp.shape)
print("欠損数:")
print(exp.isna().sum())
```

出力:

```text
   run catalyst  temp_C  yield_pct
0    1       Pd      80       78.0
1    2       Pd     110       85.0
2    3       Pt      80       70.0
3    4       Pt     110       80.0
4    5       Ni      80       58.0
5    6       Ni     110       66.0
6    7       Pd      80        NaN
7    8       Pt     110       82.0
形: (8, 4)
欠損数:
run          0
catalyst     0
temp_C       0
yield_pct    1
dtype: int64
```

`yield_pct` に欠損が1つ（run 7）あると分かりました。

---

## ステップ2　前処理：欠損行を除く（第35回）

今回は「未測定は集計から外す」方針にします（除いた事実は記録します）。

```python
exp_clean = exp.dropna()
print("欠損除去後の件数:", len(exp_clean))
```

出力:

```text
欠損除去後の件数: 7
```

---

## ステップ3　集計：触媒ごとの成績表（第36回）

触媒ごとに「試行回数・平均収率・最高収率」をまとめます。これが**レポートに載せる集計テーブル**です。

```python
summary = exp_clean.groupby("catalyst").agg(
    runs=("run", "count"),
    yield_mean=("yield_pct", "mean"),
    yield_max=("yield_pct", "max"),
).round(1)

print(summary)
```

出力:

```text
          runs  yield_mean  yield_max
catalyst                             
Ni           2        62.0       66.0
Pd           2        81.5       85.0
Pt           3        77.3       82.0
```

Pd の平均が最も高い、と読み取れます。

---

## ステップ4　深掘り：最高収率の条件を探す（第36回）

```python
best = exp_clean.sort_values("yield_pct", ascending=False).head(1)
print(best)
```

出力:

```text
   run catalyst  temp_C  yield_pct
1    2       Pd     110       85.0
```

**最高収率は run 2（Pd・110℃）で 85%** と分かりました。

---

## ステップ5　保存：結果を残す（第32回）

集計テーブルを CSV に保存して、レポートや次の解析に使えるようにします。

```python
summary.to_csv("catalyst_summary.csv")
print("catalyst_summary.csv を保存しました")
```

!!! success "これが「分析の型」"
    **① 読む → ② 確認する → ③ 整える → ④ 集計する → ⑤ 保存する。**
    分野が材料でも高分子でも生化学でも、表データの分析はこの5ステップの繰り返しです。第4部で、その型が身につきました。

---

## 演習問題

**問1.** 本文の `exp` を作り、`shape` と `isna().sum()` を表示して、どこに欠損があるか確認してください。

**問2.** 欠損行を除いたあと、**温度（temp_C）ごと**の平均収率を `groupby` で計算して表示してください（触媒ではなく温度でグループ化）。

**問3.** 欠損行を除いたデータで、`agg` を使って触媒ごとに「試行回数 `runs`」「平均収率 `yield_mean`」「収率の標準偏差 `yield_std`」の集計テーブルを作り、`summary2.csv` に保存してください。

---

## 解答

??? success "問1 の解答"
    ```python
    print(exp.shape)
    print(exp.isna().sum())
    ```

    出力:
    ```text
    (8, 4)
    run          0
    catalyst     0
    temp_C       0
    yield_pct    1
    dtype: int64
    ```

??? success "問2 の解答"
    ```python
    exp_clean = exp.dropna()
    print(exp_clean.groupby("temp_C")["yield_pct"].mean().round(2))
    ```

    出力:
    ```text
    temp_C
    80     68.67
    110    78.25
    Name: yield_pct, dtype: float64
    ```

    高温（110℃）の方が平均収率が高い傾向が見えます。

??? success "問3 の解答"
    ```python
    exp_clean = exp.dropna()
    summary2 = exp_clean.groupby("catalyst").agg(
        runs=("run", "count"),
        yield_mean=("yield_pct", "mean"),
        yield_std=("yield_pct", "std"),
    ).round(2)
    print(summary2)
    summary2.to_csv("summary2.csv")
    ```

    出力:
    ```text
              runs  yield_mean  yield_std
    catalyst                             
    Ni           2       62.00       5.66
    Pd           2       81.50       4.95
    Pt           3       77.33       6.43
    ```

---

## 第4部　修了

おめでとうございます！ これで化学データを **読み込み・確認・選択・絞り込み・結合・欠損処理・外れ値処理・集計・保存**できるようになりました。材料・高分子・石油化学・生化学、どの分野の表データも、同じ「分析の型」で扱えます。

### 次回予告

いよいよ **第5部：データ可視化**。第4部で作った集計結果を、matplotlib で**グラフ**にします。数字の表が、ひと目で伝わる図に変わります。まずは [第46回：matplotlib入門](lesson-46.md) から。
