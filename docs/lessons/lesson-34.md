# 第34回　フィルタリングと条件抽出

!!! abstract "この回のゴール"
    - 条件に合う**行だけ**を取り出す（フィルタリング）
    - `&`（かつ）・`|`（または）で複数条件を組み合わせる
    - 分析でいちばん使う操作を身につける
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性

`lesson34.py` を作り、まずデータを用意します。

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

## 1. 条件で行を絞る

「引張強さが 50 MPa を超えるポリマー」を取り出してみます。まず**条件**を書き、それを `df[ ... ]` の中に入れます。

```python
# 条件だけを見てみる（各行が True / False になる）
print(polymers["tensile_MPa"] > 50)
```

出力:

```text
0    False
1    False
2    False
3     True
4     True
5     True
6     True
7     True
8    False
Name: tensile_MPa, dtype: bool
```

この True / False の列を `df[ ... ]` に渡すと、**True の行だけ**が残ります。

```python
strong = polymers[polymers["tensile_MPa"] > 50]
print(strong)
```

出力:

```text
  polymer       category  density  tensile_MPa  Tg_C
3     PVC  thermoplastic     1.38           52    80
4     PET  thermoplastic     1.38           55    76
5    PMMA  thermoplastic     1.18           70   105
6  Nylon6  thermoplastic     1.14           80    50
7   Epoxy      thermoset     1.20           60   120
```

!!! note "仕組みはシンプル"
    「条件 → True/False の列 → True の行だけ残す」。この3段構えを頭に置くと、どんな条件でも書けます。

---

## 2. 文字列（カテゴリ）で絞る

数値だけでなく、文字列の一致 `==` でも絞れます。

```python
thermosets = polymers[polymers["category"] == "thermoset"]
print(thermosets)
```

出力:

```text
    polymer   category  density  tensile_MPa  Tg_C
7     Epoxy  thermoset      1.2           60   120
8  Phenolic  thermoset      1.3           50   180
```

---

## 3. 複数条件：& と |

条件を組み合わせるときは `&`（かつ）と `|`（または）を使い、**各条件を丸かっこ `( )` で囲みます**（これを忘れるとエラーになります）。

```python
# 熱可塑性 かつ ガラス転移点が100℃未満
result = polymers[(polymers["category"] == "thermoplastic") & (polymers["Tg_C"] < 100)]
print(result)
```

出力:

```text
  polymer       category  density  tensile_MPa  Tg_C
0      PE  thermoplastic    0.940           25  -110
1      PP  thermoplastic    0.905           35   -10
3     PVC  thermoplastic    1.380           52    80
4     PET  thermoplastic    1.380           55    76
6  Nylon6  thermoplastic    1.140           80    50
```

!!! warning "`and` ではなく `&`、そしてかっこ必須"
    pandas の条件では、Python の `and`/`or` ではなく **`&`/`|`** を使います。さらに **各条件を `( )` で囲む**必要があります。
    ```python
    # 正しい
    df[(df["a"] > 1) & (df["b"] < 5)]
    # よくある間違い（エラー）
    df[df["a"] > 1 and df["b"] < 5]
    ```

---

## 4. 絞ったあとに集計する

フィルタと集計を組み合わせると、「条件つきの平均」などがすぐ出せます。

```python
# 熱可塑性ポリマーの平均引張強さ
tp = polymers[polymers["category"] == "thermoplastic"]
print("熱可塑性の平均引張強さ:", tp["tensile_MPa"].mean().round(2), "MPa")
print("該当する件数:", len(tp), "件")
```

出力:

```text
熱可塑性の平均引張強さ: 51.71 MPa
該当する件数: 7 件
```

---

## 演習問題

**問1.** `density`（密度）が 1.2 以上のポリマーだけを取り出して表示してください。

**問2.** 「`category` が `thermoplastic`」**かつ**「`tensile_MPa` が 50 以上」のポリマーを取り出してください（`&` とかっこを使う）。

**問3.** 熱硬化性（thermoset）ポリマーの **平均ガラス転移点（Tg_C）** を計算して表示してください（まず絞ってから `.mean()`）。

---

## 解答

??? success "問1 の解答"
    ```python
    dense = polymers[polymers["density"] >= 1.2]
    print(dense)
    ```

    出力:
    ```text
        polymer       category  density  tensile_MPa  Tg_C
    3       PVC  thermoplastic     1.38           52    80
    4       PET  thermoplastic     1.38           55    76
    7     Epoxy      thermoset     1.20           60   120
    8  Phenolic      thermoset     1.30           50   180
    ```

??? success "問2 の解答"
    ```python
    result = polymers[(polymers["category"] == "thermoplastic") & (polymers["tensile_MPa"] >= 50)]
    print(result)
    ```

    出力:
    ```text
      polymer       category  density  tensile_MPa  Tg_C
    3     PVC  thermoplastic     1.38           52    80
    4     PET  thermoplastic     1.38           55    76
    5    PMMA  thermoplastic     1.18           70   105
    6  Nylon6  thermoplastic     1.14           80    50
    ```

??? success "問3 の解答"
    ```python
    ts = polymers[polymers["category"] == "thermoset"]
    print("熱硬化性の平均Tg:", ts["Tg_C"].mean(), "℃")
    ```

    出力:
    ```text
    熱硬化性の平均Tg: 150.0 ℃
    ```

    （Epoxy 120℃ と Phenolic 180℃ の平均で 150℃。）

---

## この回のまとめ

- 「条件 → True/False の列 → `df[条件]` で True の行だけ残す」
- 文字列は `==` で一致を判定して絞れる
- 複数条件は `&`（かつ）/ `|`（または）、**各条件を `( )` で囲む**（`and`/`or` は不可）
- 絞った表にそのまま `.mean()` などを使えば「条件つき集計」ができる

### 次回予告

[第35回：欠損値の扱い](lesson-35.md) では、実データにつきものの「空欄（NaN）」への対処を学びます。生化学の酵素アッセイデータを題材にします。
