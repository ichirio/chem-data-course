# 第36回　並べ替えと集計（groupby）

!!! abstract "この回のゴール"
    - `sort_values()` で表を並べ替える
    - `groupby()` で**グループごとの集計**（平均・件数など）を出す
    - `agg()` で複数の集計を1つの表にまとめる
    - レポートに載せる「集計テーブル」を作れるようになる
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性

!!! success "第4部の山場"
    「カテゴリごとに平均を出して表にまとめる」——これは実験レポートや論文の**集計テーブル**そのものです。今回でデータ集計の中心が身につきます。

`lesson36.py` を作り、データを用意します。

```python
import pandas as pd

polymers = pd.DataFrame({
    "polymer":     ["PE", "PP", "PS", "PVC", "PET", "PMMA", "Nylon6", "Epoxy", "Phenolic"],
    "category":    ["thermoplastic"]*7 + ["thermoset"]*2,
    "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],
    "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
    "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],
})
```

---

## 1. 並べ替え：sort_values()

指定した列で表を並べ替えます。`ascending=False` で降順（大きい順）。

```python
# 引張強さの大きい順に並べ、上位3件
top3 = polymers.sort_values("tensile_MPa", ascending=False).head(3)
print(top3)
```

出力:

```text
  polymer       category  density  tensile_MPa  Tg_C
6  Nylon6  thermoplastic     1.14           80    50
5    PMMA  thermoplastic     1.18           70   105
7   Epoxy      thermoset     1.20           60   120
```

「一番強いポリマーは？」がすぐ分かります。`sort_values("Tg_C")` のように列を変えれば、別の基準でも並びます。

---

## 2. groupby：グループごとに集計する

`groupby("列")` は「その列の値が同じものを、グループにまとめる」操作です。ここでは `category`（熱可塑性／熱硬化性）でまとめて、引張強さの平均を出します。

```python
mean_tensile = polymers.groupby("category")["tensile_MPa"].mean().round(2)
print(mean_tensile)
```

出力:

```text
category
thermoplastic    51.71
thermoset        55.00
Name: tensile_MPa, dtype: float64
```

たった1行で「カテゴリ別の平均引張強さ」が出ました。読み方は **「グループにして（groupby）→ 見たい列を選び → 集計（mean）」** です。

!!! note "集計はいろいろ選べる"
    `.mean()` を `.max()` / `.min()` / `.sum()` / `.count()` / `.std()` に替えれば、そのグループ集計になります。
    ```python
    print(polymers.groupby("category")["Tg_C"].max())   # カテゴリ別の最高Tg
    ```

---

## 3. agg：複数の集計を1つの表に

レポートでは「件数・平均・…」を**まとめて1つの表**にしたいことが多いです。`agg()` を使うと、列ごとに好きな集計を指定できます。

```python
summary = polymers.groupby("category").agg(
    n=("polymer", "count"),            # 件数
    tensile_mean=("tensile_MPa", "mean"),
    density_mean=("density", "mean"),
).round(3)

print(summary)
```

出力:

```text
               n  tensile_mean  density_mean
category                                    
thermoplastic  7        51.714         1.139
thermoset      2        55.000         1.250
```

これはもう、**そのままレポートに載せられる集計テーブル**です。`("元の列名", "集計方法")` の形で、新しい列名（`n`, `tensile_mean`…）を左に書きます。

!!! tip "この表を CSV にすれば完成"
    ```python
    summary.to_csv("polymer_summary.csv")
    ```
    集計テーブルをファイルに保存して、レポートや Excel に貼れます。第9部では、この表を Quarto でレポート文書に組み込みます。

---

## 4. 実務の流れ：読む → 絞る → 集計 → 保存

第4部で学んだことをつなげると、分析の基本フローができあがります。

```python
# ① 絞る（熱可塑性だけ）
tp = polymers[polymers["category"] == "thermoplastic"]

# ② 並べ替え（引張強さ順）
tp_sorted = tp.sort_values("tensile_MPa", ascending=False)

# ③ 集計（平均）
print("熱可塑性の平均引張強さ:", tp["tensile_MPa"].mean().round(2), "MPa")

# ④ 保存
tp_sorted.to_csv("thermoplastics_sorted.csv", index=False)
print("保存しました")
```

出力:

```text
熱可塑性の平均引張強さ: 51.71 MPa
保存しました
```

---

## 演習問題

**問1.** `polymers` を `density`（密度）の**小さい順**に並べ替えて、全体を表示してください。

**問2.** `groupby` を使って、カテゴリ別の **ガラス転移点（Tg_C）の平均**を表示してください。

**問3.** `agg` を使って、カテゴリ別に「件数 `n`」「Tg_C の平均 `Tg_mean`」「Tg_C の最大 `Tg_max`」をまとめた集計テーブルを作って表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    print(polymers.sort_values("density"))
    ```

    出力:
    ```text
        polymer       category  density  tensile_MPa  Tg_C
    1        PP  thermoplastic    0.905           35   -10
    0        PE  thermoplastic    0.940           25  -110
    2        PS  thermoplastic    1.050           45   100
    6    Nylon6  thermoplastic    1.140           80    50
    5      PMMA  thermoplastic    1.180           70   105
    7     Epoxy      thermoset    1.200           60   120
    8  Phenolic      thermoset    1.300           50   180
    3       PVC  thermoplastic    1.380           52    80
    4       PET  thermoplastic    1.380           55    76
    ```

??? success "問2 の解答"
    ```python
    print(polymers.groupby("category")["Tg_C"].mean().round(2))
    ```

    出力:
    ```text
    category
    thermoplastic     41.57
    thermoset        150.00
    Name: Tg_C, dtype: float64
    ```

??? success "問3 の解答"
    ```python
    table = polymers.groupby("category").agg(
        n=("polymer", "count"),
        Tg_mean=("Tg_C", "mean"),
        Tg_max=("Tg_C", "max"),
    ).round(2)
    print(table)
    ```

    出力:
    ```text
                   n  Tg_mean  Tg_max
    category                         
    thermoplastic  7    41.57     105
    thermoset      2   150.00     180
    ```

---

## この回のまとめ

- `sort_values("列", ascending=False)` で並べ替え（降順は False）。
- `groupby("列")["対象列"].mean()` で**グループ別集計**。
- `groupby(...).agg(新名=("列","集計"))` で複数集計を1つの表に。
- 集計テーブルは `to_csv()` で保存してレポートに活用。
- 分析の基本フロー：**読む → 絞る → 集計 → 保存**。

### 第4部（前半）修了

これで、化学データを**読み込み・確認・選択・絞り込み・欠損処理・集計**できるようになりました。材料・高分子・生化学・石油化学、どの分野の表データでも同じ手順で扱えます。

### 次回予告

次は **第5部：データ可視化**へ。集計した結果を、matplotlib で**グラフ**にします。数字の表が、ひと目で分かる図に変わります。（※第4部の残り〈結合・ピボット・Excel入出力など〉と第5部は、続くバッチで作成します。）
