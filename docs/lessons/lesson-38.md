# 第38回　複数データの結合（merge / concat）

!!! abstract "この回のゴール"
    - `merge()` で、共通の列（キー）を手がかりに**2つの表を横に突き合わせる**
    - `how` で結合の種類（inner / left）を選ぶ
    - `concat()` で表を**縦につなげる**（データの追加）
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性表 ＋ 生産者・価格表

現実のデータは、複数の表に分かれていることがほとんどです（物性は測定担当、価格は購買担当…）。それらを1つにまとめるのが結合です。

```python
import pandas as pd

# 物性の表
polymers = pd.DataFrame({
    "polymer":     ["PE", "PP", "PS", "PVC", "PET", "PMMA", "Nylon6", "Epoxy", "Phenolic"],
    "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
})

# 生産者・価格の表（一部のポリマーだけ）
info = pd.DataFrame({
    "polymer":      ["PE", "PP", "PS", "PVC", "PET"],
    "producer":     ["A-chem", "B-poly", "C-mat", "A-chem", "D-fiber"],
    "price_yen_kg": [180, 210, 250, 160, 300],
})
```

---

## 1. merge：共通の列で横につなぐ

両方の表にある `polymer` 列を**キー**にして、横に突き合わせます。

```python
merged = pd.merge(polymers, info, on="polymer")
print(merged)
```

出力:

```text
  polymer  tensile_MPa producer  price_yen_kg
0      PE           25   A-chem           180
1      PP           35   B-poly           210
2      PS           45    C-mat           250
3     PVC           52   A-chem           160
4     PET           55  D-fiber           300
```

`on="polymer"` で「polymer 列が一致する行どうしを合体」します。物性と価格が1つの表になりました。

!!! note "既定は inner（両方にある行だけ）"
    `info` には5種類しかないので、結果も5行だけ。**両方の表に存在するキーの行だけ**が残る（＝inner join）のが既定の動きです。

---

## 2. how：結合の種類を選ぶ

「物性表は全部残したい。価格が無いものは空欄でよい」——そんなときは `how="left"`（左の表を全部残す）を使います。

```python
merged_all = pd.merge(polymers, info, on="polymer", how="left")
print(merged_all)
```

出力:

```text
    polymer  tensile_MPa producer  price_yen_kg
0        PE           25   A-chem         180.0
1        PP           35   B-poly         210.0
2        PS           45    C-mat         250.0
3       PVC           52   A-chem         160.0
4       PET           55  D-fiber         300.0
5      PMMA           70      NaN           NaN
6    Nylon6           80      NaN           NaN
7     Epoxy           60      NaN           NaN
8  Phenolic           50      NaN           NaN
```

左（polymers）の9行が全部残り、価格表に無かった PMMA 以降は `NaN`（欠損）になりました。欠損の扱いは第35回のとおりです。

| how | 意味 |
|---|---|
| `"inner"`（既定） | 両方にあるキーだけ |
| `"left"` | 左の表を全部残す |
| `"right"` | 右の表を全部残す |
| `"outer"` | どちらかにあれば残す |

---

## 3. concat：縦につなげる（データの追加）

同じ列を持つ表を**縦に積む**のが `concat` です。別々に測定したバッチを1つにまとめる、といった場面で使います。

```python
batch1 = pd.DataFrame({"sample": ["S1", "S2"], "yield_pct": [88.5, 91.2]})
batch2 = pd.DataFrame({"sample": ["S3", "S4"], "yield_pct": [90.0, 85.7]})

combined = pd.concat([batch1, batch2], ignore_index=True)
print(combined)
```

出力:

```text
  sample  yield_pct
0     S1       88.5
1     S2       91.2
2     S3       90.0
3     S4       85.7
```

!!! tip "`ignore_index=True` を付ける"
    付けないと、元の index（0,1 と 0,1）がそのまま残って重複します。縦結合では番号を振り直す `ignore_index=True` が定番です。

!!! note "merge と concat の使い分け"
    - **merge** … 別の情報を**横に**足す（物性 ＋ 価格）。キー列で対応づける。
    - **concat** … 同じ形の表を**縦に**積む（バッチ1 ＋ バッチ2）。行を増やす。

---

## 演習問題

**問1.** `polymers` と `info` を `merge`（既定の inner）して、結合後の**行数**を `len()` で表示してください。なぜその行数になるか考えましょう。

**問2.** `how="left"` で結合した表から、`producer` が欠損している（価格情報が無い）ポリマーだけを取り出して表示してください（ヒント：`merged_all["producer"].isna()`）。

**問3.** 3つ目のバッチ `batch3 = {"sample": ["S5", "S6"], "yield_pct": [92.1, 89.9]}` を作り、`batch1`・`batch2`・`batch3` を `concat` で縦に1つにまとめて表示してください（`ignore_index=True`）。

---

## 解答

??? success "問1 の解答"
    ```python
    merged = pd.merge(polymers, info, on="polymer")
    print("行数:", len(merged))
    ```

    出力:
    ```text
    行数: 5
    ```
    `info` に5種類しかなく、inner は両方にあるキーだけ残すため、5行になります。

??? success "問2 の解答"
    ```python
    merged_all = pd.merge(polymers, info, on="polymer", how="left")
    no_price = merged_all[merged_all["producer"].isna()]
    print(no_price[["polymer", "tensile_MPa"]])
    ```

    出力:
    ```text
        polymer  tensile_MPa
    5      PMMA           70
    6    Nylon6           80
    7     Epoxy           60
    8  Phenolic           50
    ```

??? success "問3 の解答"
    ```python
    batch1 = pd.DataFrame({"sample": ["S1", "S2"], "yield_pct": [88.5, 91.2]})
    batch2 = pd.DataFrame({"sample": ["S3", "S4"], "yield_pct": [90.0, 85.7]})
    batch3 = pd.DataFrame({"sample": ["S5", "S6"], "yield_pct": [92.1, 89.9]})

    all_batches = pd.concat([batch1, batch2, batch3], ignore_index=True)
    print(all_batches)
    ```

    出力:
    ```text
      sample  yield_pct
    0     S1       88.5
    1     S2       91.2
    2     S3       90.0
    3     S4       85.7
    4     S5       92.1
    5     S6       89.9
    ```

---

## この回のまとめ

- `pd.merge(a, b, on="キー列")` … 共通の列で**横に**突き合わせる。
- `how=` で inner（既定）/ left / right / outer を選ぶ。無い所は NaN。
- `pd.concat([a, b], ignore_index=True)` … 同じ形の表を**縦に**積む。
- 「横に情報を足す＝merge」「縦に行を足す＝concat」。

### 次回予告

[第39回：ピボットテーブルとクロス集計](lesson-39.md) では、「触媒×温度ごとの収率」のような**2軸の集計表**を作ります。Excelのピボットテーブルと同じことを pandas で行います。
