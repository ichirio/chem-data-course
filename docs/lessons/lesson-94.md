# 第94回　分類モデル：活性／不活性を分ける

!!! abstract "この回のゴール"
    - 分類で「カテゴリ」を予測する
    - `LogisticRegression` で溶ける／溶けないを判定する
    - 学習した分類器で未知の分子を予測する
    - 所要時間の目安: 60分
    - 使うテーマ：**分子の水溶性の予測**

分類は「カテゴリを予測する」教師あり学習です。「溶ける/溶けない」「活性/不活性」のような**2択（または多択）**を当てます。

`ml94.py` を作りましょう。

```python
import pandas as pd
from sklearn.linear_model import LogisticRegression

# 分子量・LogP と、水に溶けるか（1=溶ける, 0=溶けない）
data = pd.DataFrame({
    "name":    ["methanol","ethanol","acetone","acetic_acid","glucose","glycerol",
                "benzene","toluene","hexane","octane","naphthalene","ibuprofen"],
    "MW":      [32, 46, 58, 60, 180, 92, 78, 92, 86, 114, 128, 206],
    "LogP":    [-0.7, -0.3, -0.2, -0.2, -3.0, -1.8, 2.1, 2.7, 3.9, 4.5, 3.3, 3.5],
    "soluble": [1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0],
})
```

---

## 1. 分類器を学習させる

`fit` / `predict` の型は回帰と同じ。モデルを `LogisticRegression`（分類の定番）に変えるだけです。

```python
X = data[["MW", "LogP"]]      # 特徴量（2列）
y = data["soluble"]           # 正解（0 or 1）

clf = LogisticRegression()
clf.fit(X, y)

print("訓練データでの正解率:", round(clf.score(X, y), 3))
```

出力:

```text
訓練データでの正解率: 1.0
```

正解率1.0＝訓練データを完璧に分類できました（このデータは溶ける/溶けないがLogPできれいに分かれているため）。

!!! note "正解率（accuracy）"
    分類の基本的な評価は**正解率**（全体のうち正しく分類できた割合）。回帰の R² にあたります。ただし正解率だけでは不十分な場合もあり、第96回でより詳しい評価を学びます。

---

## 2. 未知の分子を予測する

学習した分類器で、新しい分子が溶けるかを予測します。

```python
# [MW, LogP] の新しい分子2つ
new_mols = [[100, 3.0],    # 疎水的（LogPが高い）→ 溶けにくそう
            [50, -0.5]]    # 親水的（LogPが低い）→ 溶けそう

pred = clf.predict(new_mols)
print("予測:", pred)       # 1=溶ける, 0=溶けない
```

出力:

```text
予測: [0 1]
```

LogPが高い分子は「溶けない(0)」、低い分子は「溶ける(1)」と予測されました。化学の直感（極性が高いほど水に溶ける）と一致します。

---

## 3. 確率で予測する

`predict_proba` で「どれくらい自信があるか（確率）」も出せます。

```python
proba = clf.predict_proba(new_mols)
print(proba)     # 各行 [溶けない確率, 溶ける確率]
```

「0.9 の確率で溶ける」のように、判定の確信度が分かります。境界に近い（0.5付近の）分子は、判定が微妙ということです。

!!! success "分類のいろいろな用途"
    - 活性/不活性（創薬スクリーニング）
    - 毒性あり/なし（安全性評価）
    - 反応成功/失敗（プロセス最適化）
    
    「YES/NO を予測する」場面は化学に多く、分類が活躍します。`LogisticRegression` を `RandomForestClassifier` などに差し替えれば、より複雑な境界も学べます（型は同じ）。

---

## 演習問題

**問1.** 本文の溶解性データで `LogisticRegression` を学習させ、訓練データでの正解率を表示してください。

**問2.** 新しい分子 `[[70, 1.5], [40, -1.0], [150, 4.0]]` の溶解性を予測してください。それぞれの結果は化学的に妥当ですか？

**問3.** `predict_proba` で、問2の分子の「溶ける確率」を確認してください。最も自信を持って判定されたのはどれですか？

---

## 解答

??? success "問1 の解答"
    ```python
    from sklearn.linear_model import LogisticRegression
    X = data[["MW", "LogP"]]; y = data["soluble"]
    clf = LogisticRegression().fit(X, y)
    print("正解率:", round(clf.score(X, y), 3))
    ```

    出力:
    ```text
    正解率: 1.0
    ```

??? success "問2 の解答"
    ```python
    pred = clf.predict([[70, 1.5], [40, -1.0], [150, 4.0]])
    print(pred)
    ```

    出力:
    ```text
    [0 1 0]
    ```
    LogPが負の2つ目 `[40, -1.0]` だけ「溶ける(1)」、LogPが正の `[70, 1.5]` と `[150, 4.0]` は「溶けない(0)」。極性（LogP）が低いほど水に溶ける、という境界を学んだ結果です。

??? success "問3 の解答"
    ```python
    print(clf.predict_proba([[70, 1.5], [40, -1.0], [150, 4.0]]).round(3))
    ```
    各行の2列目が「溶ける確率」。LogPが極端な分子（−1.0 や 4.0）ほど、確率が0か1に近く、自信を持って判定されています。境界に近い分子ほど 0.5 付近になります。

---

## この回のまとめ

- 分類（`LogisticRegression`）でカテゴリ（0/1）を予測。
- 型は回帰と同じ：`fit` → `predict` → `score`（正解率）。
- `predict_proba` で判定の確信度（確率）が分かる。
- 活性/不活性・毒性・成功/失敗など、YES/NO 予測に活躍。

### 次回予告

[第95回：訓練・テスト分割と過学習](lesson-95.md) では、モデルの「本当の実力」を測るための、データの分け方と過学習の問題を学びます。
