# 第31回　pandas入門：DataFrame とは（材料データを表で扱う）

!!! abstract "この回のゴール"
    - データ分析の主役 **pandas** と **DataFrame（表）** を知る
    - 表を作り、形・列・型・要約統計をすばやく確認する
    - 1つの列（物性値）を取り出して平均・最大などを計算する
    - 所要時間の目安: 60分
    - 使うデータ：**材料科学**（代表的な金属酸化物の物性）

!!! info "第4部スタート"
    ここからは、これまでの Python 基礎を土台に、**実データの集計・テーブル・グラフ化**へ進みます。第4部は「表データの扱い」、続く第5部が「可視化」です。材料・高分子・石油化学・生化学のデータを題材にします。

`lesson31.py` を作って進めましょう（先頭が `(chem)` になっているか確認）。

---

## 1. pandas と DataFrame

**pandas** は表形式データを扱うためのライブラリです。Excel のシートを Python で自在に操るイメージ。中心となるのが **DataFrame**（行と列のある表）です。

慣例として `import pandas as pd` と略します。

```python
import pandas as pd

# 金属酸化物の物性データ（材料科学）
oxides = pd.DataFrame({
    "oxide":       ["Alumina", "Titania", "Silica", "Zinc oxide", "Magnesia", "Zirconia"],
    "formula":     ["Al2O3", "TiO2", "SiO2", "ZnO", "MgO", "ZrO2"],
    "density":     [3.95, 4.23, 2.65, 5.61, 3.58, 5.68],       # g/cm^3
    "melting_C":   [2072, 1843, 1713, 1974, 2852, 2715],       # 融点 [℃]
    "band_gap_eV": [8.8, 3.2, 9.0, 3.37, 7.8, 5.0],            # バンドギャップ [eV]
})

print(oxides)
```

出力:

```text
        oxide formula  density  melting_C  band_gap_eV
0     Alumina   Al2O3     3.95       2072         8.80
1     Titania    TiO2     4.23       1843         3.20
2      Silica    SiO2     2.65       1713         9.00
3  Zinc oxide     ZnO     5.61       1974         3.37
4    Magnesia     MgO     3.58       2852         7.80
5    Zirconia    ZrO2     5.68       2715         5.00
```

左端の `0,1,2,...` は **index（行番号）** で、pandas が自動でつけます。辞書のキー（`oxide`, `formula`…）が**列名**になります。

!!! note "DataFrame の作り方いろいろ"
    ここでは「列名: 値のリスト」の辞書から作りました。実際には CSV ファイルから読み込むことが多く、それは次回（第32回）で学びます。

---

## 2. まず全体像をつかむ4点セット

大きな表が来たら、いきなり中身を全部見るのではなく、**形・先頭・列・型**を確認するのが定石です。

```python
print(oxides.shape)      # (行数, 列数)
print(oxides.head(3))    # 先頭3行だけ見る
print(list(oxides.columns))   # 列名の一覧
print(oxides.dtypes)     # 各列のデータ型
```

出力:

```text
(6, 5)
     oxide formula  density  melting_C  band_gap_eV
0  Alumina   Al2O3     3.95       2072          8.8
1  Titania    TiO2     4.23       1843          3.2
2   Silica    SiO2     2.65       1713          9.0
['oxide', 'formula', 'density', 'melting_C', 'band_gap_eV']
oxide           object
formula         object
density        float64
melting_C        int64
band_gap_eV    float64
dtype: object
```

!!! tip "型（dtype）の読み方"
    - `object` … 文字列（名前や化学式）
    - `int64` … 整数（融点）
    - `float64` … 小数（密度・バンドギャップ）

    型が想定と違うと計算でつまずきます。最初に `dtypes` を見る癖をつけましょう。

---

## 3. describe()：要約統計を一発で

数値列の**件数・平均・標準偏差・最小最大・四分位数**をまとめて出します。データの"感触"をつかむのに最適です。

```python
print(oxides.describe())
```

出力:

