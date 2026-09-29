# 第96回　モデル評価（決定係数・精度・交差検証）

!!! abstract "この回のゴール"
    - 回帰の評価指標（R²・MAE・RMSE）を使う
    - 分類の評価（正解率・混同行列・ROC AUC）を使う
    - **交差検証**で安定した評価をする
    - **データ漏洩**を防ぐ（前処理も交差検証の中で行う）
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
from sklearn.metrics import r2_score, mean_absolute_error, root_mean_squared_error

n = np.array([1,2,3,4,5,6,7,8,9,10]).reshape(-1,1)
bp = np.array([-161.5,-88.6,-42.1,-0.5,36.1,68.7,98.4,125.7,150.8,174.1])
Xtr, Xte, ytr, yte = train_test_split(n, bp, test_size=0.3, random_state=42)

model = LinearRegression().fit(Xtr, ytr)
pred = model.predict(Xte)

print("R^2:", round(r2_score(yte, pred), 4))
print("MAE:", round(mean_absolute_error(yte, pred), 2))
print("RMSE:", round(root_mean_squared_error(yte, pred), 2))
```

出力:

```text
R^2: 0.9882
MAE: 9.23
RMSE: 10.77
```

- **R²（決定係数）**：1に近いほど良い（説明できた割合）。
- **MAE（平均絶対誤差）**：予測が平均どれだけずれたか（単位つき、小さいほど良い）。この場合「平均9.2℃ずれる」。
- **RMSE（二乗平均平方根誤差）**：誤差を2乗して平均し、平方根をとったもの（単位つき）。大きく外れた予測を重く数えるので、MAE より目立って大きければ「大外れ」が混じっています。古い資料や AI がよく書く `mean_squared_error(..., squared=False)` は scikit-learn 1.6 で削除され、現行版では `TypeError` になります。`root_mean_squared_error` を使いましょう。

R² は「相対的な良さ」、MAE は「実際の誤差の大きさ」を表します。両方見ると理解が深まります。

なお、このテストは C2・C6・C9 の**わずか3点**です。3点の R² はたまたま良い分け方だった可能性があります（次の交差検証で確かめます）。

---

## 2. 分類の評価：混同行列

分類では、正解率だけでなく**混同行列**（どう間違えたか）を見ます。

```python
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import StratifiedKFold, cross_val_predict
from sklearn.metrics import confusion_matrix, accuracy_score
# data, X, y は第94回の溶解性データ
cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)   # 各分割で 0/1 の比を保つ
pred_c = cross_val_predict(LogisticRegression(), X, y, cv=cv)     # 各分子を「その分子を学習に使っていないモデル」で予測
print(confusion_matrix(y, pred_c))
print("正解率:", round(accuracy_score(y, pred_c), 3))
print("誤分類:", data.loc[pred_c != y, "name"].tolist())
```

出力:

```text
[[6 0]
 [1 5]]
正解率: 0.917
誤分類: ['glucose']
```

混同行列の読み方（2×2の場合）:

```text
              予測:0   予測:1
