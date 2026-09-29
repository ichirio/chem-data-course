# 第68回　周期表データを分析する（mendeleev入門）

!!! abstract "この回のゴール"
    - 元素の性質データを扱う **mendeleev** ライブラリを使う
    - 原子番号・原子量・電気陰性度などを取得する
    - 周期表の傾向（周期的性質）をデータで確かめる
    - 所要時間の目安: 60分
    - 使うテーマ：**元素の周期的性質**

**mendeleev** は、全元素の性質（原子量・電気陰性度・イオン化エネルギー…）をプログラムから取得できるライブラリです。

!!! info "準備：mendeleev"
    ```bash
    pip install mendeleev       # conda環境でも venv でも
    ```

`lesson68.py` を作りましょう。

---

## 1. 元素の性質を取得する

`element("記号")` で元素オブジェクトを作り、性質を属性で取り出します。

```python
from mendeleev import element

for sym in ["Fe", "Au", "C"]:
    e = element(sym)
    print(f"{sym} ({e.name}): 原子番号 {e.atomic_number}, "
          f"原子量 {round(e.atomic_weight, 3)}, 電気陰性度 {e.en_pauling}")
```

出力:

```text
Fe (Iron): 原子番号 26, 原子量 55.845, 電気陰性度 1.83
Au (Gold): 原子番号 79, 原子量 196.967, 電気陰性度 2.4
C (Carbon): 原子番号 6, 原子量 12.011, 電気陰性度 2.55
```

!!! note "取得できる主な性質"
    `atomic_number`（原子番号）、`atomic_weight`（原子量）、`en_pauling`（ポーリングの電気陰性度）、`group_id`（族）、`period`（周期）、`atomic_radius`（原子半径）、`ionenergies`（イオン化エネルギー [eV] の辞書。第1イオン化エネルギーは `ionenergies[1]`）など、非常に多くの性質があります。

---

## 2. 元素データを DataFrame にまとめる

複数の元素の性質を、pandas の表にまとめます（第63回と同じパターン）。

```python
from mendeleev import element
import pandas as pd

symbols = ["Li", "Be", "B", "C", "N", "O", "F"]   # 第2周期

rows = []
for sym in symbols:
    e = element(sym)
    rows.append({"symbol": sym, "name": e.name, "Z": e.atomic_number,
                 "EN": e.en_pauling, "group": e.group_id})

df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

出力:

```text
symbol      name  Z   EN  group
    Li   Lithium  3 0.98      1
    Be Beryllium  4 1.57      2
     B     Boron  5 2.04     13
     C    Carbon  6 2.55     14
     N  Nitrogen  7 3.04     15
     O    Oxygen  8 3.44     16
     F  Fluorine  9 3.98     17
```

---

## 3. 周期的性質を確かめる

第2周期（Li→F）で、電気陰性度が**右に行くほど大きくなる**ことがデータで見えます。第5部を使えばグラフにもできます。

```python
# 電気陰性度が最大・最小の元素
print("電気陰性度 最大:", df.loc[df["EN"].idxmax(), "name"], df["EN"].max())
print("電気陰性度 最小:", df.loc[df["EN"].idxmin(), "name"], df["EN"].min())
```

出力:

```text
電気陰性度 最大: Fluorine 3.98
電気陰性度 最小: Lithium 0.98
```

フッ素が最も電気陰性度が高い（貴ガスを除き、周期表右上ほど高い、という傾向）——教科書の周期的性質を、データで確認できました。

!!! success "教科書の法則をデータで再現"
    原子半径・イオン化エネルギー・電気陰性度の周期的変化は、mendeleev のデータと第4部・第5部を組み合わせれば、自分の手で可視化・確認できます。「なぜそうなるか」を、データを触りながら理解できるのが強みです。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「mendeleev で第2周期（Li〜Ne）の元素について、記号・ポーリングの電気陰性度（`en_pauling`）・第1イオン化エネルギー [eV]（`ionenergies[1]`）を DataFrame にまとめてください。
    値が None の元素（貴ガスの電気陰性度など）は NaN のまま残し、欠損の件数を print してください。
    最大・最小は `idxmax` / `idxmin` で求め、どの元素かを表示してください。」

!!! warning "AIの出力で確かめること"
    - `ionenergies[0]` と書いていないか。mendeleev の `ionenergies` は**1始まりの辞書**（キー 1, 2, 3…）なので、`[0]` は `KeyError` です。
    - 欠損（None → NaN）の扱い：`df["EN"].isna().sum()` を print。Ne・He などの電気陰性度は値がありません。`max()` に None を含むリストを渡すと `TypeError` になります。
    - AI の「傾向の説明」を鵜呑みにしない：第2周期の第1イオン化エネルギーは Be > B、N > O の逆転があり、単調増加ではありません。表の値を上から読んで確かめます。
    - 単位：イオン化エネルギーは eV（kJ/mol ではない）、原子半径 `atomic_radius` は pm。

---

## 演習問題

**問1.** ハロゲン `["F", "Cl", "Br", "I"]` について、原子番号・電気陰性度を表示してください。原子番号が大きくなると電気陰性度はどう変化しますか？

**問2.** アルカリ金属 `["Li", "Na", "K", "Rb"]` の原子量を DataFrame にまとめてください。

**問3.** 第3周期 `["Na", "Mg", "Al", "Si", "P", "S", "Cl"]` の電気陰性度を DataFrame にまとめ、最大・最小の元素を `idxmax` / `idxmin` で表示してください。

**問4.** 次は AI が書いた「第2周期の電気陰性度と第1イオン化エネルギーの表」を作るコードと、AI の説明です。問題点を指摘し、直してください。

```python
from mendeleev import element
import pandas as pd

