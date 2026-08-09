# 第66回　分光データ（IR / NMRピーク）を整理する

!!! abstract "この回のゴール"
    - スペクトルの**ピークデータ**を pandas で扱う
    - 特定の領域のピークを抽出する（官能基の同定）
    - 強度で並べ替え・集計する
    - 所要時間の目安: 60分
    - 使うテーマ：**IR / NMR のピーク表**

分光分析（IR・NMR・MS…）の結果は「ピークの一覧」です。これは表データそのもの。第4部の pandas がそのまま活きます。

`lesson66.py` を作りましょう。

---

## 1. IR ピークを表にする

赤外分光（IR）のピーク（波数・強度・帰属）を DataFrame にします。

```python
import pandas as pd

ir = pd.DataFrame({
    "wavenumber":  [3000, 1750, 1690, 1600, 1450, 1200],   # 波数 [cm^-1]
    "assignment":  ["O-H/C-H", "C=O ester", "C=O acid", "C=C aromatic", "C-H bend", "C-O"],
    "intensity":   ["medium", "strong", "strong", "medium", "medium", "strong"],
})
print(ir)
```

出力:

```text
   wavenumber    assignment intensity
0        3000       O-H/C-H    medium
1        1750     C=O ester    strong
2        1690      C=O acid    strong
3        1600  C=C aromatic    medium
4        1450      C-H bend    medium
5        1200           C-O    strong
```

---

## 2. 特定領域のピークを抽出（官能基の同定）

カルボニル（C=O）は 1650〜1800 cm⁻¹ に出ます。その領域のピークだけを取り出します（第34回のフィルタ）。

```python
carbonyl = ir[(ir["wavenumber"] >= 1650) & (ir["wavenumber"] <= 1800)]
print(carbonyl)
```

出力:

```text
   wavenumber assignment intensity
1        1750  C=O ester    strong
2        1690   C=O acid    strong
```

「この波数域にピークがある＝カルボニル基がある」という、IR の解釈をプログラムで再現できます。

---

## 3. NMR ピークを整理する

核磁気共鳴（¹H NMR）は、化学シフト（ppm）・積分値・帰属で表せます。

```python
import pandas as pd

nmr = pd.DataFrame({
    "shift_ppm":  [2.1, 3.7, 7.3, 11.0],       # 化学シフト [ppm]
    "integration": [3, 2, 5, 1],                # 積分値（プロトン数の比）
    "assignment": ["CH3", "CH2", "aromatic H", "COOH"],
})

# 積分値の合計＝全プロトン数
print(nmr)
print("全プロトン数:", nmr["integration"].sum())

# 芳香族領域（6.5〜8.5 ppm）のピーク
aromatic = nmr[(nmr["shift_ppm"] >= 6.5) & (nmr["shift_ppm"] <= 8.5)]
print(aromatic)
```

出力:

```text
   shift_ppm  integration  assignment
0        2.1            3         CH3
1        3.7            2         CH2
2        7.3            5  aromatic H
3       11.0            1        COOH
全プロトン数: 11
   shift_ppm  integration  assignment
2        7.3            5  aromatic H
```

積分値の合計（11）から全プロトン数が、芳香族領域の抽出から芳香環の存在が読み取れます。

!!! success "スペクトル解析も"データ分析""
    ピークの抽出・帰属・集計は、すべて pandas の操作です。分光データを表にすれば、複数サンプルの比較、ピークの自動検出（数値データの場合）、レポート用の表作成が、これまでの技術でできます。第5部を使えばスペクトルの可視化も可能です。

---

## 演習問題

**問1.** 本文の `ir` データで、**強度が strong のピークだけ**を抽出してください（第34回のフィルタ）。

**問2.** 本文の `nmr` データを、化学シフト（shift_ppm）の**大きい順**に並べ替えてください（第36回）。

**問3.** `nmr` データで、脂肪族領域（0〜5 ppm）のピークだけを抽出し、その積分値の合計を計算してください。

---

## 解答

??? success "問1 の解答"
    ```python
    strong = ir[ir["intensity"] == "strong"]
    print(strong)
    ```

    出力:
    ```text
       wavenumber assignment intensity
    1        1750  C=O ester    strong
    2        1690   C=O acid    strong
    5        1200        C-O    strong
    ```

??? success "問2 の解答"
    ```python
    print(nmr.sort_values("shift_ppm", ascending=False))
    ```

    出力:
    ```text
       shift_ppm  integration  assignment
    3       11.0            1        COOH
    2        7.3            5  aromatic H
    1        3.7            2         CH2
    0        2.1            3         CH3
    ```

??? success "問3 の解答"
    ```python
    aliphatic = nmr[(nmr["shift_ppm"] >= 0) & (nmr["shift_ppm"] <= 5)]
    print(aliphatic)
    print("積分値の合計:", aliphatic["integration"].sum())
    ```

    出力:
    ```text
       shift_ppm  integration assignment
    0        2.1            3        CH3
    1        3.7            2        CH2
    積分値の合計: 5
    ```

---

## この回のまとめ

- 分光データ（IR/NMR/MS）は「ピークの表」。pandas でそのまま扱える。
- 特定領域のピーク抽出＝官能基・環境の同定（第34回のフィルタ）。
- 積分値の合計・並べ替えなど、第4部の操作がすべて使える。
- スペクトルの可視化には第5部を使う。

### 次回予告

[第67回：クロマトグラフィーデータの解析](lesson-67.md) では、GC / HPLC のピーク面積から組成（面積百分率）を求めます。
