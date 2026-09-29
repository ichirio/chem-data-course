# 第35回　欠損値の扱い（生化学アッセイデータ）

!!! abstract "この回のゴール"
    - **欠損値（NaN＝空欄）**が何かを知る
    - `isna()` で欠損の場所と数を調べる
    - `dropna()` で欠損のある行を除く／`fillna()` で埋める
    - 集計が欠損をどう扱うかを理解する
    - 所要時間の目安: 60分
    - 使うデータ：**生化学**（酵素アッセイの吸光度測定）

`lesson35.py` を作りましょう。現実の実験データには、測定ミスや未測定による**空欄**がつきものです。今回はわざと欠損を含むデータで練習します。

```python
import pandas as pd

# 酵素アッセイ（架空データ）：基質濃度を変え、一定時間反応させた後の吸光度を測定（一部が欠損）
assay = pd.DataFrame({
    "sample":        ["S1", "S2", "S3", "S4", "S5", "S6"],
    "substrate_mM":  [0.5, 1.0, 2.0, 4.0, 8.0, 16.0],   # 基質濃度 [mM]
    "absorbance":    [0.12, 0.21, 0.35, None, 0.58, 0.62],  # 吸光度（S4が欠損）
    "temperature_C": [25.0, 25.0, None, 25.0, 25.0, 25.0],  # 温度（S3が欠損）
})
print(assay)
```

出力:

```text
  sample  substrate_mM  absorbance  temperature_C
0     S1           0.5        0.12           25.0
1     S2           1.0        0.21           25.0
2     S3           2.0        0.35            NaN
3     S4           4.0         NaN           25.0
4     S5           8.0        0.58           25.0
5     S6          16.0        0.62           25.0
```

Python で `None` と書いた欠損は、pandas では **`NaN`（Not a Number）** と表示されます。これが「空欄」の印です。

---

## 1. 欠損を見つける：isna()

まず「どこに・いくつ」欠損があるかを把握します。

```python
print(assay.isna())          # 各セルが欠損か（True/False）
print("---")
print(assay.isna().sum())    # 列ごとの欠損の個数
```

`isna().sum()` の出力:

```text
sample           0
substrate_mM     0
absorbance       1
temperature_C    1
dtype: int64
```

`absorbance` に1個、`temperature_C` に1個の欠損があると分かりました。**分析の最初に欠損の数を確認**するのが良い習慣です。

---

## 2. 欠損を除く：dropna()

欠損のある行を丸ごと取り除きます。

```python
clean = assay.dropna()
print(clean)
```

出力:

```text
  sample  substrate_mM  absorbance  temperature_C
0     S1           0.5        0.12           25.0
1     S2           1.0        0.21           25.0
4     S5           8.0        0.58           25.0
5     S6          16.0        0.62           25.0
```

欠損を含んでいた S3・S4 の行が消えました。index が 2,3 だけ飛んでいるのが、行が除かれた証拠です。

!!! warning "捨てすぎに注意"
    `dropna()` は手軽ですが、貴重なデータを失うこともあります。「1列でも欠損があれば行ごと削除」なので、欠損が多いと激減します。**まず isna で状況を見てから**判断しましょう。

---

## 3. 集計は欠損を自動でスキップする

pandas の集計（`mean` など）は、**欠損を除いて**計算してくれます。

```python
print("吸光度の平均:", assay["absorbance"].mean().round(3))
print("有効な測定数:", assay["absorbance"].count())   # count は欠損を数えない
```

出力:

```text
吸光度の平均: 0.376
有効な測定数: 5
```

6サンプル中、欠損の S4 を除いた**5個**で平均が計算されています。`count()` が「欠損でない件数」を返す点も便利です。

---

## 4. 欠損を埋める：fillna()

行を消さずに、代わりの値で埋める方法です。よく使うのは「平均で埋める」「決まった値で埋める」。

