# 第92回　scikit-learn入門

!!! abstract "この回のゴール"
    - scikit-learn の基本パターン（fit / predict / score）を覚える
    - 最初のモデルを学習させ、予測する
    - すべての手法に共通する「型」をつかむ
    - 所要時間の目安: 60分

**scikit-learn**（sklearn）は Python の機械学習ライブラリの定番です。回帰・分類・クラスタリングなど、あらゆる手法が**同じ書き方（型）**で使えるのが最大の魅力です。

!!! info "準備：scikit-learn"
    第1回で入れていなければ：
    ```bash
    conda install -c conda-forge scikit-learn -y   # conda環境
    pip install scikit-learn                         # venv
    ```

`ml92.py` を作りましょう。

---

## 1. scikit-learn の基本パターン

どのモデルも、次の3ステップで使います。**この型さえ覚えれば、手法を差し替えるだけ**で色々できます。

```python
from sklearn.linear_model import LinearRegression

# ① モデルを作る
model = LinearRegression()

# ② 学習させる（fit）：X = 入力、y = 正解
model.fit(X, y)

# ③ 予測する（predict）
model.predict(新しいX)
```

---

## 2. 実際にやってみる

簡単な例で試します。入力 X と正解 y からモデルを学習させ、新しい値を予測します。

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# 入力（2次元にする必要がある：[[1],[2],...] の形）
X = np.array([[1], [2], [3], [4]])
y = np.array([2.1, 4.0, 6.1, 7.9])

model = LinearRegression()
model.fit(X, y)                       # 学習

print("傾き:", round(model.coef_[0], 3))
print("切片:", round(model.intercept_, 3))
print("x=5 の予測:", round(model.predict([[5]])[0], 2))
print("R^2:", round(model.score(X, y), 4))
```

出力:

```text
傾き: 1.95
切片: 0.15
x=5 の予測: 9.9
R^2: 0.9992
```

`fit` で学習し、`predict` で予測、`score` で当てはまりの良さ（回帰なら R²）を確認できました。

!!! warning "X は「2次元」にする"
    scikit-learn では、入力 X は **`[[1], [2], [3]]` の形（行＝サンプル、列＝特徴量）**にします。1次元の `[1, 2, 3]` はエラーになります。`np.array([...]).reshape(-1, 1)` で2次元にできます。

---

## 3. なぜ「同じ型」が便利か

手法を変えても、`fit` → `predict` の型は同じです。

```python
from sklearn.linear_model import LinearRegression, LogisticRegression
from sklearn.tree import DecisionTreeRegressor

# どれも fit → predict の型は同じ
model = LinearRegression()      # 線形回帰
# model = LogisticRegression()  # ロジスティック回帰（分類）
# model = DecisionTreeRegressor()  # 決定木

model.fit(X, y)
model.predict(新しいX)
```

!!! success "1つ覚えれば応用が利く"
    `fit` / `predict` / `score` の型は、scikit-learn の**すべてのモデルで共通**です。だから、線形回帰を覚えれば、決定木もランダムフォレストもサポートベクターマシンも、ほぼ同じ書き方で試せます。第93回以降、いろいろなモデルをこの型で使っていきます。

---

## 演習問題

**問1.** `X = np.array([[10], [20], [30], [40]])`、`y = np.array([25, 45, 62, 85])` で `LinearRegression` を学習させ、傾き・切片・R² を表示してください。

**問2.** 問1のモデルで、`x = 50` のときの予測値を表示してください。

**問3.** 1次元配列 `np.array([1, 2, 3])` をそのまま `fit` に渡すとエラーになります。`reshape(-1, 1)` で2次元に直してから使うと動くことを確認してください。

---

## 解答

??? success "問1 の解答"
    ```python
    import numpy as np
    from sklearn.linear_model import LinearRegression
    X = np.array([[10], [20], [30], [40]])
    y = np.array([25, 45, 62, 85])
    model = LinearRegression().fit(X, y)
    print("傾き:", round(model.coef_[0], 3))
    print("切片:", round(model.intercept_, 3))
    print("R^2:", round(model.score(X, y), 4))
    ```

    出力:
    ```text
    傾き: 1.97
    切片: 5.0
    R^2: 0.9968
    ```

??? success "問2 の解答"
    ```python
    print("x=50 の予測:", round(model.predict([[50]])[0], 2))
    ```

    出力:
    ```text
    x=50 の予測: 103.5
    ```

??? success "問3 の解答"
    ```python
    import numpy as np
    a = np.array([1, 2, 3])
    print(a.reshape(-1, 1))     # 2次元になる
    ```

    出力:
    ```text
    [[1]
     [2]
     [3]]
    ```
    この2次元の形が、scikit-learn の入力 X の正しい形です。

---

## この回のまとめ

- scikit-learn の基本型：`model.fit(X, y)` → `model.predict(新X)` → `model.score(X, y)`。
- 入力 X は**2次元**（行＝サンプル、列＝特徴量）。`reshape(-1, 1)` で直せる。
- この型は**すべてのモデルで共通**。1つ覚えれば応用が利く。

### 次回予告

[第93回：回帰モデルで物性を予測する](lesson-93.md) では、分子の性質（沸点）を回帰で予測します。
