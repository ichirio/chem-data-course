# 第15回　クラス入門：分子を「オブジェクト」で表す

!!! abstract "この回のゴール"
    - **クラス**で「データ＋処理」をひとまとめにする
    - `__init__` で初期化、メソッドで機能を持たせる
    - 分子を表す `Molecule` クラスを作る
    - 所要時間の目安: 60分
    - 使うテーマ：**分子をオブジェクトとして扱う**

これまで「分子の組成（辞書）」と「分子量を計算する関数」は別々でした。**クラス**を使うと、それらを1つの"もの（オブジェクト）"にまとめられます。

`lesson15.py` を作りましょう。

---

## 1. クラスとは「設計図」

クラスは、同じ種類のものを作るための**設計図**です。設計図から作った実体を**インスタンス**（オブジェクト）と呼びます。

```python
class Molecule:
    def __init__(self, name, comp):
        self.name = name        # 名前
        self.comp = comp        # 組成（辞書）

# 設計図からインスタンスを作る
water = Molecule("water", {"H": 2, "O": 1})

print(water.name)     # water
print(water.comp)     # {'H': 2, 'O': 1}
```

出力:

```text
water
{'H': 2, 'O': 1}
```

!!! note "`__init__` と `self` の意味"
    - `__init__` … インスタンスを作るときに**自動で呼ばれる**初期化メソッド。
    - `self` … 「作られたインスタンス自身」を指す。`self.name = name` で、そのインスタンスに名前をしまう。
    - `Molecule("water", {...})` と呼ぶと、`__init__` の `name` に "water"、`comp` に辞書が渡される（`self` は自動）。

---

## 2. メソッド：オブジェクトに機能を持たせる

クラスの中に定義した関数を**メソッド**といいます。分子量を計算する `molar_mass` メソッドを足しましょう。

```python
class Molecule:
    ATOMIC_MASS = {"H": 1.008, "C": 12.011, "O": 15.999, "N": 14.007}

    def __init__(self, name, comp):
        self.name = name
        self.comp = comp

    def molar_mass(self):
        total = sum(self.ATOMIC_MASS[s] * n for s, n in self.comp.items())
        return round(total, 3)

# 使ってみる
water = Molecule("water", {"H": 2, "O": 1})
ethanol = Molecule("ethanol", {"C": 2, "H": 6, "O": 1})

print(f"{water.name} の分子量: {water.molar_mass()}")
print(f"{ethanol.name} の分子量: {ethanol.molar_mass()}")
```

出力:

```text
water の分子量: 18.015
ethanol の分子量: 46.069
```

`water.molar_mass()` のように、**オブジェクトに「.」でメソッドを呼びます**。データ（組成）と処理（分子量計算）が1つにまとまり、扱いやすくなりました。

!!! tip "これまで使ってきた `.` の正体"
    `"H2O".upper()` や `df.mean()` の `.` は、まさにこれ——**オブジェクトのメソッドを呼んでいた**のです。文字列も DataFrame も、実はクラスから作られたオブジェクトです。

---

## 3. たくさんのオブジェクトをまとめて扱う

インスタンスはいくつでも作れます。リストに入れてループすれば、一括処理も簡単です。

```python
molecules = [
    Molecule("water", {"H": 2, "O": 1}),
    Molecule("methane", {"C": 1, "H": 4}),
    Molecule("glucose", {"C": 6, "H": 12, "O": 6}),
]

for m in molecules:
    print(f"{m.name}: {m.molar_mass()} g/mol")
```

出力:

```text
water: 18.015 g/mol
methane: 16.043 g/mol
glucose: 180.156 g/mol
```

---

## 演習問題

**問1.** 本文の `Molecule` クラスを使って、二酸化炭素 `Molecule("CO2", {"C": 1, "O": 2})` のインスタンスを作り、名前と分子量を表示してください。

**問2.** `Molecule` クラスに、原子の総数を返すメソッド `atom_count` を追加してください（ヒント：`sum(self.comp.values())`）。水なら 3、グルコースなら 24 になります。

**問3.** 3つの分子（水・アンモニア `{"N":1,"H":3}`・メタノール `{"C":1,"H":4,"O":1}`）をリストに入れ、for ループで「名前・分子量・原子数」を表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    co2 = Molecule("CO2", {"C": 1, "O": 2})
    print(f"{co2.name} の分子量: {co2.molar_mass()}")
    ```

    出力:
    ```text
    CO2 の分子量: 44.009
    ```

??? success "問2 の解答"
    ```python
    class Molecule:
        ATOMIC_MASS = {"H": 1.008, "C": 12.011, "O": 15.999, "N": 14.007}

        def __init__(self, name, comp):
            self.name = name
            self.comp = comp

        def molar_mass(self):
            return round(sum(self.ATOMIC_MASS[s] * n for s, n in self.comp.items()), 3)

        def atom_count(self):
            return sum(self.comp.values())

    print(Molecule("water", {"H": 2, "O": 1}).atom_count())        # 3
    print(Molecule("glucose", {"C": 6, "H": 12, "O": 6}).atom_count())  # 24
    ```

    出力:
    ```text
    3
    24
    ```

??? success "問3 の解答"
    ```python
    molecules = [
        Molecule("water", {"H": 2, "O": 1}),
        Molecule("ammonia", {"N": 1, "H": 3}),
        Molecule("methanol", {"C": 1, "H": 4, "O": 1}),
    ]
    for m in molecules:
        print(f"{m.name}: {m.molar_mass()} g/mol, 原子数 {m.atom_count()}")
    ```

    出力:
    ```text
    water: 18.015 g/mol, 原子数 3
    ammonia: 17.031 g/mol, 原子数 4
    methanol: 32.042 g/mol, 原子数 6
    ```

---

## この回のまとめ

- クラスは「データ＋処理」をまとめた設計図。実体がインスタンス。
- `__init__(self, ...)` で初期化、`self.属性` にデータをしまう。
- クラス内の関数＝メソッド。`オブジェクト.メソッド()` で呼ぶ。
- 文字列や DataFrame の `.メソッド()` も、この仕組みだった。

### 次回予告

[第16回：Jupyter／セル実行に慣れる](lesson-16.md) では、コードを少しずつ対話的に試せる Jupyter 形式の使い方を学びます。データ分析で大活躍します。
