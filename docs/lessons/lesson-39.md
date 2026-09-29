# 第39回　ピボットテーブルとクロス集計

!!! abstract "この回のゴール"
    - `pivot_table()` で「2つの軸 × 値」の集計表を作る
    - Excel のピボットテーブルと同じ発想を pandas で
    - `crosstab()` で件数のクロス集計を作る
    - 所要時間の目安: 60分
    - 使うデータ：**触媒反応**（架空データ：ニトロベンゼンの水素化でアニリンを得る反応を想定し、触媒 × 温度ごとの収率）

`lesson39.py` を作り、反応データを用意します。Pd・Pt・Ni はいずれも担持金属触媒（例：Pd/C）を想定した架空の値です。「縦長」のデータ（1行＝1条件）から始めます。

```python
import pandas as pd

rxn = pd.DataFrame({
    "catalyst":  ["Pd", "Pd", "Pd", "Pt", "Pt", "Pt", "Ni", "Ni", "Ni"],
    "temp_C":    [50, 80, 110, 50, 80, 110, 50, 80, 110],
    "yield_pct": [62, 78, 85, 55, 70, 80, 40, 58, 66],
})
print(rxn)
```

出力:

```text
  catalyst  temp_C  yield_pct
0       Pd      50         62
1       Pd      80         78
2       Pd     110         85
3       Pt      50         55
4       Pt      80         70
5       Pt     110         80
6       Ni      50         40
7       Ni      80         58
8       Ni     110         66
```

この縦長データは正確ですが、「触媒ごと・温度ごとに一覧で比べたい」ときは見づらいですね。

---

## 1. pivot_table：2軸の集計表にする

「行＝触媒、列＝温度、中身＝収率」の表に組み替えます。

```python
pivot = rxn.pivot_table(index="catalyst", columns="temp_C", values="yield_pct")
print(pivot)
```

出力:

```text
temp_C     50    80    110
catalyst                  
Ni        40.0  58.0  66.0
Pd        62.0  78.0  85.0
Pt        55.0  70.0  80.0
```

一気に見やすくなりました。**「どの触媒が・どの温度で・何%か」がひと目**で分かります。これが実験レポートに載る典型的な表です。

!!! note "3つの引数の意味"
    - `index` … 縦軸にする列（触媒）
    - `columns` … 横軸にする列（温度）
    - `values` … マスの中身にする値（収率）

---

## 2. 集計方法を指定する（aggfunc）

同じ条件の測定が複数あるとき、`pivot_table` は既定で**平均**を取ります。`aggfunc` で `max` や `sum` などに変えられます。

```python
# 触媒ごとの「最高収率」（温度をまたいだ最大値）を1列で
best = rxn.pivot_table(index="catalyst", values="yield_pct", aggfunc="max")
print(best)
```

出力:

```text
          yield_pct
catalyst           
Ni               66
Pd               85
Pt               80
```

`aggfunc="mean"`（既定）/ `"max"` / `"min"` / `"sum"` / `"count"` などが使えます。

---

## 3. crosstab：件数のクロス集計

「どの条件が何回測定されたか」のように、**件数**を数えたいときは `crosstab` が手軽です。

```python
print(pd.crosstab(rxn["catalyst"], rxn["temp_C"]))
```

出力:

```text
temp_C    50   80   110
catalyst               
Ni          1    1    1
Pd          1    1    1
Pt          1    1    1
```

各条件が1回ずつ測定されている、と分かります（測定の抜け漏れチェックにも便利）。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    pandas の縦長データ `rxn2`（列: catalyst [str], temp_C [℃], yield_pct [%]、同じ条件の繰り返し測定を含む）から、
    行＝触媒、列＝温度、中身＝収率の**平均**のピボットテーブルを作ってください（`pivot_table`, `aggfunc="mean"`）。
    あわせて `pd.crosstab` で各条件の測定回数の表も作り、2つを並べて表示してください。

!!! warning "AIの出力で確かめること"
    - `aggfunc` が目的どおりか。収率に `"sum"` を使うと、繰り返し測定が足し合わされて 100% を超える値（例：Pd・80 ℃ で 154%）が出ます。収率・濃度・温度のような「量でない値」は平均（mean）が基本です。
    - 平均値のマスが「何回の測定の平均か」を `crosstab` で確認します。1回だけの値と2回の平均が同じ表に混じっていることを把握します。
    - 空のマスは `NaN` です（その条件は未測定）。AI が `fill_value=0` で 0 にしていたら、「収率 0%」と誤読されるので注意します。
    - index と columns が指示どおりか（行＝触媒、列＝温度）。入れ替わっていないか、表の左上の見出し（`temp_C` / `catalyst`）で確かめます。

