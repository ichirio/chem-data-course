# 第82回　カイ二乗検定：カテゴリデータ

!!! abstract "この回のゴール"
    - カテゴリデータ（成功/失敗など）の偏りを調べる
    - 分割表（クロス集計）を作る
    - `chisq.test()` で独立性を検定する
    - 所要時間の目安: 60分
    - 使うテーマ：**溶媒による反応成功率の違い**

「溶媒によって反応の成功率が違うか？」——数値ではなく**カテゴリ（成功/失敗）**の偏りを調べるのが**カイ二乗検定**です。

`stats82.R` を作りましょう。

---

## 1. 分割表を作る

「溶媒（水/エタノール）× 結果（成功/失敗）」の件数を、行列（分割表）にします。

```r
tbl <- matrix(
  c(18, 2,    # 水: 成功18・失敗2
    12, 8),   # エタノール: 成功12・失敗8
  nrow = 2, byrow = TRUE,
  dimnames = list(Solvent = c("water", "ethanol"), Result = c("success", "fail"))
)
print(tbl)
```

出力:

```text
         Result
Solvent   success fail
  water        18    2
  ethanol      12    8
```

水は20回中18回成功、エタノールは20回中12回成功。**見かけ上は水の方が成功率が高い**ですが、これは偶然の範囲でしょうか？

---

## 2. カイ二乗検定を実行する

`chisq.test()` で、「溶媒と結果に関連があるか（独立でないか）」を検定します。

```r
chisq.test(tbl, correct = FALSE)
```

出力:

```text
	Pearson's Chi-squared test

data:  tbl
X-squared = 4.8, df = 1, p-value = 0.02846
```

読み方:

- **p-value = 0.02846** … 0.05 より小さい → **有意**。「溶媒と成功率には関連がある」と言えます。
- **X-squared = 4.8** … カイ二乗統計量（観測と期待のズレの大きさ）。

つまり、水とエタノールで成功率が違うのは偶然ではない、と結論できます。

!!! note "`correct = FALSE` について"
    2×2 の表では、R は既定で「イェーツの連続性補正」を行います。教科書どおりの計算にするため、ここでは `correct = FALSE` を指定しています。サンプルが少ないときは補正あり（既定）が推奨されます。

---

## 3. 期待度数を確認する

「もし関連がなければ何回成功するはずか（期待度数）」も見られます。

```r
result <- chisq.test(tbl, correct = FALSE)
result$expected      # 期待度数
```

出力:

```text
         Result
Solvent   success fail
  water        15    5
  ethanol      15    5
```

「関連がなければ両溶媒とも15成功・5失敗のはず」。実際の水18・エタノール12は、この期待から大きくずれている——だから有意、というわけです。

!!! warning "期待度数が小さいとき"
    どこかのマスの期待度数が5未満だと、カイ二乗検定は不正確になります。その場合は**フィッシャーの正確検定**（`fisher.test`）を使います。

---

## 演習問題

**問1.** 触媒A・Bの反応結果 `tbl2 <- matrix(c(25, 5, 15, 15), nrow=2, byrow=TRUE, dimnames=list(Catalyst=c("A","B"), Result=c("success","fail")))` を作って表示してください。

**問2.** 問1の表に `chisq.test(tbl2, correct = FALSE)` を適用し、触媒と成功率に関連があるか調べてください。p値はいくつですか？

**問3.** 問1の期待度数（`$expected`）を表示し、実際の値とどれくらいずれているか見てください。

---

## 解答

??? success "問1 の解答"
    ```r
    tbl2 <- matrix(c(25, 5, 15, 15), nrow = 2, byrow = TRUE,
                   dimnames = list(Catalyst = c("A", "B"), Result = c("success", "fail")))
    print(tbl2)
    ```

    出力:
    ```text
          Result
    Catalyst success fail
         A        25    5
         B        15   15
    ```

??? success "問2 の解答"
    ```r
    chisq.test(tbl2, correct = FALSE)
    ```

    出力:
    ```text
    X-squared = 7.5, df = 1, p-value = 0.00617
    ```
    p値 ≈ 0.006 で有意。触媒Aの方が成功率が高いのは偶然ではない、と言えます。

??? success "問3 の解答"
    ```r
    chisq.test(tbl2, correct = FALSE)$expected
    ```

    出力:
    ```text
          Result
    Catalyst success fail
         A        20   10
         B        20   10
    ```
    関連がなければ両触媒とも20成功・10失敗のはず。実際のA(25)・B(15)はここから大きくずれています。

---

## この回のまとめ

- カテゴリデータの偏りは**カイ二乗検定**（`chisq.test`）。
- `matrix(..., dimnames=...)` で分割表（クロス集計）を作る。
- **p < 0.05 なら「関連あり（独立でない）」**。
- 期待度数が5未満なら `fisher.test`（フィッシャーの正確検定）を使う。

### 次回予告

[第83回：相関と回帰](lesson-83.md) では、2つの量の関係を調べる相関係数と回帰直線を学びます。
