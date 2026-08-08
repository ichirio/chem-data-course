# 第41回　実験データの前処理レシピ

!!! abstract "この回のゴール"
    - 現実の「汚れたデータ」によくある問題を知る
    - 余分な空白・表記ゆれ・数値でない値を、順を追ってきれいにする
    - 「前処理の定石」を1つの流れとして身につける
    - 所要時間の目安: 60分
    - 使うデータ：**実験記録**（表記がバラバラなサンプル表）

実データは、人が手入力するため必ず"汚れて"います。分析の前に整える**前処理**が、実は分析時間の大半を占めます。

```python
import pandas as pd

raw = pd.DataFrame({
    "Sample ": [" S1", "S2 ", " s3", "S4"],           # 前後に空白、大文字小文字ゆれ
    "Solvent": ["Water", "water ", "ETHANOL", "Ethanol"],  # 表記ゆれ
    "yield":   ["88.5", "91.2", "not measured", "85.7"],   # 数値でない値が混入
})
print(raw)
```

出力:

```text
  Sample   Solvent         yield
0      S1    Water          88.5
1     S2    water           91.2
2      s3  ETHANOL  not measured
3      S4  Ethanol          85.7
```

一見それっぽいですが、問題だらけです。列名に空白、値に前後の空白、`Water/water`の表記ゆれ、収率に文字列。

---

## レシピ1　列名を整える

列名の前後の空白を取り、小文字に統一します。

```python
df = raw.copy()
df.columns = df.columns.str.strip().str.lower()
print(list(df.columns))
```

出力:

```text
['sample', 'solvent', 'yield']
```

`Sample ` の余分な空白が消え、すべて小文字になりました。列名がきれいだと、以降の指定でミスが減ります。

---

## レシピ2　文字列の空白・大文字小文字をそろえる

`.str` を使うと、列の全文字列にまとめて処理できます。

```python
df["sample"] = df["sample"].str.strip().str.upper()    # 前後空白を除き、大文字に
df["solvent"] = df["solvent"].str.strip().str.title()  # 前後空白を除き、先頭大文字に
print(df)
```

出力:

```text
  sample  solvent         yield
0     S1    Water          88.5
1     S2    Water          91.2
2     S3  Ethanol  not measured
3     S4  Ethanol          85.7
```

`s3 → S3`、`water /ETHANOL → Water/Ethanol` と表記が統一されました。これで「同じ溶媒なのに別物扱い」を防げます。

!!! note "文字列メソッドは `.str` を通す"
    列（Series）に対しては、`.str.strip()` `.str.upper()` `.str.title()` `.str.replace()` のように **`.str` を挟んで**呼びます（第9回で学んだ文字列メソッドの列版です）。

---

## レシピ3　数値でない値を数値に（変換不能は NaN）

`yield` 列には `"not measured"` が混じっています。`pd.to_numeric(..., errors="coerce")` を使うと、**数値に直せない値を自動で NaN** にしてくれます。

```python
df["yield"] = pd.to_numeric(df["yield"], errors="coerce")
print(df)
print(df.dtypes)
```

出力:

```text
  sample  solvent  yield
0     S1    Water   88.5
1     S2    Water   91.2
2     S3  Ethanol    NaN
3     S4  Ethanol   85.7
sample      object
solvent     object
yield      float64
dtype: object
```

`"not measured"` が `NaN` になり、`yield` が計算できる `float64` 型になりました。あとは第35回の欠損処理（dropna / fillna）につなげられます。

!!! success "前処理の定石"
    **① 列名を整える → ② 文字列の表記をそろえる → ③ 型を正しくする（数値化）→ ④ 欠損を処理する**。
    この順番を覚えておけば、たいていの汚れたデータに立ち向かえます。

---

## 演習問題

**問1.** 本文の `raw` を作り、レシピ1で列名を整えた後の列名一覧を表示してください。

**問2.** レシピ2まで行い、`solvent` 列に何種類の溶媒があるか `nunique()` で数えてください（表記をそろえた結果、2種類になるはずです）。

**問3.** レシピ3まで行い、`yield` 列の**平均**（NaN は自動でスキップ）を小数第2位まで表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    df = raw.copy()
    df.columns = df.columns.str.strip().str.lower()
    print(list(df.columns))
    ```

    出力:
    ```text
    ['sample', 'solvent', 'yield']
    ```

??? success "問2 の解答"
    ```python
    df["solvent"] = df["solvent"].str.strip().str.title()
    print("溶媒の種類数:", df["solvent"].nunique())
    print(df["solvent"].unique())
    ```

    出力:
    ```text
    溶媒の種類数: 2
    ['Water' 'Ethanol']
    ```

??? success "問3 の解答"
    ```python
    df["yield"] = pd.to_numeric(df["yield"], errors="coerce")
    print("平均収率:", round(df["yield"].mean(), 2))
    ```

    出力:
    ```text
    平均収率: 88.47
    ```

    （88.5, 91.2, 85.7 の平均。NaN の行は除かれています。）

---

## この回のまとめ

- 実データは必ず汚れている。分析前の**前処理**が肝心。
- 列名は `df.columns.str.strip().str.lower()` で整える。
- 文字列列は `.str.strip()` `.str.upper()` `.str.title()` で表記統一。
- `pd.to_numeric(..., errors="coerce")` で数値化（不能値は NaN）。
- 定石：**列名 → 表記 → 型 → 欠損**の順。

### 次回予告

[第42回：外れ値の検出](lesson-42.md) では、測定ミスなどで紛れ込む「明らかにおかしい値」を、統計的な基準で見つける方法を学びます。
