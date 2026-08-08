# 第27回　単位換算をまとめて処理する

!!! abstract "この回のゴール"
    - 温度・圧力などの単位換算を、配列で一括処理する
    - 換算を関数にまとめて再利用する
    - 所要時間の目安: 60分
    - 使うテーマ：**温度（℃/K/℉）・圧力（atm/kPa）**の換算

化学では単位換算が頻出します。NumPy を使えば、たくさんの値をまとめて換算できます。

`lesson27.py` を作りましょう。`import numpy as np` から。

---

## 1. 温度の換算

温度の配列を、ケルビン・華氏にまとめて変換します（第21回・第24回の応用）。

```python
import numpy as np

temps_C = np.array([0, 25, 100])

temps_K = temps_C + 273.15          # 摂氏 → ケルビン
temps_F = temps_C * 9 / 5 + 32      # 摂氏 → 華氏

print("K:", temps_K)
print("F:", temps_F)
```

出力:

```text
K: [273.15 298.15 373.15]
F: [ 32.  77. 212.]
```

---

## 2. 圧力の換算

1 atm = 101.325 kPa。圧力の配列をまとめて kPa に直します。

```python
import numpy as np

pressure_atm = np.array([1.0, 2.0, 0.5])
pressure_kPa = pressure_atm * 101.325

print(pressure_kPa.round(3))
```

出力:

```text
[101.325 202.65   50.662]
```

---

## 3. 換算を関数にまとめる（第6回の応用）

よく使う換算は関数にしておくと便利です。配列を渡せば、配列が返ります。

```python
import numpy as np

def celsius_to_kelvin(celsius):
    """摂氏[℃]をケルビン[K]に換算（配列でもスカラーでもOK）"""
    return celsius + 273.15

# スカラーでも配列でも同じ関数で動く
print(celsius_to_kelvin(25))                        # 298.15
print(celsius_to_kelvin(np.array([0, 50, 100])))    # 配列を一括換算
```

出力:

```text
298.15
[273.15 323.15 373.15]
```

!!! success "1つの関数がスカラーにも配列にも効く"
    NumPy のベクトル化のおかげで、`celsius_to_kelvin` は**1個の値でも、何千個の配列でも**同じコードで動きます。関数（第6回）とベクトル化（第24回）が組み合わさると、再利用しやすい部品になります。

---

## 演習問題

**問1.** 温度の配列 `temps_C = np.array([-40, 0, 37, 100])` を、ケルビンと華氏の両方にまとめて変換して表示してください（−40℃は摂氏と華氏が一致する有名な温度です。確かめてみましょう）。

**問2.** 圧力の配列 `p_atm = np.array([0.5, 1.0, 1.5, 2.0])` を kPa に換算してください（× 101.325）。

**問3.** 「kJ を kcal に換算する」関数 `kj_to_kcal(kj)` を作ってください（1 kcal = 4.184 kJ なので、kcal = kj / 4.184）。エネルギーの配列 `np.array([100, 200, 418.4])` kJ を換算して表示しましょう。

---

## 解答

??? success "問1 の解答"
    ```python
    import numpy as np
    temps_C = np.array([-40, 0, 37, 100])
    print("K:", temps_C + 273.15)
    print("F:", temps_C * 9 / 5 + 32)
    ```

    出力:
    ```text
    K: [233.15 273.15 310.15 373.15]
    F: [-40.   32.   98.6 212. ]
    ```
    −40℃ → −40℉。摂氏と華氏が一致する唯一の温度です。

??? success "問2 の解答"
    ```python
    import numpy as np
    p_atm = np.array([0.5, 1.0, 1.5, 2.0])
    print((p_atm * 101.325).round(3))
    ```

    出力:
    ```text
    [ 50.662 101.325 151.988 202.65 ]
    ```

??? success "問3 の解答"
    ```python
    import numpy as np
    def kj_to_kcal(kj):
        """キロジュール[kJ]をキロカロリー[kcal]に換算"""
        return kj / 4.184

    energy_kj = np.array([100, 200, 418.4])
    print(kj_to_kcal(energy_kj).round(3))
    ```

    出力:
    ```text
    [ 23.901  47.801 100.   ]
    ```

---

## この回のまとめ

- 温度・圧力などの単位換算は、配列に数式を適用するだけで一括処理できる。
- 換算を**関数**にまとめると、スカラーにも配列にも使い回せる。
- ベクトル化＋関数で、再利用しやすい部品を作れる。

### 次回予告

[第28回：濃度計算を配列で](lesson-28.md) では、モル濃度・希釈などの計算を配列でまとめて行います。
