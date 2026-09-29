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

`fit` で学習し、`predict` で予測、`score` で当てはまりの良さ（回帰なら R²）を確認できました。ただし、ここでの R² は**学習に使ったデータ自身**での値です。未知のデータでの実力の測り方は第95回で学びます。

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
    `fit` / `predict` / `score` の型は、scikit-learn の**教師あり学習のモデルで共通**です（標準化や PCA のような「変換器」は `fit` / `transform` の型、第97・98回）。だから、線形回帰を覚えれば、決定木もランダムフォレストもサポートベクターマシンも、ほぼ同じ書き方で試せます。第93回以降、いろいろなモデルをこの型で使っていきます。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「scikit-learn の LinearRegression で検量線を作りたいです。
    標準液の Fe 濃度 [mg/L] が `[0, 0.5, 1, 2, 3, 4]`、吸光度が `[0.001, 0.099, 0.200, 0.398, 0.601, 0.797]` です。
    X は濃度（2次元配列にする）、y は吸光度です。傾き（単位つき）・切片・R² を表示し、
    吸光度 0.500 の試料の濃度を**逆算**するコードを書いてください。NumPy と scikit-learn だけを使ってください。」

!!! warning "AIの出力で確かめること"
    - **X が2次元か**：`print(X.shape)` が `(6, 1)` のように「(サンプル数, 特徴量数)」になっているか。`(6,)` なら `reshape(-1, 1)` が抜けている。
    - **X と y の向き**：`fit(X, y)` の X が「原因（濃度）」、y が「結果（吸光度）」になっているか。`model.predict` に入れる値は **X と同じ量**（ここでは濃度）でなければならない。吸光度を `predict` に入れていたら誤り。
    - **傾きの単位と桁**：傾きは「吸光度 / (mg/L)」。Fe-フェナントロリン錯体ならおよそ 0.2 L/mg。桁が 1000 倍ずれていたら、単位（mg/L と g/L など）の取り違えを疑う。
    - **R² の意味**：`score(X, y)` は学習に使ったデータでの R²。「予測精度」と書かれていたら、未知データでの値ではないと指摘する。
    - **逆算値を検算する**：求めた濃度を `model.predict([[c]])` に入れ、元の吸光度（0.500）に戻るか確かめる。

---

## 演習問題

**問1.** `X = np.array([[10], [20], [30], [40]])`、`y = np.array([25, 45, 62, 85])` で `LinearRegression` を学習させ、傾き・切片・R² を表示してください。

**問2.** 問1のモデルで、`x = 50` のときの予測値を表示してください。

**問3.** 1次元配列 `np.array([1, 2, 3])` をそのまま `fit` に渡すとエラーになります。`reshape(-1, 1)` で2次元に直してから使うと動くことを確認してください。

**問4.** 次は、検量線から試料の Fe 濃度を求めるために AI が書いたコードです。問題点を2つ指摘し、直してください。

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# 鉄(II)-1,10-フェナントロリン錯体の検量線（510 nm, 光路長 1 cm）
conc = np.array([0.0, 0.5, 1.0, 2.0, 3.0, 4.0])              # Fe 濃度 [mg/L]
absorb = np.array([0.001, 0.099, 0.200, 0.398, 0.601, 0.797])  # 吸光度

model = LinearRegression()
model.fit(conc, absorb)

sample_A = 0.500                     # 試料の吸光度
print("試料の Fe 濃度:", model.predict([[sample_A]]), "mg/L")
```

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
    from sklearn.linear_model import LinearRegression
    a = np.array([1, 2, 3])
    y = np.array([2.0, 4.1, 5.9])
    try:
        LinearRegression().fit(a, y)          # 1次元のまま渡す
    except ValueError as e:
        print("エラー:", str(e).splitlines()[0])
    print(a.reshape(-1, 1))                    # 2次元になる
    m = LinearRegression().fit(a.reshape(-1, 1), y)
    print("傾き:", round(m.coef_[0], 2))
    ```

    出力:
    ```text
    エラー: Expected 2D array, got 1D array instead:
    [[1]
     [2]
     [3]]
    傾き: 1.95
    ```
    `reshape(-1, 1)` で「3行1列」にすると動きます。この2次元の形が、scikit-learn の入力 X の正しい形です。

??? success "問4 の解答"
    **誤り1**：`conc` が1次元のまま `fit` に渡されています。実行すると `ValueError: Expected 2D array, got 1D array instead` で止まります。`reshape(-1, 1)` で2次元にします。

    **誤り2（化学的な誤り）**：このモデルは「濃度 → 吸光度」を学習しています。`model.predict([[0.500]])` は「濃度 0.500 mg/L のときの吸光度」を返すだけで、試料の濃度にはなりません（誤りを1だけ直して実行すると `[0.10001754]` が出て、単位を「mg/L」と書いてあるため気づきにくい）。吸光度から濃度を求めるには、A = a·c + b を c について解き、c = (A − b) / a と**逆算**します。

    確かめ方：求めた濃度をモデルに入れて、元の吸光度 0.500 に戻るかを見ます。

    ```python
    import numpy as np
    from sklearn.linear_model import LinearRegression

    conc = np.array([0.0, 0.5, 1.0, 2.0, 3.0, 4.0]).reshape(-1, 1)  # Fe 濃度 [mg/L]（2次元に）
    absorb = np.array([0.001, 0.099, 0.200, 0.398, 0.601, 0.797])    # 吸光度

    model = LinearRegression()
    model.fit(conc, absorb)                   # 濃度 → 吸光度 のモデル
    a = model.coef_[0]
    b = model.intercept_
    print("傾き:", round(a, 4), "(L/mg)")
    print("切片:", round(b, 4))

    sample_A = 0.500
    c = (sample_A - b) / a                    # 吸光度 → 濃度 は逆算する
    print("試料の Fe 濃度:", round(c, 2), "mg/L")
    print("検算（予測吸光度）:", round(model.predict([[c]])[0], 3))
    ```

    出力:
    ```text
    傾き: 0.1995 (L/mg)
    切片: 0.0003
    試料の Fe 濃度: 2.51 mg/L
    検算（予測吸光度）: 0.5
    ```
    傾き約 0.2 L/mg は、モル吸光係数 約 1.1×10⁴ L mol⁻¹ cm⁻¹ の Fe(II)-フェナントロリン錯体として妥当な値です。

---

## この回のまとめ

- scikit-learn の基本型：`model.fit(X, y)` → `model.predict(新X)` → `model.score(X, y)`。
- 入力 X は**2次元**（行＝サンプル、列＝特徴量）。`reshape(-1, 1)` で直せる。
- この型は**教師あり学習のモデルで共通**（変換器は fit / transform）。1つ覚えれば応用が利く。

### 次回予告

[第93回：回帰モデルで物性を予測する](lesson-93.md) では、分子の性質（沸点）を回帰で予測します。
