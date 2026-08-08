# 第11回　モジュールとインポート：mathライブラリを使う

!!! abstract "この回のゴール"
    - **モジュール**（部品の詰め合わせ）を `import` して使う
    - `math` で平方根・対数・指数などを計算する
    - pH 計算（対数）を Python で行う
    - 所要時間の目安: 60分
    - 使うテーマ：**pH と対数**、アレニウスの式

Python には、便利な機能をまとめた**モジュール**が最初から山ほど付いてきます（標準ライブラリ）。今回はその代表 `math` を使います。

`lesson11.py` を作りましょう。

---

## 1. import して使う

モジュールは `import` で読み込み、`モジュール名.機能名` で使います。

```python
import math

print(math.sqrt(16))     # 平方根 → 4.0
print(math.log10(1000))  # 常用対数（10を底）→ 3.0
print(math.exp(1))       # e^1 → 2.718...
print(math.pi)           # 円周率 → 3.14159...
```

出力:

```text
4.0
3.0
2.718281828459045
3.141592653589793
```

!!! note "いろいろな import の書き方"
    ```python
    import math                 # math.sqrt(...) と書く（おすすめ・どこの機能か明確）
    from math import sqrt, pi   # sqrt(...) と直接書ける
    import numpy as np          # 別名をつける（第3部で使う）
    ```

---

## 2. pH を計算する（対数の出番）

化学で対数といえば pH です。$\text{pH} = -\log_{10}[\text{H}^+]$。`math.log10` でそのまま書けます。

```python
import math

h = 1e-3                     # 水素イオン濃度 [mol/L]（= 1×10^-3）
pH = -math.log10(h)
print(f"[H+] = {h} のとき pH = {pH}")
```

出力:

```text
[H+] = 0.001 のとき pH = 3.0
```

逆に、pH から水素イオン濃度を求めるには $[\text{H}^+] = 10^{-\text{pH}}$。べき乗は `**` です。

```python
pH = 8.5
h = 10 ** (-pH)
pOH = 14 - pH
print(f"pH {pH} → [H+] = {h:.2e} mol/L, pOH = {pOH}")
```

出力:

```text
pH 8.5 → [H+] = 3.16e-09 mol/L, pOH = 5.5
```

---

## 3. アレニウスの式（指数関数）

反応速度定数の温度依存を表すアレニウスの式 $k = A\,e^{-E_a/RT}$ も、`math.exp` で書けます。

```python
import math

A = 1e13          # 頻度因子
Ea = 75000        # 活性化エネルギー [J/mol]
R = 8.314         # 気体定数 [J/(mol·K)]
T = 298           # 温度 [K]

k = A * math.exp(-Ea / (R * T))
print(f"速度定数 k = {k:.4f}")
```

出力:

```text
速度定数 k = 0.7132
```

!!! tip "どんなモジュールがある？"
    `math`（数学）、`statistics`（統計）、`random`（乱数）、`datetime`（日時）、`os`（ファイル操作）… 標準ライブラリだけでも膨大です。「やりたいこと Python モジュール」で検索するか、AIに聞くと、たいてい既にある部品が見つかります。

---

## 演習問題

**問1.** `math` を import して、(a) `math.sqrt(2)`、(b) `math.log10(100)`、(c) `math.exp(0)` を表示してください。

**問2.** 水素イオン濃度 `h = 2.5e-5` mol/L の溶液の pH を、`math.log10` を使って小数第2位まで計算してください。酸性・中性・塩基性のどれですか？

**問3.** アレニウスの式で、温度を `T = 310` K に上げたときの速度定数 k を計算してください（他の値は本文と同じ）。298 K のときと比べて、k は大きくなりますか小さくなりますか？

---

## 解答

??? success "問1 の解答"
    ```python
    import math
    print(math.sqrt(2))     # 1.4142135623730951
    print(math.log10(100))  # 2.0
    print(math.exp(0))      # 1.0
    ```

??? success "問2 の解答"
    ```python
    import math
    h = 2.5e-5
    pH = -math.log10(h)
    print(f"pH = {pH:.2f}")
    ```

    出力:
    ```text
    pH = 4.60
    ```
    pH 4.60 は 7 より小さいので**酸性**です。

??? success "問3 の解答"
    ```python
    import math
    A, Ea, R = 1e13, 75000, 8.314
    k310 = A * math.exp(-Ea / (R * 310))
    print(f"k(310K) = {k310:.4f}")
    ```

    出力:
    ```text
    k(310K) = 2.3021
    ```
    298 K では 0.7132 だったので、温度を上げると k は**大きく**なります（反応が速くなる）。わずか12 K の上昇で約3.2倍。温度の影響の大きさが分かります。

---

## この回のまとめ

- `import モジュール` で機能の詰め合わせを読み込み、`モジュール.機能()` で使う。
- `math.sqrt` `math.log10` `math.exp` `math.pi` など。
- pH = −log10[H+]、[H+] = 10^(−pH)、アレニウスは `math.exp` で。
- 標準ライブラリは膨大。まず「既にある部品」を探す。

### 次回予告

[第12回：ファイルの読み書き](lesson-12.md) では、計算結果や実験データをテキストファイルに保存し、読み戻す方法を学びます。