rows = []
for s in ["Li", "Be", "B", "C", "N", "O", "F", "Ne"]:
    e = element(s)
    rows.append({"symbol": s, "EN": e.en_pauling, "IE1_eV": e.ionenergies[0]})
df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

AI の出力:

```text
KeyError: 0
```

AI の説明：「第2周期では、電気陰性度も第1イオン化エネルギーも、右へ行くほど単調に大きくなります。」

---

## 解答

??? success "問1 の解答"
    ```python
    from mendeleev import element
    for sym in ["F", "Cl", "Br", "I"]:
        e = element(sym)
        print(sym, e.atomic_number, e.en_pauling)
    ```

    出力:
    ```text
    F 9 3.98
    Cl 17 3.16
    Br 35 2.96
    I 53 2.66
    ```
    原子番号が大きくなる（下に行く）ほど、電気陰性度は**小さく**なります。

??? success "問2 の解答"
    ```python
    from mendeleev import element
    import pandas as pd
    rows = [{"symbol": s, "atomic_weight": round(element(s).atomic_weight, 3)}
            for s in ["Li", "Na", "K", "Rb"]]
    print(pd.DataFrame(rows).to_string(index=False))
    ```

    出力:
    ```text
    symbol  atomic_weight
        Li          6.940
        Na         22.990
         K         39.098
        Rb         85.468
    ```

??? success "問3 の解答"
    ```python
    from mendeleev import element
    import pandas as pd
    rows = [{"symbol": s, "EN": element(s).en_pauling}
            for s in ["Na", "Mg", "Al", "Si", "P", "S", "Cl"]]
    df = pd.DataFrame(rows)
    print("最大:", df.loc[df["EN"].idxmax(), "symbol"], df["EN"].max())
    print("最小:", df.loc[df["EN"].idxmin(), "symbol"], df["EN"].min())
    ```

    出力:
    ```text
    最大: Cl 3.16
    最小: Na 0.93
    ```

??? success "問4 の解答"
    誤りは2つです。

    1. **インデックスの誤り**：mendeleev の `ionenergies` は `{1: 第1, 2: 第2, ...}` という1始まりの辞書なので、`[0]` は `KeyError` です。第1イオン化エネルギーは `ionenergies[1]`。
    2. **説明が化学的に誤り**：実際のデータでは第1イオン化エネルギーが Be（9.32 eV）> B（8.30 eV）、N（14.53 eV）> O（13.62 eV）と逆転します（2p 軌道への入り始め、2p の電子対形成のため）。また Ne の電気陰性度は値が無く（NaN）、「単調に大きくなる」とは言えません。

    ```python
    from mendeleev import element
    import pandas as pd

    rows = []
    for s in ["Li", "Be", "B", "C", "N", "O", "F", "Ne"]:
        e = element(s)
        rows.append({"symbol": s, "EN": e.en_pauling, "IE1_eV": e.ionenergies[1]})  # 第1イオン化エネルギー
    df = pd.DataFrame(rows)
    print(df.to_string(index=False))
    print("EN の欠損:", df["EN"].isna().sum(), "件")
    print("EN 最大:", df.loc[df["EN"].idxmax(), "symbol"])
    print("IE1 最大:", df.loc[df["IE1_eV"].idxmax(), "symbol"])
    ```

    出力:
    ```text
    symbol   EN    IE1_eV
        Li 0.98  5.391715
        Be 1.57  9.322699
         B 2.04  8.298019
         C 2.55 11.260288
         N 3.04 14.534130
         O 3.44 13.618055
         F 3.98 17.422820
        Ne  NaN 21.564541
    EN の欠損: 1 件
    EN 最大: F
    IE1 最大: Ne
    ```

---

## この回のまとめ

- mendeleev で全元素の性質を取得。`element("記号")` → `.atomic_number` `.en_pauling` など。
- 複数元素を DataFrame にまとめれば、第4部・第5部の分析・可視化が使える。
- 周期的性質（電気陰性度など）を、教科書ではなく**データで**確認できる。

### 次回予告

[第69回：単位・物理定数ライブラリ（pint）](lesson-69.md) では、単位つきの計算を安全に行うライブラリを学びます。単位換算のミスを防げます。
