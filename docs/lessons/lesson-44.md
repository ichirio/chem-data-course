# 第44回　Excelファイルの読み書き

!!! abstract "この回のゴール"
    - pandas で Excel ファイル（.xlsx）を**読み書き**する
    - 複数シートを1つのファイルに書き分ける
    - シートを指定して読み込む
    - 所要時間の目安: 60分
    - 使うデータ：**高分子** と **金属酸化物**（2シート）

研究現場のデータは Excel で管理されていることが多いもの。pandas は Excel も CSV とほぼ同じ感覚で扱えます。

!!! info "準備：openpyxl"
    Excel を扱うには `openpyxl` が必要です（第1回の `conda install` 群に含めていなければ）。
    ```bash
    conda install -c conda-forge openpyxl -y   # conda環境の人
    pip install openpyxl                        # venvの人
    ```

`lesson44.py` を作り、2つの表を用意します。

```python
import pandas as pd

polymers = pd.DataFrame({
    "polymer":     ["PE", "PP", "PS"],
    "tensile_MPa": [25, 35, 45],
})
oxides = pd.DataFrame({
    "oxide":   ["Al2O3", "TiO2"],
    "density": [3.95, 4.23],
})
```

---

## 1. Excel に書き出す（1シート）

CSV の `to_csv` に対応するのが `to_excel` です。

```python
polymers.to_excel("polymers.xlsx", index=False)
print("polymers.xlsx を保存しました")
```

`index=False` で行番号を書かないのは CSV と同じです。できたファイルを Excel で開くと、そのまま表になっています。

---

## 2. 複数シートに書き分ける

1つの Excel ファイルに、複数の表を**別々のシート**として保存できます。`ExcelWriter` を使います。

```python
with pd.ExcelWriter("results.xlsx") as writer:
    polymers.to_excel(writer, sheet_name="polymers", index=False)
    oxides.to_excel(writer, sheet_name="oxides", index=False)

print("results.xlsx（2シート）を保存しました")
```

`with ... as writer:` の中で、シート名を変えながら書き込みます。これで「polymers」「oxides」の2シートを持つ1ファイルができます。

!!! note "`with` 構文について"
    `with pd.ExcelWriter(...) as writer:` は「ファイルを開いて、ブロックを抜けたら自動で閉じる」書き方です。閉じ忘れによる書き込み失敗を防いでくれます（第12回で詳しく扱います）。

---

## 3. Excel を読み込む

`read_csv` に対応するのが `read_excel`。`sheet_name` で読むシートを指定します。

```python
back = pd.read_excel("results.xlsx", sheet_name="polymers")
print(back)
```

出力:

```text
  polymer  tensile_MPa
0      PE           25
1      PP           35
2      PS           45
```

どんなシートがあるか調べたいときは `ExcelFile` を使います。

```python
xls = pd.ExcelFile("results.xlsx")
print(xls.sheet_names)
```

出力:

```text
['polymers', 'oxides']
```

!!! tip "全シートをまとめて読む"
    `sheet_name=None` にすると、**全シートを辞書**（シート名 → DataFrame）で受け取れます。
    ```python
    all_sheets = pd.read_excel("results.xlsx", sheet_name=None)
    print(all_sheets["oxides"])
    ```

---

## CSV と Excel、どちらを使う？

| | CSV | Excel (.xlsx) |
|---|---|---|
| 中身 | ただのテキスト | 独自形式（複数シート・書式可） |
| 手軽さ・互換性 | ◎（何でも開ける） | ○（Excel等が必要） |
| 複数シート・書式 | ✕ | ◎ |
| プログラム処理・再現性 | ◎ | ○ |

!!! note "使い分けの目安"
    - **データの受け渡し・解析・再現性** → CSV（軽くて堅牢）。
    - **人に見せる・複数の表をまとめる・書式をつける** → Excel。
    
    分析の途中は CSV、最終的な報告物は Excel、という併用がよくあります。

---

## 演習問題

**問1.** 本文の `polymers` を `polymers.xlsx` に保存し（`index=False`）、`read_excel` で読み直して表示してください。

**問2.** `ExcelWriter` を使って、`polymers` を "poly" シート、`oxides` を "oxide" シートとして `materials.xlsx` に保存してください。そのあと `ExcelFile` でシート名一覧を表示しましょう。

**問3.** 問2で作った `materials.xlsx` から、"oxide" シートだけを読み込んで表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    polymers.to_excel("polymers.xlsx", index=False)
    df = pd.read_excel("polymers.xlsx")
    print(df)
    ```

    出力:
    ```text
      polymer  tensile_MPa
    0      PE           25
    1      PP           35
    2      PS           45
    ```

??? success "問2 の解答"
    ```python
    with pd.ExcelWriter("materials.xlsx") as writer:
        polymers.to_excel(writer, sheet_name="poly", index=False)
        oxides.to_excel(writer, sheet_name="oxide", index=False)

    xls = pd.ExcelFile("materials.xlsx")
    print(xls.sheet_names)
    ```

    出力:
    ```text
    ['poly', 'oxide']
    ```

??? success "問3 の解答"
    ```python
    df = pd.read_excel("materials.xlsx", sheet_name="oxide")
    print(df)
    ```

    出力:
    ```text
       oxide  density
    0  Al2O3     3.95
    1   TiO2     4.23
    ```

---

## この回のまとめ

- 書き出し `df.to_excel("名前.xlsx", index=False)`、読み込み `pd.read_excel("名前.xlsx")`。
- 複数シートは `with pd.ExcelWriter(...) as writer:` の中で `sheet_name` を変えて書く。
- `pd.ExcelFile(...).sheet_names` でシート名一覧、`sheet_name=None` で全シート辞書。
- 解析・再現性は CSV、見せる・まとめるは Excel。

### 次回予告

[第45回：まとめ演習（実験ノートCSVを分析する）](lesson-45.md) では、第4部の総仕上げとして、実験記録のCSVを「読み込み→前処理→集計→保存」まで一気通貫でやります。
