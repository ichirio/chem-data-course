# 第67回　クロマトグラフィーデータの解析

!!! abstract "この回のゴール"
    - GC / HPLC のピークデータを扱う
    - **面積百分率（area%）**で組成を求める
    - 保持時間で成分を整理する
    - 所要時間の目安: 60分
    - 使うテーマ：**混合物の組成分析**

クロマトグラフィー（GC・HPLC）は、混合物を分離して各成分を検出します。結果は「保持時間とピーク面積」の表。**面積の比＝おおよその組成比**として使えます。

`lesson67.py` を作りましょう。

---

## 1. クロマトグラムのピークを表にする

```python
import pandas as pd

peaks = pd.DataFrame({
    "compound":      ["A", "B", "C", "D"],
    "retention_min": [2.3, 3.8, 5.1, 7.6],    # 保持時間 [分]
    "area":          [1500, 8200, 4300, 900],  # ピーク面積
})
print(peaks)
```

出力:

```text
  compound  retention_min  area
0        A            2.3  1500
1        B            3.8  8200
2        C            5.1  4300
3        D            7.6   900
```

---

## 2. 面積百分率で組成を求める

各ピークの面積が全体に占める割合＝**面積百分率**。混合物の組成のよい近似です（第37回の列計算、第43回の正規化の応用）。

```python
total = peaks["area"].sum()
peaks["area_pct"] = (peaks["area"] / total * 100).round(2)
print(peaks)
print("合計:", peaks["area_pct"].sum())
```

出力:

```text
  compound  retention_min  area  area_pct
0        A            2.3  1500     10.07
1        B            3.8  8200     55.03
2        C            5.1  4300     28.86
3        D            7.6   900      6.04
```

（合計はちょうど100%）成分Bが約55%と最も多い、と分かりました。

---

## 3. 主成分を見つける・保持時間順に並べる

```python
# 面積百分率の大きい順（主成分を上に）
print(peaks.sort_values("area_pct", ascending=False))

# 最も多い成分
main = peaks.loc[peaks["area_pct"].idxmax()]
print(f"\n主成分: {main['compound']}（{main['area_pct']}%）")
```

出力:

```text
  compound  retention_min  area  area_pct
1        B            3.8  8200     55.03
2        C            5.1  4300     28.86
0        A            2.3  1500     10.07
3        D            7.6   900      6.04

主成分: B（55.03%）
```

!!! success "分析化学とデータ分析の合流"
    面積百分率の計算、主成分の同定、複数サンプルの比較——すべて第4部の pandas 操作です。純度検定（主成分が何%以上か）や、反応前後の組成変化の追跡なども、同じ手法でできます。第5部でクロマトグラムのピークを棒グラフにすれば、視覚的な報告書にもなります。

!!! note "面積百分率の注意"
    面積百分率は「各成分の検出器応答が同じ」と仮定した近似です。厳密な定量には、成分ごとの**検量線**（第50回）や補正係数が必要です。目安として使い、正確な定量とは区別しましょう。

---

## 演習問題

**問1.** 本文の `peaks` データで、面積百分率（area_pct）の列を計算して追加し、表を表示してください。

**問2.** 面積百分率が **20%以上**の成分（主成分）だけを抽出してください（微量成分を除く）。

**問3.** 保持時間（retention_min）の**早い順**（小さい順）に並べ替えて表示してください（溶出順になります）。

---

## 解答

??? success "問1 の解答"
    ```python
    total = peaks["area"].sum()
    peaks["area_pct"] = (peaks["area"] / total * 100).round(2)
    print(peaks)
    ```

    出力:
    ```text
      compound  retention_min  area  area_pct
    0        A            2.3  1500     10.07
    1        B            3.8  8200     55.03
    2        C            5.1  4300     28.86
    3        D            7.6   900      6.04
    ```

??? success "問2 の解答"
    ```python
    major = peaks[peaks["area_pct"] >= 20]
    print(major)
    ```

    出力:
    ```text
      compound  retention_min  area  area_pct
    1        B            3.8  8200     55.03
    2        C            5.1  4300     28.86
    ```
    20%未満の A（10.07%）と D（6.04%）が除かれました。

??? success "問3 の解答"
    ```python
    print(peaks.sort_values("retention_min"))
    ```

    出力:
    ```text
      compound  retention_min  area  area_pct
    0        A            2.3  1500     10.07
    1        B            3.8  8200     55.03
    2        C            5.1  4300     28.86
    3        D            7.6   900      6.04
    ```
    保持時間が早い順＝カラムから早く溶出した順です。

---

## この回のまとめ

- クロマトデータは「保持時間・面積」の表。
- **面積百分率** = 各面積 ÷ 総面積 × 100 で組成の近似。
- `idxmax()` で主成分を特定、`sort_values` で溶出順に整理。
- 厳密な定量には検量線が必要（面積百分率は目安）。

### 次回予告

[第68回：周期表データを分析する（mendeleev）](lesson-68.md) では、元素の性質データを扱えるライブラリで、周期表の傾向を分析します。
