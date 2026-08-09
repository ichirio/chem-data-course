# 第96回　モデル評価（決定係数・精度・交差検証）

!!! abstract "この回のゴール"
    - 回帰の評価指標（R²・MAE）を使う
    - 分類の評価（正解率・混同行列）を使う
    - **交差検証**で安定した評価をする
    - 所要時間の目安: 60分
    - 使うテーマ：**モデルの多面的な評価**

「R²だけ」「正解率だけ」では、モデルの実力を見誤ることがあります。複数の指標で多面的に評価しましょう。

`ml96.py` を作りましょう。

---

## 1. 回帰の評価：R² と MAE

```python
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score, mean_absolute_error

n = np.array([1,2,3,4,5,6,7,8,9,10]).reshape(-1,1)
bp = np.array([-161.5,-88.6,-42.1,-0.5,36.1,68.7,98.4,125.7,150.8,174.1])
Xtr, Xte, ytr, yte = train_test_split(n, bp, test_size=0.3, random_state=42)

model = LinearRegression().fit(Xtr, ytr)
pred = model.predict(Xte)

print("R^2:", round(r2_score(yte, pred), 4))
print("MAE:", round(mean_absolute_error(yte, pred), 2))
```

出力:

```text
R^2: 0.9882
MAE: 9.23
```

- **R²（決定係数）**：1に近いほど良い（説明できた割合）。
- **MAE（平均絶対誤差）**：予測が平均どれだけずれたか（単位つき、小さいほど良い）。この場合「平均9.2℃ずれる」。

R² は「相対的な良さ」、MAE は「実際の誤差の大きさ」を表します。両方見ると理解が深まります。

---

## 2. 分類の評価：混同行列

分類では、正解率だけでなく**混同行列**（どう間違えたか）を見ます。

```python
from sklearn.metrics import confusion_matrix
# clf は第94回の溶解性分類器（学習済みとする）
cm = confusion_matrix(y, clf.predict(X))
print(cm)
```

出力:

```text
[[6 0]
 [0 6]]
```

混同行列の読み方（2×2の場合）:

```text
              予測:0   予測:1
実際:0(不溶)    6       0      ← 不溶を正しく6個
実際:1(可溶)    0       6      ← 可溶を正しく6個
```

対角線（左上・右下）が「正解」、それ以外が「間違い」。この例は全問正解（間違い0）です。

!!! note "なぜ混同行列が大事か"
    正解率が高くても、「見逃し（活性を不活性と誤判定）」が多いと危険な場合があります。混同行列を見れば、**どの種類の間違いが多いか**が分かります。医薬・安全性評価では特に重要です。

---

## 3. 交差検証：安定した評価

train_test_split は「1回の分け方」に結果が左右されます。**交差検証**は、分け方を何通りも変えて平均を取り、安定した評価を得ます。

```python
from sklearn.model_selection import cross_val_score, KFold

kf = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(LinearRegression(), n, bp, cv=kf, scoring="r2")

print("各分割の R^2:", scores.round(3))
print("平均 R^2:", round(scores.mean(), 3))
```

出力:

```text
各分割の R^2: [0.994 0.875 0.988 0.864 0.888]
平均 R^2: 0.922
```

5通りの分け方でそれぞれ評価し、平均0.922。1回のテストより信頼できる評価です。

!!! success "評価は多面的に"
    - 回帰：R²（相対）＋ MAE（実誤差）
    - 分類：正解率 ＋ 混同行列
    - 安定性：交差検証で平均を取る
    
    1つの数字を鵜呑みにせず、複数の指標で見る——これが正しいモデル評価です。データが少ないほど、交差検証の価値が高まります。

---

## 演習問題

**問1.** アルカンデータを `random_state=42, test_size=0.3` で分け、線形回帰のテスト R² と MAE を表示してください。

**問2.** 第94回の溶解性分類器の混同行列を `confusion_matrix` で表示してください。間違いはいくつありますか？

**問3.** `cross_val_score` で、アルカンデータの線形回帰を5分割交差検証（`KFold(shuffle=True, random_state=42)`）し、各スコアと平均を表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    import numpy as np
    from sklearn.model_selection import train_test_split
    from sklearn.linear_model import LinearRegression
    from sklearn.metrics import r2_score, mean_absolute_error
    n = np.array([1,2,3,4,5,6,7,8,9,10]).reshape(-1,1)
    bp = np.array([-161.5,-88.6,-42.1,-0.5,36.1,68.7,98.4,125.7,150.8,174.1])
    Xtr,Xte,ytr,yte = train_test_split(n, bp, test_size=0.3, random_state=42)
    m = LinearRegression().fit(Xtr, ytr)
    print("R^2:", round(r2_score(yte, m.predict(Xte)), 4))
    print("MAE:", round(mean_absolute_error(yte, m.predict(Xte)), 2))
    ```

    出力:
    ```text
    R^2: 0.9882
    MAE: 9.23
    ```

??? success "問2 の解答"
    ```python
    from sklearn.metrics import confusion_matrix
    print(confusion_matrix(y, clf.predict(X)))
    ```

    出力:
    ```text
    [[6 0]
     [0 6]]
    ```
    対角線以外が0なので、**間違いは0個**（全問正解）です。

??? success "問3 の解答"
    ```python
    from sklearn.model_selection import cross_val_score, KFold
    kf = KFold(n_splits=5, shuffle=True, random_state=42)
    scores = cross_val_score(LinearRegression(), n, bp, cv=kf, scoring="r2")
    print(scores.round(3))
    print("平均:", round(scores.mean(), 3))
    ```

    出力:
    ```text
    [0.994 0.875 0.988 0.864 0.888]
    平均: 0.922
    ```

---

## この回のまとめ

- 回帰：**R²**（相対的な良さ）＋ **MAE**（実際の誤差、単位つき）。
- 分類：正解率 ＋ **混同行列**（どう間違えたか）。
- **交差検証**（`cross_val_score` ＋ `KFold`）で、分け方に依存しない安定した評価。
- 1つの指標を鵜呑みにせず、多面的に評価する。

### 次回予告

[第97回：特徴量エンジニアリング](lesson-97.md) では、RDKit の分子記述子を特徴量にして、機械学習の入力を作ります。第6部と第8部が合流します。