```python
filled = assay.copy()      # 元を壊さないようコピー

# 吸光度は「その列の平均」で埋める
mean_abs = filled["absorbance"].mean().round(3)
filled["absorbance"] = filled["absorbance"].fillna(mean_abs)

# 温度は測定条件から 25.0 と分かっているので固定値で埋める
filled["temperature_C"] = filled["temperature_C"].fillna(25.0)

print(filled)
```

出力:

```text
  sample  substrate_mM  absorbance  temperature_C
0     S1           0.5       0.120           25.0
1     S2           1.0       0.210           25.0
2     S3           2.0       0.350           25.0
3     S4           4.0       0.376           25.0
4     S5           8.0       0.580           25.0
5     S6          16.0       0.620           25.0
```

S4 の吸光度が平均値 0.376 で、S3 の温度が 25.0 で埋まりました。

!!! warning "測定値を平均で埋めてよいか、化学で判断する"
    この表の吸光度は**基質濃度ごとに違う条件**で測った値です。0.5〜16 mM 全体の平均を 4 mM の値として使うのは、化学的には根拠がありません（ここでは操作の練習として埋めています）。実際の酵素アッセイでは、S4 は**再測定する**か、**解析から除く**（`dropna(subset=["absorbance"])`）のが普通です。平均で埋めてよいのは、同じ条件の繰り返し測定の一部が欠けた場合などに限られます。

!!! note "どれを選ぶ？（実務の判断）"
    - **dropna（除く）**… 欠損が少なく、消しても十分データが残るとき。
    - **fillna 固定値**… 欠損の理由が分かっているとき（今回の温度＝実験条件は25℃固定）。
    - **fillna 平均/中央値**… 同じ条件の値が並んでいて、平均で代用しても意味が変わらないとき（条件の違う測定値を平均で埋めると、傾向をゆがめます）。
    
    「正解は1つ」ではありません。**なぜその処理を選んだかを説明できる**ことが大切です（研究の再現性）。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    pandas の DataFrame `assay`（列: sample, substrate_mM [mM], absorbance [吸光度・単位なし], temperature_C [℃]）の欠損を処理してください。
    まず `isna().sum()` で列ごとの欠損数を表示してください。
    吸光度が欠損した行だけを除き（`dropna(subset=["absorbance"])`）、温度の欠損は実験条件の 25.0 ℃ で埋めてください。
    `inplace=True` は使わず、結果は変数に代入し直してください。処理前後の行数も print してください。

!!! warning "AIの出力で確かめること"
    - `dropna()` を引数なしで使っていないか。それだと、温度だけが欠けた S3（吸光度 0.35 は有効）まで消えて、6点が4点になります。処理前後の `len()` を比べ、吸光度の解析なら5点残っているかを確認します。
    - `df["列"].fillna(値, inplace=True)` と書かれていないか。pandas 3.x ではこの形は**元の表を変えず**、`ChainedAssignmentError` という警告が出るだけです。`df["列"] = df["列"].fillna(値)` か `df = df.fillna({"列": 値})` の形になっているかを見ます。処理後に `isna().sum()` がすべて 0 になっているかも確認します。
    - 吸光度のように**条件（基質濃度）で値が変わる測定値**を、列の平均で埋めていないか。埋めるなら理由を説明できるかを確認します。
    - 集計の件数を `count()` で確認します（`len()` は欠損も数え、`count()` は数えません）。吸光度の有効数は 5 です。

---

## 演習問題

**問1.** 本文の `assay` を作り、`isna().sum()` で列ごとの欠損数を表示してください。どの列にいくつ欠損がありますか？

**問2.** `dropna()` で欠損のない行だけを残した表を作り、その**行数**を `len()` で表示してください。

**問3.** `temperature_C` の欠損を `25.0` で、`absorbance` の欠損を**その列の中央値**（`.median()`）で埋めた表を作って表示してください。

**問4.**（検証型）次は AI が「吸光度が測れている点だけで解析し、温度の欠損は 25.0 ℃ で埋める」ために書いたコードとその出力です。問題点を**2つ**指摘し、直してください。