実際:0(不溶)    6       0      ← 不溶を正しく6個
実際:1(可溶)    1       5      ← 可溶のうち1個を「不溶」と誤判定
```

対角線（左上・右下）が「正解」、それ以外が「間違い」。第94回では学習データで測って正解率 1.0 でしたが、学習に使っていない分子で測ると glucose を「溶けない」と誤判定しました。glucose（MW 180）は、溶ける分子の中で飛び抜けて大きく、glucose をテスト側に回したとき、訓練側には「大きくて水に溶ける分子」が1つもないためです。学習データに似た分子がないと予測は外れる——**適用範囲**の問題です（第100回）。

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

5通りの分け方でそれぞれ評価し、平均0.922。1回のテストより信頼できる評価です。ただし、10点を5分割すると各テストは**たった2点**です。R² は「テスト点のばらつき」を基準にした比なので、2点では不安定になります（0.864〜0.994）。少ないデータでは、全分子の予測をまとめてから誤差を計算するのが確実です。

```python
from sklearn.model_selection import cross_val_predict
pred_cv = cross_val_predict(LinearRegression(), n, bp, cv=kf)   # 各分子を「自分を学習に使っていないモデル」で予測
print("交差検証 MAE:", round(mean_absolute_error(bp, pred_cv), 1), "℃")
```

出力:

```text
交差検証 MAE: 17.7 ℃
```

§1 の1回の分割では MAE 9.23 ℃でしたが、交差検証では 17.7 ℃。§1 の分け方は「たまたま当たりやすい3点」だったのです。

---

## 4. データ漏洩：前処理も交差検証の「中」で

交差検証やテストの前に、**全データを見て**標準化・記述子の選択などをすると、テストの情報が学習にもれ込みます（**データ漏洩**）。極端な例で確かめます。活性値とまったく無関係な「記述子」（乱数）2000 個から、活性と相関の高い10個を選んでモデルを作ります。

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_selection import SelectKBest, f_regression
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import cross_val_score, KFold

rng = np.random.default_rng(0)
X_rand = rng.normal(size=(30, 2000))   # 30化合物 × 2000個の「記述子」（すべて乱数）
y_rand = rng.normal(size=30)           # 活性値（これも乱数。X とは無関係）
kf_r = KFold(n_splits=5, shuffle=True, random_state=0)

# 誤り：全データを見て「活性と相関の高い記述子10個」を選んでから交差検証
X_sel = SelectKBest(f_regression, k=10).fit_transform(X_rand, y_rand)
leak = cross_val_score(LinearRegression(), X_sel, y_rand, cv=kf_r, scoring="r2")
print("漏洩あり 平均 R^2:", round(leak.mean(), 2))

# 正しい：選択もパイプラインに入れ、各分割の訓練データだけで選ぶ
pipe = make_pipeline(SelectKBest(f_regression, k=10), LinearRegression())
ok = cross_val_score(pipe, X_rand, y_rand, cv=kf_r, scoring="r2")
print("Pipeline 平均 R^2:", round(ok.mean(), 2))
```

出力:

```text
漏洩あり 平均 R^2: 0.54
Pipeline 平均 R^2: -1.18
```

（変数名を `X_rand`・`y_rand` にしているのは、§2 の溶解性データの `X`・`y` を上書きしないためです。）

X_rand と y_rand は無関係なので、本当の実力は「予測できない」（R² は0以下）です。ところが全データで記述子を選ぶと、テストの分子にたまたま合う記述子が選ばれ、R² 0.54 の「それなりのモデル」に見えてしまいます。

!!! warning "前処理はモデルの一部"
    標準化（`StandardScaler`）・記述子の選択・欠損値の補完など、**データから何かを計算する前処理**はすべて `make_pipeline(前処理, モデル)` にまとめ、パイプラインごと `cross_val_score` や `fit` に渡します。そうすれば、分割のたびに訓練データだけで前処理が計算されます。

---

## 5. クラスの偏りと ROC AUC

創薬スクリーニングでは、活性化合物は全体の数 % しかないのが普通です。このとき正解率は当てになりません。

```python
import numpy as np
from sklearn.metrics import accuracy_score, recall_score, balanced_accuracy_score, confusion_matrix

y_true = np.array([1] * 5 + [0] * 95)     # 100化合物中、活性(1)は5個だけ
y_pred = np.zeros(100, dtype=int)         # 「全部 不活性」と答えるだけのモデル
print("正解率:", accuracy_score(y_true, y_pred))
print("再現率（活性の検出率）:", recall_score(y_true, y_pred))
print("バランス正解率:", balanced_accuracy_score(y_true, y_pred))
print(confusion_matrix(y_true, y_pred))
```

出力:

```text
正解率: 0.95
再現率（活性の検出率）: 0.0
バランス正解率: 0.5
[[95  0]
 [ 5  0]]
```

活性を1つも見つけていないのに正解率は 0.95。**再現率**（実際の活性のうち、見つけられた割合）や**バランス正解率**（各クラスの正解率の平均）を合わせて見ます。偏ったデータを分割するときは `train_test_split(..., stratify=y)` や `StratifiedKFold` で、各分割の 0/1 の比をそろえます。

