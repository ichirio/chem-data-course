# 第25回　統計量をNumPyで：平均・分散・標準偏差

!!! abstract "この回のゴール"
    - 平均・中央値・最大最小を配列から求める
    - **分散・標準偏差**を計算し、ばらつきを表す
    - 「標本」と「母集団」で割り方が違うこと（ddof）を知る
    - 所要時間の目安: 60分
    - 使うテーマ：**繰り返し測定**のばらつき

`lesson25.py` を作りましょう。`import numpy as np` から。

```python
import numpy as np
data = np.array([12.1, 12.3, 12.0, 12.4, 12.2, 12.5])   # 6回の測定値
```

---

## 1. 基本の統計量

配列に対して、メソッドや関数でまとめて求められます。

```python
import numpy as np
data = np.array([12.1, 12.3, 12.0, 12.4, 12.2, 12.5])

print("平均:", data.mean())
print("中央値:", np.median(data))
print("最小:", data.min(), " 最大:", data.max())
```

出力:

```text
平均: 12.25
中央値: 12.25
最小: 12.0  最大: 12.5
```

---

## 2. 標準偏差：ばらつきの大きさ

**標準偏差**は「平均からどれくらい散らばっているか」を表す代表的な指標です。`std()` で求めます。

```python
import numpy as np
data = np.array([12.1, 12.3, 12.0, 12.4, 12.2, 12.5])

print("標準偏差(母集団):", round(data.std(), 4))
print("標準偏差(標本):", round(data.std(ddof=1), 4))
print("分散(標本):", round(data.var(ddof=1), 6))
```

出力:

```text
標準偏差(母集団): 0.1708
標準偏差(標本): 0.1871
分散(標本): 0.035
```

!!! warning "ddof＝標本か母集団か"
    - `std()`（既定 `ddof=0`）… **母集団**の標準偏差（n で割る）。
    - `std(ddof=1)` … **標本**の標準偏差（n−1 で割る）。

    実験データは「母集団から取った標本」であることが多いので、**`ddof=1`（n−1で割る）を使うのが一般的**です。pandas の `.std()` は既定で `ddof=1`、NumPy は既定で `ddof=0` と**既定値が違う**ので注意しましょう。

---

## 3. 分位数（percentile）

データを小さい順に並べたとき、下から何%の位置にあるかが分位数です（第42回の四分位数）。

```python
import numpy as np
data = np.array([12.1, 12.3, 12.0, 12.4, 12.2, 12.5])

print("25%点:", np.percentile(data, 25))
print("75%点:", np.percentile(data, 75))
```

出力:

```text
25%点: 12.125
75%点: 12.375
```

---

## 4. まとめて要約する

複数の統計量を一度に表示すると、データの"感触"がつかめます。

```python
import numpy as np
data = np.array([12.1, 12.3, 12.0, 12.4, 12.2, 12.5])

print(f"n={len(data)}, 平均={data.mean():.3f}, 標準偏差={data.std(ddof=1):.3f}")
print(f"範囲: {data.min()} 〜 {data.max()}")
```

出力:

```text
n=6, 平均=12.250, 標準偏差=0.187
範囲: 12.0 〜 12.5
```

これは第31回で使った pandas の `describe()` と同じ情報です。NumPy でも同じことができます。

---

## 演習問題

**問1.** 測定値 `data = np.array([25.1, 24.8, 25.3, 25.0, 24.9, 25.2])` の平均・中央値・最小・最大を表示してください。

**問2.** 同じデータの**標本標準偏差**（`ddof=1`）を小数第4位まで表示してください。母集団標準偏差（`ddof=0`）とどれくらい違いますか？

**問3.** 同じデータの 25%点・50%点・75%点を `np.percentile` で表示してください（50%点は中央値と一致するはずです）。

---

## 解答

??? success "問1 の解答"
    ```python
    import numpy as np
    data = np.array([25.1, 24.8, 25.3, 25.0, 24.9, 25.2])
    print("平均:", round(data.mean(), 4))
    print("中央値:", np.median(data))
    print("最小:", data.min(), "最大:", data.max())
    ```

    出力:
    ```text
    平均: 25.05
    中央値: 25.05
    最小: 24.8 最大: 25.3
    ```

??? success "問2 の解答"
    ```python
    import numpy as np
    data = np.array([25.1, 24.8, 25.3, 25.0, 24.9, 25.2])
    print("標本(ddof=1):", round(data.std(ddof=1), 4))
    print("母集団(ddof=0):", round(data.std(), 4))
    ```

    出力:
    ```text
    標本(ddof=1): 0.1871
    母集団(ddof=0): 0.1708
    ```
    n が小さいほど、両者の差は大きくなります。

??? success "問3 の解答"
    ```python
    import numpy as np
    data = np.array([25.1, 24.8, 25.3, 25.0, 24.9, 25.2])
    print("25%:", round(np.percentile(data, 25), 3))
    print("50%:", round(np.percentile(data, 50), 3))
    print("75%:", round(np.percentile(data, 75), 3))
    ```

    出力:
    ```text
    25%: 24.925
    50%: 25.05
    75%: 25.175
    ```

    !!! note "なぜ `round` を付けた？"
        `round` を付けずに `np.percentile(data, 25)` を表示すると `24.924999999999997` のような値になります。これは第29回で学ぶ**小数の誤差**です。表示前に `round` で丸めておくと読みやすくなります。

---

## この回のまとめ

- `mean` / `np.median` / `min` / `max` で基本統計量。
- `std` / `var` でばらつき。**実験データは `ddof=1`（標本）**が一般的。
- NumPy の既定は `ddof=0`、pandas は `ddof=1` と違うので注意。
- `np.percentile` で分位数（四分位数）。

### 次回予告

[第26回：乱数とシミュレーション入門](lesson-26.md) では、乱数を使って測定のばらつきを模擬したり、モンテカルロ法で値を推定したりします。
