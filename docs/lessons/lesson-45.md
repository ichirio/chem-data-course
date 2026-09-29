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
exp_clean = exp.dropna(subset=["yield_pct"])
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

Pd の平均が最も高い、と読み取れます。ただし、触媒ごとに温度条件の回数がそろっていない点に注意しましょう（欠損を除いた後、Pt だけ 110 ℃ が2回あります）。条件をそろえて比べるには、第39回の `pivot_table(index="catalyst", columns="temp_C", values="yield_pct")` で温度ごとに並べます。この表では 80 ℃・110 ℃ のどちらでも Pd > Pt > Ni の順になっています。また、各触媒 2〜3 回の測定では、平均の差がばらつきの範囲内かどうかまでは判断できません。

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

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    実験ノート `experiment.csv`（列: run [int], catalyst [Pd/Pt/Ni], temp_C [℃], yield_pct [%、未測定は空欄]）を pandas で分析してください。
    読み込み後に shape と列ごとの欠損数を表示し、yield_pct が欠損した行だけを除いてください（未測定を 0% とみなさないこと）。
    触媒ごとに「実施回数（run の数）」「収率を測れた回数」「平均収率」「最高収率」をまとめ、さらに触媒×温度の平均収率のピボットテーブルも作ってください。
    集計表は index（触媒名）付きで `catalyst_summary.csv` に保存してください。

!!! warning "AIの出力で確かめること"
    - 欠損を `fillna(0)` で埋めていないか。未測定を「収率 0%」として平均に入れると、Pd の平均が 81.5% から 54.3% に下がり、結論が逆転します。
    - 件数を数える列。`("run", "count")` は実施回数、`("yield_pct", "count")` は収率を測れた回数です。どちらを「n」と呼んでいるか確認します。
    - 触媒の比較で、温度条件の回数がそろっているか（`pd.crosstab(df["catalyst"], df["temp_C"])`）。そろっていないなら、触媒×温度のピボットで条件をそろえて比べます。
    - 収率が 0〜100% の範囲に収まっているか（`df["yield_pct"].describe()`）。100% を超える値は、入力ミスか集計ミスです。
    - 保存した CSV を読み直して、触媒名の列が残っているか確認します（集計表を `index=False` で保存すると触媒名が消えます）。

---

## 演習問題

**問1.** 本文の `exp` を作り、`shape` と `isna().sum()` を表示して、どこに欠損があるか確認してください。

**問2.** 欠損行を除いたあと、**温度（temp_C）ごと**の平均収率を `groupby` で計算して表示してください（触媒ではなく温度でグループ化）。

**問3.** 欠損行を除いたデータで、`agg` を使って触媒ごとに「試行回数 `runs`」「平均収率 `yield_mean`」「収率の標準偏差 `yield_std`」の集計テーブルを作り、`summary2.csv` に保存してください。

**問4.**（検証型）次は AI が書いた「触媒ごとの成績表」のコードとその出力です。AI は「Pd は平均収率が 54.3% で最も低い」と報告しました。問題点を指摘し、直してください。

```python
exp["yield_pct"] = exp["yield_pct"].fillna(0)
summary = exp.groupby("catalyst").agg(
    runs=("run", "count"),
    yield_mean=("yield_pct", "mean"),
).round(1)
print(summary)
```

出力:
```text
          runs  yield_mean
catalyst                  
Ni           2        62.0
Pd           3        54.3
Pt           3        77.3
```

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
    exp_clean = exp.dropna(subset=["yield_pct"])
    print(exp_clean.groupby("temp_C")["yield_pct"].mean().round(2))
    ```

    出力:
    ```text
    temp_C
    80     68.67
    110    78.25
    Name: yield_pct, dtype: float64
    ```

    高温（110℃）の方が平均収率が高い傾向が見えます。ただし 110℃ の4件には Pt が2件含まれ、80℃ の3件とは触媒の構成が違います。同じ触媒どうしで比べても（Pd 78→85、Pt 70→81、Ni 58→66）110℃ の方が高いので、傾向は確かそうです。

??? success "問3 の解答"
    ```python
    exp_clean = exp.dropna(subset=["yield_pct"])
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

    Ni と Pd は2回ずつしか測定していないので、この標準偏差はあくまで目安です。

??? success "問4 の解答"
    **誤り**：未測定（run 7）の欠損を `fillna(0)` で埋めたため、「収率 0% の実験」として平均に入りました。測っていないことと、生成物が得られなかった（収率 0%）ことは、化学的にまったく別の意味です。その結果、Pd の平均が (78 + 85 + 0) / 3 = 54.3% に下がり、結論が逆転しています。

    **確かめ方**：`isna().sum()` で欠損の有無を確認し、`runs`（実施回数）とは別に `("yield_pct", "count")`（測れた回数）を並べます。0% という収率が出てきたら、実験ノートで「本当に 0% だったのか」を確かめます。

    ```python
    exp = pd.DataFrame({
        "run":       [1, 2, 3, 4, 5, 6, 7, 8],
        "catalyst":  ["Pd", "Pd", "Pt", "Pt", "Ni", "Ni", "Pd", "Pt"],
        "temp_C":    [80, 110, 80, 110, 80, 110, 80, 110],
        "yield_pct": [78, 85, 70, 80, 58, 66, None, 82],
    })
    summary = exp.groupby("catalyst").agg(
        runs=("run", "count"),
        measured=("yield_pct", "count"),
        yield_mean=("yield_pct", "mean"),
    ).round(1)
    print(summary)
    ```

    出力:
    ```text
              runs  measured  yield_mean
    catalyst                            
    Ni           2         2        62.0
    Pd           3         2        81.5
    Pt           3         3        77.3
    ```

    `mean` は NaN を自動で除くので、欠損を埋めなくても「測れた2回」の平均 81.5% になります。Pd は3回実施して2回測定できた、と表から読み取れます。

---

## 第4部　修了

おめでとうございます！ これで化学データを **読み込み・確認・選択・絞り込み・結合・欠損処理・外れ値処理・集計・保存**できるようになりました。材料・高分子・石油化学・生化学、どの分野の表データも、同じ「分析の型」で扱えます。

### 次回予告

いよいよ **第5部：データ可視化**。第4部で作った集計結果を、matplotlib で**グラフ**にします。数字の表が、ひと目で伝わる図に変わります。まずは [第46回：matplotlib入門](lesson-46.md) から。