**ROC AUC** は、「確率」の順位の良さを 0〜1 で表す指標です。ランダムに選んだ「1」の分子が、ランダムに選んだ「0」の分子より高い確率をもらえる割合で、0.5 ならでたらめ、1.0 なら完璧です。しきい値（0.5）の決め方に左右されないのが利点です。

```python
from sklearn.metrics import roc_auc_score
# 第94回の溶解性データ。確率も交差検証で出す
proba = cross_val_predict(LogisticRegression(), X, y, cv=cv, method="predict_proba")[:, 1]
print("ROC AUC:", round(roc_auc_score(y, proba), 3))
```

出力:

```text
ROC AUC: 0.889
```

`predict_proba` の**2列目**（クラス1＝溶ける確率）を渡していることに注意します。

!!! success "評価は多面的に"
    - 回帰：R²（相対）＋ MAE・RMSE（実誤差、単位つき）
    - 分類：正解率 ＋ 混同行列 ＋ 再現率・ROC AUC（クラスが偏っているときは特に）
    - 安定性：交差検証で、学習に使っていない予測から評価する
    - 前処理はパイプラインに入れ、交差検証の「中」で行う
    
    1つの数字を鵜呑みにせず、複数の指標で見る——これが正しいモデル評価です。データが少ないほど、交差検証の価値が高まります。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「化合物 40 個の記述子（DataFrame `X`、12 列）と活性値 pIC50（Series `y`）があります。
    StandardScaler と Ridge を make_pipeline でまとめ、KFold(n_splits=5, shuffle=True, random_state=0) で交差検証してください。
    cross_val_predict で全化合物の予測を作り、R²・MAE・RMSE（root_mean_squared_error を使用）を表示し、予測 vs 実測の散布図を描いてください。
    標準化や特徴量選択は必ずパイプラインの中で行ってください。」

!!! warning "AIの出力で確かめること"
    - **前処理の位置**：`StandardScaler().fit_transform(X)` や `SelectKBest(...).fit_transform(X, y)` が交差検証・分割の**前**に全データで実行されていたらデータ漏洩。`make_pipeline(...)` の中に入っているか見る。
    - **符号**：`scoring="neg_mean_absolute_error"` の結果は負の値。「MAE: -17.7」をそのまま報告していないか。
    - **非推奨・削除された API**：`mean_squared_error(..., squared=False)` は現行版で `TypeError`。`root_mean_squared_error` になっているか。
    - **評価に使ったデータ**：混同行列や R² が、学習に使ったデータで計算されていないか。`confusion_matrix(y, clf.predict(X))` で `clf` が全データで fit 済みなら、それは訓練成績。
    - **クラスの偏り**：`y.value_counts()` を出させ、偏っていれば正解率ではなく再現率・バランス正解率・ROC AUC を見る。ROC AUC に `predict_proba(...)[:, 1]`（クラス1の確率）を渡しているか。

---

## 演習問題

**問1.** アルカンデータを `random_state=42, test_size=0.3` で分け、線形回帰のテスト R² と MAE を表示してください。

**問2.** 第94回の溶解性データで、`StratifiedKFold(n_splits=3, shuffle=True, random_state=42)` による交差検証予測の混同行列を表示してください。間違いはいくつありますか？学習データで測った場合（第94回）と比べましょう。

**問3.** `cross_val_score` で、アルカンデータの線形回帰を5分割交差検証（`KFold(shuffle=True, random_state=42)`）し、各スコアと平均を表示してください。

**問4.** 次は、交差検証の MAE と RMSE を求めるために AI が書いたコードです。実行すると次のように表示され、途中でエラーになりました。問題点を2つ指摘し、直してください。