```python
clean = assay.dropna()
print(len(clean), "点で解析:", clean["substrate_mM"].tolist())
assay["temperature_C"].fillna(25.0, inplace=True)
print(assay.isna().sum())
```

出力（警告文は冒頭のみ）:
```text
4 点で解析: [0.5, 1.0, 8.0, 16.0]
ChainedAssignmentError: A value is being set on a copy of a DataFrame or Series through chained assignment using an inplace method.
（中略）
sample           0
substrate_mM     0
absorbance       1
temperature_C    1
dtype: int64
```

---

## 解答

??? success "問1 の解答"
    ```python
    print(assay.isna().sum())
    ```

    出力:
    ```text
    sample           0
    substrate_mM     0
    absorbance       1
    temperature_C    1
    dtype: int64
    ```
    → `absorbance` と `temperature_C` に1個ずつ。

??? success "問2 の解答"
    ```python
    clean = assay.dropna()
    print("欠損なしの行数:", len(clean))
    ```

    出力:
    ```text
    欠損なしの行数: 4
    ```

??? success "問3 の解答"
    ```python
    filled = assay.copy()
    filled["temperature_C"] = filled["temperature_C"].fillna(25.0)
    med = filled["absorbance"].median()
    filled["absorbance"] = filled["absorbance"].fillna(med)
    print(filled)
    print("使った中央値:", med)
    ```

    出力:
    ```text
      sample  substrate_mM  absorbance  temperature_C
    0     S1           0.5        0.12           25.0
    1     S2           1.0        0.21           25.0
    2     S3           2.0        0.35           25.0
    3     S4           4.0        0.35           25.0
    4     S5           8.0        0.58           25.0
    5     S6          16.0        0.62           25.0
    使った中央値: 0.35
    ```

    （欠損を除いた5個 0.12, 0.21, 0.35, 0.58, 0.62 の中央値は 0.35。）

??? success "問4 の解答"
    **誤り1**：`dropna()` を引数なしで使ったため、**温度だけ**が欠けていた S3（2.0 mM、吸光度 0.35 は有効）まで消えました。吸光度の解析に使えるのは5点なのに、4点になっています。`dropna(subset=["absorbance"])` のように、見る列を指定します。

    **誤り2**：`assay["temperature_C"].fillna(25.0, inplace=True)` は、pandas 3.x では**元の表を変えません**（`ChainedAssignmentError` の警告が出ます）。実際、直後の `isna().sum()` で temperature_C の欠損が 1 のまま残っています。`fillna({"列名": 値})` の結果を変数に代入します。

    ```python
    clean = assay.dropna(subset=["absorbance"])
    print(len(clean), "点で解析:", clean["substrate_mM"].tolist())
    filled = assay.fillna({"temperature_C": 25.0})
    print(filled.isna().sum())
    ```

    出力:
    ```text
    5 点で解析: [0.5, 1.0, 2.0, 8.0, 16.0]
    sample           0
    substrate_mM     0
    absorbance       1
    temperature_C    0
    dtype: int64
    ```

    （absorbance の欠損 1 は、埋めずに残す方針なのでそのままです。解析では `clean` を使います。）

---

## この回のまとめ

- 空欄は **NaN**。`isna().sum()` で「どこに・いくつ」を最初に確認。
- `dropna()` … 欠損のある行を除く（捨てすぎ注意）。
- `fillna(値)` … 欠損を埋める（固定値・平均・中央値など）。
- `mean()` など集計は欠損を自動でスキップ、`count()` は有効件数を返す。
- 処理法に唯一の正解はない。**選んだ理由を説明できる**ことが大切。

### 次回予告

[第36回：並べ替えと集計（groupby）](lesson-36.md) では、「カテゴリごとの平均」のような**グループ集計**を学びます。第4部の山場であり、レポートの表づくりに直結します。
