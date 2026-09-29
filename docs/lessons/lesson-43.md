# 第43回　データの正規化・標準化

!!! abstract "この回のゴール"
    - スケールの違う指標をそろえる「正規化」「標準化」を知る
    - **min-max正規化**（0〜1に収める）を計算する
    - **標準化（zスコア）**（平均0・標準偏差1にする）を計算する
    - どちらをいつ使うかを理解する
    - 所要時間の目安: 60分
    - 使うデータ：**分子記述子**（LogP など）

分子量・LogP・融点…単位もスケールもバラバラな指標を、そのまま比べたり機械学習に入れたりするとうまくいきません。**同じ土俵に乗せる**のが正規化・標準化です。

```python
import pandas as pd

d = pd.DataFrame({
    "compound": ["A", "B", "C", "D"],
    "logP":     [1.2, 2.8, 3.5, 0.5],
})
print(d)
```

出力:

```text
  compound  logP
0        A   1.2
1        B   2.8
2        C   3.5
3        D   0.5
```

---

## 1. min-max正規化：0〜1に収める

最小値を0、最大値を1にして、その間に比例配分します。

$$ x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}} $$

```python
d["minmax"] = ((d["logP"] - d["logP"].min()) / (d["logP"].max() - d["logP"].min())).round(3)
print(d)
```

出力:

```text
  compound  logP  minmax
0        A   1.2   0.233
1        B   2.8   0.767
2        C   3.5   1.000
3        D   0.5   0.000
```

最小の D が 0、最大の C が 1 になりました。「全体の中での相対位置」がひと目で分かります。

!!! note "min-maxが向く場面"
    値の範囲が決まっている・0〜1にそろえたい・画像やグラフの色スケールに使う、といった場面。ただし**外れ値に弱い**（極端な最大値があると、他がすべて0付近に潰れる）ので、事前の外れ値チェック（第42回）が大切です。

---

## 2. 標準化（zスコア）：平均0・標準偏差1にする

平均を引いて標準偏差で割ります。「平均から標準偏差いくつ分ずれているか」を表します。

$$ z = \frac{x - \bar{x}}{s} $$

```python
d["zscore"] = ((d["logP"] - d["logP"].mean()) / d["logP"].std()).round(3)
print(d)
```

出力:

```text
  compound  logP  minmax  zscore
0        A   1.2   0.233  -0.576
1        B   2.8   0.767   0.576
2        C   3.5   1.000   1.081
3        D   0.5   0.000  -1.081
```

zスコアが**正なら平均より上、負なら平均より下**。C は平均より約1.08σ上、D は約1.08σ下、と分かります。

!!! note "標準化が向く場面"
    多くの統計手法・機械学習（第8部）が前提とする形。値の範囲が 0〜1 に固定されないので、極端な値が1つあっても他の値が0付近に潰れにくい、という利点があります。ただし平均と標準偏差そのものが外れ値に引っぱられるので、**外れ値の影響は受けます**。外れ値チェック（第42回）を先に行い、迷ったらまず標準化、が無難です。

---

## 3. 使い分けのまとめ

| | min-max正規化 | 標準化（zスコア） |
|---|---|---|
| 変換後 | 0〜1 の範囲 | 平均0・標準偏差1 |
| 式 | (x−min)/(max−min) | (x−平均)/標準偏差 |
| 外れ値 | 弱い（他の値が潰れやすい） | 影響を受ける（平均・標準偏差が引っぱられる） |
| 向く用途 | 範囲を固定したい | 統計・機械学習の前処理 |

!!! tip "本番では scikit-learn が定番"
    実務では `sklearn.preprocessing` の `MinMaxScaler` / `StandardScaler` を使うことが多いです（第8部で登場）。中で何をしているかは、この回の手計算とほぼ同じです。ただし `StandardScaler` は標準偏差を **n で割る版**（`std(ddof=0)`）で計算するので、pandas の `.std()`（n−1 で割る版）を使った本文の値とは少しずれます（本文のデータでは C が 1.081 と 1.248）。どちらの標準偏差を使ったかをそろえて比べましょう。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    pandas の DataFrame `d`（列: compound, logP [無次元], mw [g/mol]）の logP と mw について、
    min-max 正規化（(x−min)/(max−min)）と標準化（(x−平均)/標準偏差、pandas の `.std()`＝ddof=1 を使用）の列を追加してください。
    scikit-learn は使わず pandas だけで計算し、最後に変換後の各列の min・max・mean・std を `agg` で表示して、変換が正しいか確認できるようにしてください。

!!! warning "AIの出力で確かめること"
    - min-max 列の最小が 0、最大が 1 になっているか。分母が `(max − min)` ではなく `max` になっていると、最大が 1 になりません。
    - zスコア列の平均が 0（`-0.000` と表示されるのは浮動小数点の誤差で、0 と同じ）、標準偏差が 1 になっているか。`d[["minmax", "zscore"]].agg(["min", "max", "mean", "std"])` で一度に確かめられます。
    - AI が `StandardScaler` を使った場合、値が pandas の手計算と少し違うのは正常です（StandardScaler は ddof=0、pandas の `.std()` は ddof=1）。どちらを使ったかをそろえて比較します。
    - 機械学習の前処理では、min・max・平均・標準偏差を**訓練データだけ**から求めているか（テストデータを含めて計算するとデータ漏洩になります。第8部で扱います）。
    - 変換前の値が化学的に妥当か（例：分子量 46 の小分子で logP 3.5 はあり得ない）。スケーリングは元の値の誤りを隠してしまいます。

