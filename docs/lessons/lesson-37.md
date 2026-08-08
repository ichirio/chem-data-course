# 第37回　列の追加・計算・変換

!!! abstract "この回のゴール"
    - 既存の列から**新しい列を計算して追加**する
    - 単位換算など列全体をまとめて変換する
    - `apply()` で「条件による分類」列を作る
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**

`lesson37.py` を作り、データを用意します。

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

## 1. 列どうしの計算で新しい列を作る

列は**まとめて計算**できます（1行ずつ書く必要はありません）。比強度（引張強さ÷密度）と、ガラス転移点のケルビン換算を追加してみましょう。

```python
polymers["specific_strength"] = (polymers["tensile_MPa"] / polymers["density"]).round(1)
polymers["Tg_K"] = polymers["Tg_C"] + 273.15

print(polymers[["polymer", "tensile_MPa", "density", "specific_strength", "Tg_K"]].head(4))
```

出力:

```text
  polymer  tensile_MPa  density  specific_strength    Tg_K
0      PE           25    0.940               26.6  163.15
1      PP           35    0.905               38.7  263.15
2      PS           45    1.050               42.9  373.15
3     PVC           52    1.380               37.7  353.15
```

`df["新しい列"] = 計算式` と書くだけで、全行に一括で計算されます。これが pandas の強力さです。

---

## 2. apply()：条件で分類する列を作る

「引張強さが 55 MPa 以上なら high、そうでなければ standard」のような**判定つきの列**は、`apply()` に関数を渡して作ります。手軽な**ラムダ式**（第14回で詳説）を使います。

```python
polymers["grade"] = polymers["tensile_MPa"].apply(lambda x: "high" if x >= 55 else "standard")

print(polymers[["polymer", "tensile_MPa", "grade"]])
```

出力:

```text
    polymer  tensile_MPa     grade
0        PE           25  standard
1        PP           35  standard
2        PS           45  standard
3       PVC           52  standard
4       PET           55      high
5      PMMA           70      high
6    Nylon6           80      high
7     Epoxy           60      high
8  Phenolic           50  standard
```

!!! note "`lambda x: ...` の読み方"
    `lambda x: "high" if x >= 55 else "standard"` は「値 x を受け取り、55以上なら high、でなければ standard を返す」使い捨ての小さな関数です。各行の値が順に x に入ります。

---

## 3. 列名の変更

分かりやすい名前に変えるには `rename` を使います。

```python
renamed = polymers.rename(columns={"Tg_C": "glass_transition_C"})

print("Tg_C" in renamed.columns)                 # 変更後の表 → False
print("glass_transition_C" in renamed.columns)   # 変更後の表 → True
print("元の polymers は無事:", "Tg_C" in polymers.columns)   # → True
```

出力:

```text
False
True
元の polymers は無事: True
```

!!! warning "多くのメソッドは「新しい表を返す」"
    `rename`・`sort_values`・`dropna` などは、元を書き換えず**新しい結果を返します**。上のように `renamed` の方だけが変わり、元の `polymers` はそのままです。変更を残したいときは `df = df.rename(...)` のように**受け取り直す**か、結果を別の変数に入れて使いましょう。

---

## 演習問題

**問1.** `polymers` に、密度を kg/m³ に直した列 `density_kg_m3` を追加してください（g/cm³ × 1000）。先頭5行を表示しましょう。

**問2.** `apply` を使って、`Tg_C` が 100 以上なら `"high-Tg"`、未満なら `"low-Tg"` とする列 `tg_class` を追加し、`polymer` と一緒に表示してください。

**問3.** 列 `tensile_MPa` を `tensile_strength_MPa` に変更した新しい表を `rename` で作り、列名一覧を表示してください（元の `polymers` は変えないこと）。

---

## 解答

??? success "問1 の解答"
    ```python
    polymers["density_kg_m3"] = polymers["density"] * 1000
    print(polymers[["polymer", "density", "density_kg_m3"]].head(5))
    ```

    出力:
    ```text
      polymer  density  density_kg_m3
    0      PE    0.940          940.0
    1      PP    0.905          905.0
    2      PS    1.050         1050.0
    3     PVC    1.380         1380.0
    4     PET    1.380         1380.0
    ```

??? success "問2 の解答"
    ```python
    polymers["tg_class"] = polymers["Tg_C"].apply(lambda x: "high-Tg" if x >= 100 else "low-Tg")
    print(polymers[["polymer", "Tg_C", "tg_class"]])
    ```

    出力:
    ```text
        polymer  Tg_C tg_class
    0        PE  -110   low-Tg
    1        PP   -10   low-Tg
    2        PS   100  high-Tg
    3       PVC    80   low-Tg
    4       PET    76   low-Tg
    5      PMMA   105  high-Tg
    6    Nylon6    50   low-Tg
    7     Epoxy   120  high-Tg
    8  Phenolic   180  high-Tg
    ```

??? success "問3 の解答"
    ```python
    renamed = polymers.rename(columns={"tensile_MPa": "tensile_strength_MPa"})
    print(list(renamed.columns))
    print("元は変わらない:", "tensile_MPa" in polymers.columns)
    ```

---

## この回のまとめ

- `df["新列"] = 列を使った計算式` で、全行に一括計算して列を追加。
- `df["列"].apply(lambda x: ...)` で、条件による分類列を作る。
- `rename(columns={...})` で列名変更。
- 多くのメソッドは**新しい表を返す**（元は変わらない）。受け取り直すこと。

### 次回予告

[第38回：複数データの結合（merge / concat）](lesson-38.md) では、別々の表を1つにつなげる方法を学びます。物性表に価格表を突き合わせる、といった実務そのものです。