```text
        density    melting_C  band_gap_eV
count  6.000000     6.000000     6.000000
mean   4.283333  2194.833333     6.195000
std    1.182128   473.760875     2.668661
min    2.650000  1713.000000     3.200000
25%    3.672500  1875.750000     3.777500
50%    4.090000  2023.000000     6.400000
75%    5.265000  2554.250000     8.550000
max    5.680000  2852.000000     9.000000
```

`50%` は中央値（メジアン）です。文字列の列（oxide, formula）は自動的に除かれ、数値列だけがまとめられます。

---

## 4. 1つの列を取り出して計算する

列は `df["列名"]` で取り出せます。取り出した列（**Series** といいます）には、平均などの計算がそのまま使えます。

```python
print(oxides["density"])              # 密度の列だけ
print("平均密度:", oxides["density"].mean().round(3))
print("最高融点:", oxides["melting_C"].max())
print("最低バンドギャップ:", oxides["band_gap_eV"].min())
```

出力:

```text
0    3.95
1    4.23
2    2.65
3    5.61
4    3.58
5    5.68
Name: density, dtype: float64
平均密度: 4.283
最高融点: 2852
最低バンドギャップ: 3.2
```

!!! note "よく使う集計メソッド"
    `.mean()`（平均）, `.sum()`（合計）, `.max()` / `.min()`, `.std()`（標準偏差）, `.count()`（件数）, `.median()`（中央値）。列に対してそのまま呼べます。

---

## 演習問題

**問1.** 本文の `oxides` を作り、(a) 表全体、(b) 形 `shape`、(c) 先頭2行 `head(2)` を表示してください。

**問2.** `melting_C`（融点）の列について、**平均・最大・最小**を表示してください。

**問3.** 新しい材料データを自分で1つ作ってみましょう。3種類の半導体材料について、`material`（名前）・`band_gap_eV`（バンドギャップ）の2列を持つ DataFrame を作り、`describe()` で要約を表示してください。例：Silicon 1.12、Germanium 0.67、GaAs 1.42。

---

## 解答

??? success "問1 の解答"
    ```python
    import pandas as pd

    oxides = pd.DataFrame({
        "oxide":       ["Alumina", "Titania", "Silica", "Zinc oxide", "Magnesia", "Zirconia"],
        "formula":     ["Al2O3", "TiO2", "SiO2", "ZnO", "MgO", "ZrO2"],
        "density":     [3.95, 4.23, 2.65, 5.61, 3.58, 5.68],
        "melting_C":   [2072, 1843, 1713, 1974, 2852, 2715],
        "band_gap_eV": [8.8, 3.2, 9.0, 3.37, 7.8, 5.0],
    })

    print(oxides)
    print(oxides.shape)
    print(oxides.head(2))
    ```

??? success "問2 の解答"
    ```python
    print("平均融点:", oxides["melting_C"].mean().round(1))
    print("最高融点:", oxides["melting_C"].max())
    print("最低融点:", oxides["melting_C"].min())
    ```

    出力:
    ```text
    平均融点: 2194.8
    最高融点: 2852
    最低融点: 1713
    ```

??? success "問3 の解答"
    ```python
    import pandas as pd

    semi = pd.DataFrame({
        "material":    ["Silicon", "Germanium", "GaAs"],
        "band_gap_eV": [1.12, 0.67, 1.42],
    })
    print(semi)
    print(semi.describe())
    ```

    出力:
    ```text
        material  band_gap_eV
    0    Silicon         1.12
    1  Germanium         0.67
    2       GaAs         1.42
           band_gap_eV
    count     3.000000
    mean      1.070000
    std       0.377492
    min       0.670000
    25%       0.895000
    50%       1.120000
    75%       1.270000
    max       1.420000
    ```

---

## この回のまとめ

- `import pandas as pd`。表は **DataFrame**、1列は **Series**。
- 大きな表はまず `shape` / `head()` / `columns` / `dtypes` で全体像を確認。
- `describe()` で要約統計を一発表示（数値列のみ）。
- 列は `df["列名"]` で取り出し、`.mean()` `.max()` などで集計できる。

### 次回予告

[第32回：CSVを読み込む・書き出す](lesson-32.md) では、実際のデータファイル（CSV）を pandas で読み書きします。高分子（ポリマー）の物性データを題材にします。
