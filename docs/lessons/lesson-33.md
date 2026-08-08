# 第33回　行と列の選択（loc / iloc）

!!! abstract "この回のゴール"
    - 特定の**列**を1つ／複数取り出す
    - `iloc`（位置で）と `loc`（ラベルで）で**行**を取り出す
    - 行と列を同時に指定して、表の一部を切り出す
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性（第32回と同じ）

`lesson33.py` を作りましょう。まず毎回のデータを用意します（今回もコピペで完結します）。

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

## 1. 列を取り出す

1列なら `df["列名"]`、複数列なら**リストで**渡します（角かっこが二重になる点に注意）。

```python
# 1列（Series）
print(polymers["tensile_MPa"].head(3))

# 複数列（DataFrame）→ [[ ]] と二重
print(polymers[["polymer", "tensile_MPa"]].head(4))
```

出力:

```text
0    25
1    35
2    45
Name: tensile_MPa, dtype: int64
  polymer  tensile_MPa
0      PE           25
1      PP           35
2      PS           45
3     PVC           52
```

!!! note "1列と複数列で「型」が違う"
    `df["列"]` は **Series**（1次元）、`df[["列"]]` は **DataFrame**（表のまま）。複数列を選ぶときは必ず `[[ ... ]]` の二重かっこです。

---

## 2. iloc：位置（番号）で行を選ぶ

`iloc` は「integer location」。**0から数えた位置**で行・列を指定します。Python のリストと同じ感覚です。

```python
print(polymers.iloc[0])        # 0番目の行（1件） → 縦向きに表示
print("---")
print(polymers.iloc[0:3, 0:2]) # 0〜2行、0〜1列の範囲
```

出力:

```text
polymer                   PE
category       thermoplastic
density                 0.94
tensile_MPa               25
Tg_C                    -110
Name: 0, dtype: object
---
  polymer       category
0      PE  thermoplastic
1      PP  thermoplastic
2      PS  thermoplastic
```

`iloc[行, 列]` の順で、範囲は `0:3`（0,1,2＝3は含まない）のように書きます。

---

## 3. loc：ラベル（名前）で行・列を選ぶ

`loc` は**ラベル（index や列名）**で指定します。今回の index は 0,1,2… の番号ですが、`loc` では**終端を含む**点が `iloc` と違います。

```python
# index 0〜2 の行、指定した列名だけ
print(polymers.loc[0:2, ["polymer", "Tg_C"]])
```

出力:

```text
  polymer  Tg_C
0      PE  -110
1      PP   -10
2      PS   100
```

!!! warning "`iloc` と `loc` の範囲の違い"
    - `iloc[0:3]` … 0,1,2（**3は含まない**、Python流）
    - `loc[0:2]` … 0,1,2（**2を含む**、ラベル流）

    最初は迷いますが、「**loc は名前で・終端を含む**」「**iloc は位置で・終端を含まない**」と覚えましょう。列名で選ぶときは `loc`、位置で手早く見るときは `iloc` が便利です。

---

## 4. よく使う組み合わせ

```python
# 特定の列の、特定の行だけ
print(polymers.loc[3, "polymer"])          # index3 の polymer → PVC

# 最後の3行
print(polymers.tail(3))

# ある列を基準に見たい列を並べる
print(polymers[["polymer", "category", "tensile_MPa"]].head(3))
```

出力:

```text
PVC
    polymer       category  density  tensile_MPa  Tg_C
6    Nylon6  thermoplastic     1.14           80    50
7     Epoxy      thermoset     1.20           60   120
8  Phenolic      thermoset     1.30           50   180
  polymer       category  tensile_MPa
0      PE  thermoplastic           25
1      PP  thermoplastic           35
2      PS  thermoplastic           45
```

---

## 演習問題

**問1.** `polymers` から、`polymer` と `density` の**2列だけ**を取り出して先頭5行を表示してください。

**問2.** `iloc` を使って、**最初の3行・最初の3列**を取り出して表示してください。

**問3.** `loc` を使って、index が 5〜8 の行について、`polymer` と `Tg_C` の2列を表示してください（index 8 まで含まれることを確認しましょう）。

---

## 解答

??? success "問1 の解答"
    ```python
    print(polymers[["polymer", "density"]].head(5))
    ```

    出力:
    ```text
      polymer  density
    0      PE    0.940
    1      PP    0.905
    2      PS    1.050
    3     PVC    1.380
    4     PET    1.380
    ```

??? success "問2 の解答"
    ```python
    print(polymers.iloc[0:3, 0:3])
    ```

    出力:
    ```text
      polymer       category  density
    0      PE  thermoplastic    0.940
    1      PP  thermoplastic    0.905
    2      PS  thermoplastic    1.050
    ```

??? success "問3 の解答"
    ```python
    print(polymers.loc[5:8, ["polymer", "Tg_C"]])
    ```

    出力:
    ```text
        polymer  Tg_C
    5      PMMA   105
    6    Nylon6    50
    7     Epoxy   120
    8  Phenolic   180
    ```

    `loc` は終端（8）を**含む**ので、PMMA〜Phenolic の4行が出ます。

---

## この回のまとめ

- 1列は `df["列"]`（Series）、複数列は `df[["列A","列B"]]`（DataFrame）。
- `iloc[行, 列]` … **位置**で選ぶ。範囲の終端は含まない。
- `loc[行, 列]` … **ラベル**で選ぶ。範囲の終端を含む。
- 「名前で選ぶなら loc、位置でサッと見るなら iloc」。

### 次回予告

[第34回：フィルタリングと条件抽出](lesson-34.md) では、「引張強さが50 MPaを超えるポリマーだけ」のように、**条件に合う行だけ**を取り出す方法を学びます。分析の核心です。
