# 第32回　CSVを読み込む・書き出す（高分子データを扱う）

!!! abstract "この回のゴール"
    - **CSV ファイル**とは何かを知る
    - `to_csv()` で DataFrame をファイルに保存する
    - `read_csv()` でファイルを読み込む
    - 文字コード・index の扱いなど、つまずきポイントを押さえる
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性

`lesson32.py` を作って進めましょう。

---

## 1. CSV とは

**CSV**（Comma-Separated Values）は、値をカンマで区切っただけの、いちばん基本的な表データ形式です。中身はただのテキストなので、Excel でもメモ帳でも開けます。

```text
polymer,category,density,tensile_MPa,Tg_C
PE,thermoplastic,0.94,25,-110
PP,thermoplastic,0.905,35,-10
...
```

実験機器の出力、公開データベース、Excel からの書き出し——データは CSV でやり取りされることが本当に多いです。**pandas で CSV を読み書きできれば、データ分析の入口はクリア**です。

---

## 2. まずデータを用意して保存する（to_csv）

今回はポリマーの物性データを DataFrame で作り、それを CSV に書き出してみます。

```python
import pandas as pd

polymers = pd.DataFrame({
    "polymer":     ["PE", "PP", "PS", "PVC", "PET", "PMMA", "Nylon6", "Epoxy", "Phenolic"],
    "category":    ["thermoplastic"]*7 + ["thermoset"]*2,   # 熱可塑性/熱硬化性
    "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],  # g/cm^3
    "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],    # 引張強さ [MPa]
    "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],  # ガラス転移点 [℃]
})

# CSV に保存（index=False で左端の行番号を書き込まない）
polymers.to_csv("polymers.csv", index=False)
print("polymers.csv を保存しました")
```

出力:

```text
polymers.csv を保存しました
```

実行すると、同じフォルダに `polymers.csv` ができます。開くと上のようなカンマ区切りテキストになっています。

!!! warning "`index=False` を忘れると"
    付けないと、左端の行番号（0,1,2…）まで余計な列としてファイルに書き込まれます。普通は `index=False` を付けるのが無難です。

---

## 3. CSV を読み込む（read_csv）

保存した CSV を読み込んで、DataFrame に戻します。これが一番よく使う操作です。

```python
df = pd.read_csv("polymers.csv")
print(df.head())
```

出力:

```text
  polymer       category  density  tensile_MPa  Tg_C
0      PE  thermoplastic    0.940           25  -110
1      PP  thermoplastic    0.905           35   -10
2      PS  thermoplastic    1.050           45   100
3     PVC  thermoplastic    1.380           52    80
4     PET  thermoplastic    1.380           55    76
```

たった1行 `pd.read_csv("ファイル名")` で、テキストが「型つきの表」になりました。`density` は小数、`tensile_MPa` は整数、と自動で判別されています。

!!! tip "ファイルの場所（パス）に注意"
    `read_csv("polymers.csv")` は「**いま実行しているフォルダ**にある polymers.csv」を探します。見つからないとき（`FileNotFoundError`）は、ファイルとスクリプトが同じフォルダにあるか確認しましょう。別フォルダなら `pd.read_csv("data/polymers.csv")` のように相対パスで指定します。

!!! note "日本語が含まれる CSV の文字コード"
    日本語列名や全角を含む CSV で文字化けするときは、文字コードの指定が必要です。
    ```python
    df = pd.read_csv("data.csv", encoding="utf-8")     # 多くはこれ
    df = pd.read_csv("data.csv", encoding="cp932")     # Excel(日本語Windows)由来はこちらのことも
    ```

---

## 4. 読み込んだらまず確認（前回の4点セット）

CSV を読んだら、第31回の習慣どおり全体像を確認します。

```python
df = pd.read_csv("polymers.csv")

print(df.shape)          # (9, 5)
print(list(df.columns))  # 列名
print(df.dtypes)         # 型
print(df.describe())     # 数値列の要約
```

!!! success "この2ステップが分析の土台"
    **「read_csv で読む → shape/head/dtypes/describe で確認」**。どんなデータ分析もここから始まります。次回以降の「選ぶ・絞る・集計する」は、すべてこの読み込んだ表に対して行います。

---

## 演習問題

**問1.** 本文の `polymers` を作って `polymers.csv` に保存し（`index=False`）、そのあと `read_csv` で読み直して `head()` を表示してください。保存→読み込みの往復を体験しましょう。

**問2.** 読み込んだ `df` について、`shape`・列名・`dtypes` を表示してください。行数・列数はいくつですか？

**問3.** 石油化学のデータを自分で作って保存・読み込みしてみましょう。原油の代表的な留分について `fraction`（留分名）・`bp_low_C`・`bp_high_C`（沸点範囲）の3列を持つ DataFrame を作り、`distillation.csv` に保存してから読み込んで表示してください。例：Naphtha 20〜150、Kerosene 150〜250、Diesel 250〜350。

---

## 解答

??? success "問1 の解答"
    ```python
    import pandas as pd

    polymers = pd.DataFrame({
        "polymer":     ["PE", "PP", "PS", "PVC", "PET", "PMMA", "Nylon6", "Epoxy", "Phenolic"],
        "category":    ["thermoplastic"]*7 + ["thermoset"]*2,
        "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],
        "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
        "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],
    })
    polymers.to_csv("polymers.csv", index=False)

    df = pd.read_csv("polymers.csv")
    print(df.head())
    ```

??? success "問2 の解答"
    ```python
    print(df.shape)
    print(list(df.columns))
    print(df.dtypes)
    ```

    出力:
    ```text
    (9, 5)
    ['polymer', 'category', 'density', 'tensile_MPa', 'Tg_C']
    polymer         object
    category        object
    density        float64
    tensile_MPa      int64
    Tg_C             int64
    dtype: object
    ```

    → **9行・5列**です。

??? success "問3 の解答"
    ```python
    import pandas as pd

    dist = pd.DataFrame({
        "fraction": ["Naphtha", "Kerosene", "Diesel"],
        "bp_low_C":  [20, 150, 250],
        "bp_high_C": [150, 250, 350],
    })
    dist.to_csv("distillation.csv", index=False)

    df = pd.read_csv("distillation.csv")
    print(df)
    ```

    出力:
    ```text
       fraction  bp_low_C  bp_high_C
    0   Naphtha        20        150
    1  Kerosene       150        250
    2    Diesel       250        350
    ```

---

## この回のまとめ

- CSV はカンマ区切りのテキスト。データ交換の共通語。
- 保存は `df.to_csv("名前.csv", index=False)`。
- 読み込みは `pd.read_csv("名前.csv")`。たった1行で型つきの表になる。
- ファイルの場所（パス）と文字コード（`encoding`）がつまずきポイント。
- 読んだら必ず `shape`/`head`/`dtypes`/`describe` で確認。

!!! note "リポジトリのサンプルデータ"
    このコースのリポジトリには `data/` フォルダに `polymers.csv` `metal_oxides.csv` `enzyme_assay.csv` を用意してあります。clone すれば `pd.read_csv("data/polymers.csv")` のように使えます。

### 次回予告

[第33回：行と列の選択（loc / iloc）](lesson-33.md) では、大きな表から「必要な行・列だけ」を取り出す方法を学びます。