---

## 演習問題

**問1.** 本文の `d` に、`logP` の **min-max正規化**列 `minmax` を追加して表示してください。値が 0〜1 に収まっていることを確認しましょう。

**問2.** `logP` の**標準化（zスコア）**列 `zscore` を追加してください。zスコアの**平均**が（ほぼ）0、**標準偏差**が（ほぼ）1 になることを `.mean()` `.std()` で確かめましょう。

**問3.** 新しい記述子 `mw`（分子量、架空の値）`[120, 180, 250, 60]` を `d` に加え、その min-max正規化列 `mw_minmax` を作ってください。logP と分子量が、同じ 0〜1 のスケールで比べられることを確認しましょう。

**問4.**（検証型）次は AI が logP の min-max 正規化と標準化を書いたコードと、その確認用の出力です。問題点を**2つ**指摘し、直してください。

```python
d = pd.DataFrame({"compound": ["A", "B", "C", "D"], "logP": [1.2, 2.8, 3.5, 0.5]})
d["minmax"] = (d["logP"] - d["logP"].min()) / d["logP"].max()
d["zscore"] = (d["logP"] - d["logP"].mean()) / d["logP"].max()
print(d[["minmax", "zscore"]].agg(["min", "max", "mean", "std"]).round(3))
```

出力:
```text
      minmax  zscore
min    0.000  -0.429
max    0.857   0.429
mean   0.429  -0.000
std    0.397   0.397
```

---

## 解答

??? success "問1 の解答"
    ```python
    d["minmax"] = ((d["logP"] - d["logP"].min()) / (d["logP"].max() - d["logP"].min())).round(3)
    print(d[["compound", "logP", "minmax"]])
    ```

    出力:
    ```text
      compound  logP  minmax
    0        A   1.2   0.233
    1        B   2.8   0.767
    2        C   3.5   1.000
    3        D   0.5   0.000
    ```

??? success "問2 の解答"
    ```python
    d["zscore"] = ((d["logP"] - d["logP"].mean()) / d["logP"].std()).round(3)
    print("zscoreの平均:", round(d["zscore"].mean(), 3))
    print("zscoreの標準偏差:", round(d["zscore"].std(), 3))
    ```

    出力:
    ```text
    zscoreの平均: 0.0
    zscoreの標準偏差: 1.0
    ```

    平均0・標準偏差1。標準化の狙いどおりになっています。

??? success "問3 の解答"
    ```python
    d["mw"] = [120, 180, 250, 60]
    d["mw_minmax"] = ((d["mw"] - d["mw"].min()) / (d["mw"].max() - d["mw"].min())).round(3)
    print(d[["compound", "logP", "minmax", "mw", "mw_minmax"]])
    ```

    出力:
    ```text
      compound  logP  minmax   mw  mw_minmax
    0        A   1.2   0.233  120      0.316
    1        B   2.8   0.767  180      0.632
    2        C   3.5   1.000  250      1.000
    3        D   0.5   0.000   60      0.000
    ```

    logP と分子量が同じ 0〜1 スケールになり、指標をまたいだ比較ができます。

??? success "問4 の解答"
    **誤り1（min-max の分母）**：分母が `max` だけになっています。正しくは `(max − min)` です。その結果、最大値（C の logP 3.5）が 1 にならず 0.857 になっています。

    **誤り2（標準化の分母）**：標準偏差 `std()` で割るべきところを `max()` で割っています。zスコアの標準偏差が 1 ではなく 0.397 になっていることで分かります。

    **確かめ方**：min-max は「min 0・max 1」、zスコアは「mean 0・std 1」になるはずです。`agg(["min", "max", "mean", "std"])` の表で一目で確認できます（`-0.000` は浮動小数点の誤差で、0 と同じです）。

    ```python
    d = pd.DataFrame({"compound": ["A", "B", "C", "D"], "logP": [1.2, 2.8, 3.5, 0.5]})
    d["minmax"] = (d["logP"] - d["logP"].min()) / (d["logP"].max() - d["logP"].min())
    d["zscore"] = (d["logP"] - d["logP"].mean()) / d["logP"].std()
    print(d.round(3))
    print(d[["minmax", "zscore"]].agg(["min", "max", "mean", "std"]).round(3))
    ```

    出力:
    ```text
      compound  logP  minmax  zscore
    0        A   1.2   0.233  -0.576
    1        B   2.8   0.767   0.576
    2        C   3.5   1.000   1.081
    3        D   0.5   0.000  -1.081
          minmax  zscore
    min    0.000  -1.081
    max    1.000   1.081
    mean   0.500  -0.000
    std    0.463   1.000
    ```

---

## この回のまとめ

- スケールの違う指標は、そろえてから比べる／学習させる。
- **min-max**：(x−min)/(max−min) で 0〜1 に。範囲固定に便利、外れ値に弱い。
- **標準化(zスコア)**：(x−平均)/標準偏差 で平均0・標準偏差1に。統計・機械学習の定番。
- 実務は scikit-learn の Scaler。中身はこの手計算とほぼ同じ（StandardScaler は ddof=0 の標準偏差を使う）。

### 次回予告

[第44回：Excelファイルの読み書き](lesson-44.md) では、研究現場で避けて通れない Excel ファイル（.xlsx）を pandas で読み書きします。