---

## 演習問題

**問1.** `rxn` から「行＝温度、列＝触媒、中身＝収率」のピボットテーブルを作って表示してください（`index` と `columns` を入れ替える）。

**問2.** `pivot_table` で、触媒ごとの**平均収率**（温度をまたいだ平均）を1列で表示してください（`values="yield_pct"`, `aggfunc="mean"`）。

**問3.** 次の拡張データで、触媒×温度の**測定回数**を `crosstab` で数えてください。Pd の 80℃ が2回になっているはずです。
```python
rxn2 = pd.DataFrame({
    "catalyst":  ["Pd", "Pd", "Pd", "Pt", "Ni", "Pd"],
    "temp_C":    [50, 80, 110, 80, 80, 80],
    "yield_pct": [62, 78, 85, 70, 58, 76],
})
```

**問4.**（検証型）次は AI が問3の `rxn2` から「触媒×温度の収率の表」を作ったコードとその出力です。問題点を指摘し、直してください。

```python
table = rxn2.pivot_table(index="catalyst", columns="temp_C", values="yield_pct", aggfunc="sum")
print(table)
```

出力:
```text
temp_C     50     80    110
catalyst                   
Ni         NaN   58.0   NaN
Pd        62.0  154.0  85.0
Pt         NaN   70.0   NaN
```

---

## 解答

??? success "問1 の解答"
    ```python
    pivot2 = rxn.pivot_table(index="temp_C", columns="catalyst", values="yield_pct")
    print(pivot2)
    ```

    出力:
    ```text
    catalyst    Ni    Pd    Pt
    temp_C                    
    50        40.0  62.0  55.0
    80        58.0  78.0  70.0
    110       66.0  85.0  80.0
    ```

??? success "問2 の解答"
    ```python
    mean_yield = rxn.pivot_table(index="catalyst", values="yield_pct", aggfunc="mean")
    print(mean_yield.round(2))
    ```

    出力:
    ```text
              yield_pct
    catalyst           
    Ni            54.67
    Pd            75.00
    Pt            68.33
    ```

??? success "問3 の解答"
    ```python
    rxn2 = pd.DataFrame({
        "catalyst":  ["Pd", "Pd", "Pd", "Pt", "Ni", "Pd"],
        "temp_C":    [50, 80, 110, 80, 80, 80],
        "yield_pct": [62, 78, 85, 70, 58, 76],
    })
    print(pd.crosstab(rxn2["catalyst"], rxn2["temp_C"]))
    ```

    出力:
    ```text
    temp_C    50   80   110
    catalyst               
    Ni          0    1    0
    Pd          1    2    1
    Pt          0    1    0
    ```

    Pd の 80℃ が **2** になっています。

??? success "問4 の解答"
    **誤り**：`aggfunc="sum"` で、Pd・80 ℃ の2回の測定（78% と 76%）が**足し算**されています。収率 154% は化学的にあり得ません（100% を超える収率は、生成物の乾燥不足や不純物の混入を疑う値です）。繰り返し測定をまとめるなら平均（`"mean"`）を使います。

    **確かめ方**：100% を超える収率が出た時点でおかしいと気づけます。`crosstab` で測定回数を並べて見ると、2回分が合計されたことが分かります。

    ```python
    table = rxn2.pivot_table(index="catalyst", columns="temp_C", values="yield_pct", aggfunc="mean")
    print(table)
    print(pd.crosstab(rxn2["catalyst"], rxn2["temp_C"]))
    ```

    出力:
    ```text
    temp_C     50    80    110
    catalyst                  
    Ni         NaN  58.0   NaN
    Pd        62.0  77.0  85.0
    Pt         NaN  70.0   NaN
    temp_C    50   80   110
    catalyst               
    Ni          0    1    0
    Pd          1    2    1
    Pt          0    1    0
    ```

    Pd・80 ℃ の 77.0 は「2回の平均」、ほかは1回だけの値です。NaN は未測定の条件です。

---

## この回のまとめ

- `pivot_table(index=, columns=, values=)` で2軸の集計表に組み替える。
- 同条件が複数あると既定で平均。`aggfunc` で max / sum / count などに変更。
- `crosstab(行, 列)` は件数のクロス集計。測定の抜け漏れ確認に便利。
- 「縦長データ → 見やすい2軸の表」がピボットの役割。

### 次回予告

[第40回：日付・時刻データの扱い](lesson-40.md) では、反応を時間ごとに追ったデータなど、**時刻を含むデータ**の扱い方を学びます。