```python
import numpy as np
from sklearn.model_selection import cross_val_score, KFold
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

n = np.array([1,2,3,4,5,6,7,8,9,10]).reshape(-1,1)
bp = np.array([-161.5,-88.6,-42.1,-0.5,36.1,68.7,98.4,125.7,150.8,174.1])
kf = KFold(n_splits=5, shuffle=True, random_state=42)

scores = cross_val_score(LinearRegression(), n, bp, cv=kf, scoring="neg_mean_absolute_error")
print("交差検証 MAE:", round(scores.mean(), 1), "℃")

model = LinearRegression().fit(n, bp)
print("RMSE:", round(mean_squared_error(bp, model.predict(n), squared=False), 1), "℃")
```

出力:

```text
交差検証 MAE: -17.7 ℃
Traceback (most recent call last):
  ...
TypeError: got an unexpected keyword argument 'squared'
```

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
    from sklearn.linear_model import LogisticRegression
    from sklearn.model_selection import StratifiedKFold, cross_val_predict
    from sklearn.metrics import confusion_matrix
    cv = StratifiedKFold(n_splits=3, shuffle=True, random_state=42)
    pred_c = cross_val_predict(LogisticRegression(), X, y, cv=cv)
    print(confusion_matrix(y, pred_c))
    ```

    出力:
    ```text
    [[6 0]
     [1 5]]
    ```
    間違いは**1個**（左下：溶ける分子を「溶けない」と誤判定。glucose）。学習データで測ると `[[6 0] [0 6]]` の全問正解でしたが、それは丸暗記も含んだ成績です。

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

??? success "問4 の解答"
    **誤り1（符号）**：`scoring="neg_mean_absolute_error"` は「大きいほど良い」にそろえるため、MAE に**マイナスを付けた値**を返します。誤差が負になることはないので、「−17.7 ℃」はおかしいと気づけます。`-scores.mean()` で符号を戻します。

    **誤り2（削除された API と評価データ）**：`mean_squared_error(..., squared=False)` は scikit-learn 1.6 で削除され、1.8 では `TypeError` になります。`root_mean_squared_error` を使います。また、このコードは全データで学習したモデルを同じデータで評価しており、交差検証の MAE と比べられません。`cross_val_predict` で交差検証の予測から RMSE を出します。

    ```python
    import numpy as np
    from sklearn.model_selection import cross_val_score, cross_val_predict, KFold
    from sklearn.linear_model import LinearRegression
    from sklearn.metrics import root_mean_squared_error

    n = np.array([1,2,3,4,5,6,7,8,9,10]).reshape(-1,1)
    bp = np.array([-161.5,-88.6,-42.1,-0.5,36.1,68.7,98.4,125.7,150.8,174.1])
    kf = KFold(n_splits=5, shuffle=True, random_state=42)

    scores = cross_val_score(LinearRegression(), n, bp, cv=kf, scoring="neg_mean_absolute_error")
    print("交差検証 MAE:", round(-scores.mean(), 1), "℃")     # 符号を戻す

    pred = cross_val_predict(LinearRegression(), n, bp, cv=kf)  # 各分子を「学習に使っていないモデル」で予測
    print("交差検証 RMSE:", round(root_mean_squared_error(bp, pred), 1), "℃")
    ```

    出力:
    ```text
    交差検証 MAE: 17.7 ℃
    交差検証 RMSE: 23.4 ℃
    ```
    RMSE が MAE よりかなり大きいのは、メタン（C1）や C10 のような大外れが含まれているためです。

---

## この回のまとめ

- 回帰：**R²**（相対的な良さ）＋ **MAE**（実際の誤差、単位つき）。
- 分類：正解率 ＋ **混同行列**（どう間違えたか）＋ 再現率・**ROC AUC**。クラスが偏っていたら正解率は当てにならない。
- 標準化・特徴量選択は **Pipeline** に入れ、交差検証の中で行う（データ漏洩を防ぐ）。
- **交差検証**（`cross_val_score` ＋ `KFold`）で、分け方に依存しない安定した評価。
- 1つの指標を鵜呑みにせず、多面的に評価する。

### 次回予告

[第97回：特徴量エンジニアリング](lesson-97.md) では、RDKit の分子記述子を特徴量にして、機械学習の入力を作ります。第6部と第8部が合流します。
