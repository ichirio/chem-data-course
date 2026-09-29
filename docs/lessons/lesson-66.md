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

赤外分光（IR）のピーク（波数・強度・帰属）を DataFrame にします。値はアスピリンの主な吸収帯を丸めたものです（カルボン酸の O–H は実際には 2500〜3300 cm⁻¹ の幅広い吸収になります）。

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

# 3-フェニルプロパン酸 PhCH2CH2COOH の ¹H NMR（CDCl3、代表値を丸めた例）
nmr = pd.DataFrame({
    "shift_ppm":  [2.7, 3.0, 7.3, 11.0],       # 化学シフト [ppm]
    "integration": [2, 2, 5, 1],                # 積分値（プロトン数の比）
    "assignment": ["CH2COOH", "PhCH2", "aromatic H", "COOH"],
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
0        2.7            2     CH2COOH
1        3.0            2       PhCH2
2        7.3            5  aromatic H
3       11.0            1        COOH
全プロトン数: 10
   shift_ppm  integration  assignment
2        7.3            5  aromatic H
```

積分値の合計（10）から全プロトン数（C9H10O2 の H 10個）が、芳香族領域の抽出から芳香環の存在が読み取れます。

!!! success "スペクトル解析も"データ分析""
    ピークの抽出・帰属・集計は、すべて pandas の操作です。分光データを表にすれば、複数サンプルの比較、ピークの自動検出（数値データの場合）、レポート用の表作成が、これまでの技術でできます。第5部を使えばスペクトルの可視化も可能です。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「IR ピーク表（列：wavenumber [cm⁻¹]、assignment、intensity）の pandas DataFrame から、カルボニル領域 1650〜1800 cm⁻¹ のピークを抽出してください。
    条件は `(ir["wavenumber"] >= 1650) & (ir["wavenumber"] <= 1800)` の形で、`&` と括弧を使ってください。
    あわせて波数を波長 [µm] に換算した列（波長 = 10⁴ / 波数）を追加し、どちらも単位を列名に入れてください。」

!!! warning "AIの出力で確かめること"
    - **範囲の条件が `&`（かつ）になっているか**。`|`（または）だと全ピークが通ります。抽出後の行数を print し、期待（酢酸エチルならカルボニルは 1本）と比べます。
    - **単位換算の係数**：波数 [cm⁻¹] → 波長 [µm] は 10000 / 波数。1742 cm⁻¹ が約 5.74 µm になるかで確認します。
    - 帰属が教科書の範囲と合うか：C=O は 1650〜1800 cm⁻¹、O–H（カルボン酸）は 2500〜3300 cm⁻¹ の幅広い吸収、¹H NMR の芳香族 H は 6.5〜8.5 ppm、COOH は 10〜13 ppm。
    - NMR の積分値の合計が、想定した分子式の H の数と一致するか（AI が作った「例」のデータが実在の構造として成り立つか）。

---

## 演習問題

**問1.** 本文の `ir` データで、**強度が strong のピークだけ**を抽出してください（第34回のフィルタ）。

**問2.** 本文の `nmr` データを、化学シフト（shift_ppm）の**大きい順**に並べ替えてください（第36回）。

**問3.** `nmr` データで、sp3 炭素上の H が主に現れる 0〜5 ppm の領域のピークだけを抽出し、その積分値の合計を計算してください。

**問4.** 次は AI が書いた「酢酸エチルの IR ピーク表に波長の列を加え、カルボニル領域のピーク数を数える」コードです。問題点を指摘し、直してください。

```python
import pandas as pd

# 酢酸エチルの IR ピーク（液膜、代表値）
ir = pd.DataFrame({
    "wavenumber": [2983, 1742, 1374, 1243, 1047],    # 波数 [cm^-1]
    "assignment": ["C-H", "C=O", "C-H bend", "C-O", "C-O"],
})
ir["wavelength_um"] = (1000 / ir["wavenumber"]).round(2)    # 波長 [µm]
carbonyl = ir[(ir["wavenumber"] >= 1650) | (ir["wavenumber"] <= 1800)]
print(ir)
print("カルボニル領域のピーク数:", len(carbonyl))
```

AI の出力:

```text
   wavenumber assignment  wavelength_um
0        2983        C-H           0.34
1        1742        C=O           0.57
2        1374   C-H bend           0.73
3        1243        C-O           0.80
4        1047        C-O           0.96
カルボニル領域のピーク数: 5
```

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
    1        3.0            2       PhCH2
    0        2.7            2     CH2COOH
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
    0        2.7            2    CH2COOH
    1        3.0            2      PhCH2
    積分値の合計: 4
    ```

??? success "問4 の解答"
    誤りは2つです。

    1. **範囲の条件が「または」**：`|` だと「1650 以上 **または** 1800 以下」となり、すべての波数が当てはまります。範囲は `&`（かつ）で書きます。酢酸エチルの C=O は1本なのに 5 本と出た時点でおかしいと分かります。
    2. **波長の換算係数が1桁違う**：1 cm = 10⁴ µm なので、波長 [µm] = 10000 / 波数 [cm⁻¹]。中赤外は約 2.5〜25 µm なので、0.3〜1 µm（可視〜近赤外）という値はおかしいと気づけます。

    ```python
    import pandas as pd

    ir = pd.DataFrame({
        "wavenumber": [2983, 1742, 1374, 1243, 1047],    # 波数 [cm^-1]
        "assignment": ["C-H", "C=O", "C-H bend", "C-O", "C-O"],
    })
    ir["wavelength_um"] = (10000 / ir["wavenumber"]).round(2)   # 1 cm = 10^4 µm
    carbonyl = ir[(ir["wavenumber"] >= 1650) & (ir["wavenumber"] <= 1800)]
    print(ir)
    print("カルボニル領域のピーク数:", len(carbonyl))
    ```

    出力:
    ```text
       wavenumber assignment  wavelength_um
    0        2983        C-H           3.35
    1        1742        C=O           5.74
    2        1374   C-H bend           7.28
    3        1243        C-O           8.05
    4        1047        C-O           9.55
    カルボニル領域のピーク数: 1
    ```

---

## この回のまとめ

- 分光データ（IR/NMR/MS）は「ピークの表」。pandas でそのまま扱える。
- 特定領域のピーク抽出＝官能基・環境の同定（第34回のフィルタ）。
- 積分値の合計・並べ替えなど、第4部の操作がすべて使える。
- スペクトルの可視化には第5部を使う。

### 次回予告

[第67回：クロマトグラフィーデータの解析](lesson-67.md) では、GC / HPLC のピーク面積から組成（面積百分率）を求めます。
