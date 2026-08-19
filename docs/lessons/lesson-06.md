# 第6回　関数をつくる：分子量計算を「部品」にまとめる

!!! abstract "この回のゴール"
    - `def` で自分の**関数（部品）**を作る
    - **引数**でデータを渡し、`return` で結果を受け取る
    - **デフォルト引数**と、説明書き（docstring）を書く
    - 分子量計算を関数にして、どんな分子でも一発で計算できるようにする
    - 所要時間の目安: 60分

`lesson06.py` を作って進めましょう。

---

## 1. 関数とは「名前をつけた処理のまとまり」

これまで同じような計算を何度も書いてきました。**関数**にまとめておけば、名前を呼ぶだけで何度でも使えます。

```python
def greet():
    print("ようこそ、化学プログラミングへ！")

# 呼び出す（何度でも）
greet()
greet()
```

出力:

```text
ようこそ、化学プログラミングへ！
ようこそ、化学プログラミングへ！
```

!!! note "書き方のきまり"
    - `def 関数名():` で始め、行末に**コロン `:`**。
    - 中身は**字下げ**する。
    - 定義しただけでは動かない。**呼び出して**はじめて実行される。

---

## 2. 引数と return：値を渡して、結果を受け取る

関数に値を渡す入り口が**引数（ひきすう）**、結果を返す出口が **`return`** です。

```python
def celsius_to_kelvin(celsius):
    kelvin = celsius + 273.15
    return kelvin

# 使ってみる
t = celsius_to_kelvin(25)
print(f"25℃ は {t} K です")
print(f"100℃ は {celsius_to_kelvin(100)} K です")
```

出力:

```text
25℃ は 298.15 K です
100℃ は 373.15 K です
```

!!! tip "`print` と `return` は違う"
    - `print` … 画面に**表示するだけ**。
    - `return` … 計算結果を**呼び出し元に返す**（あとで変数に入れたり、さらに計算に使える）。

    関数は「計算して return で返す」のが基本。表示は呼び出した側でやると、部品として使い回せます。

---

## 3. 引数は複数、デフォルト値も持てる

引数はいくつでも書けます。また、`=` で**初期値（デフォルト）**を決めておくと、省略できます。

```python
def moles(mass_g, molar_mass):
    """質量[g]とモル質量[g/mol]から物質量[mol]を返す"""
    return mass_g / molar_mass

def dilute(c1, v1, v2=1.0):
    """C1・V1 = C2・V2 から、薄めたあとの濃度 C2 を返す（V2の既定は1.0 L）"""
    return c1 * v1 / v2

print(moles(36.0, 18.015))       # 水36gの物質量
print(dilute(1.0, 0.1))          # 1.0 mol/L を 0.1 L 取り、1.0 L にうすめる
print(dilute(1.0, 0.1, 0.5))     # 同じものを 0.5 L にうすめる
```

出力:

```text
1.9983347210657785
0.1
0.2
```

!!! note "三重引用符 `\"\"\" ... \"\"\"` は説明書き（docstring）"
    関数のすぐ下に書く説明文です。**何をする関数か**を一言で書いておくと、後で読み返したとき（そしてAIに読ませたとき）に役立ちます。`help(moles)` で読み出せます。

小数のケタがそろわないのが気になりますね。表示する側で整えましょう。

```python
n = moles(36.0, 18.015)
print(f"水 36 g は {n:.3f} mol です")
```

出力:

```text
水 36 g は 1.998 mol です
```

---

## 4. 本命：分子量を計算する関数

第3回で作った「原子量表（辞書）× 分子の組成（辞書）」の計算を、**関数**にまとめます。これで一度書けば、どんな分子でも一発です。

```python
# 原子量表（関数の外に置いて、みんなで使う）
ATOMIC_MASS = {
    "H": 1.008, "C": 12.011, "N": 14.007, "O": 15.999,
    "Na": 22.990, "S": 32.06, "Cl": 35.453,
}

def molar_mass(composition):
    """組成の辞書 {元素: 個数} からモル質量[g/mol]を返す"""
    total = 0.0
    for symbol, count in composition.items():
        total += ATOMIC_MASS[symbol] * count
    return total

# いろいろな分子で試す
water   = {"H": 2, "O": 1}
ethanol = {"C": 2, "H": 6, "O": 1}
glucose = {"C": 6, "H": 12, "O": 6}

print(f"水       : {molar_mass(water):.3f} g/mol")
print(f"エタノール: {molar_mass(ethanol):.3f} g/mol")
print(f"グルコース: {molar_mass(glucose):.3f} g/mol")
```

出力:

```text
水       : 18.015 g/mol
エタノール: 46.069 g/mol
グルコース: 180.156 g/mol
```

!!! success "部品が育っていく"
    `molar_mass()` という部品ができました。この先、この関数を「質量→モル数」「モル数→分子数」の計算と組み合わせれば、どんどん便利になります。関数を積み重ねるのがプログラミングの醍醐味です。

### 関数どうしを組み合わせる

```python
def molecules(mass_g, composition):
    """質量[g]と組成から、分子の個数を返す"""
    AVOGADRO = 6.022e23
    mm = molar_mass(composition)        # さっき作った関数を中で使う
    n = moles(mass_g, mm)               # これも使う
    return n * AVOGADRO

glucose = {"C": 6, "H": 12, "O": 6}
num = molecules(90.0, glucose)
print(f"グルコース 90 g 中の分子数は およそ {num:.3e} 個")
```

出力:

```text
グルコース 90 g 中の分子数は およそ 3.008e+23 個
```

---

## 演習問題

**問1.** 温度を**ケルビンから摂氏**に変換する関数 `kelvin_to_celsius(kelvin)` を作り、`298.15 K` を変換して表示してください。

**問2.** 気体の状態方程式 $PV = nRT$ から、**物質量 n を返す関数** `ideal_gas_n(P, V, T, R=0.0821)` を作ってください（`n = PV / (RT)`、R の既定値は 0.0821 L·atm/(mol·K)）。`P=1.0 atm, V=22.4 L, T=273 K` のときの n を小数第3位まで表示してください。

**問3.** 本文の `molar_mass()` と `ATOMIC_MASS` を使って、**酢酸 $C_2H_4O_2$（C 2・H 4・O 2）** と **炭酸ナトリウム $Na_2CO_3$（Na 2・C 1・O 3）** のモル質量を表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    def kelvin_to_celsius(kelvin):
        """絶対温度[K]を摂氏[℃]に変換して返す"""
        return kelvin - 273.15

    print(f"298.15 K は {kelvin_to_celsius(298.15)} ℃ です")
    ```

    出力:
    ```text
    298.15 K は 25.0 ℃ です
    ```

??? success "問2 の解答"
    ```python
    def ideal_gas_n(P, V, T, R=0.0821):
        """PV = nRT から物質量 n [mol] を返す"""
        return P * V / (R * T)

    n = ideal_gas_n(1.0, 22.4, 273)
    print(f"n = {n:.3f} mol")
    ```

    出力:
    ```text
    n = 0.999 mol
    ```

    （標準状態の理想気体 22.4 L はおよそ 1 mol。理論どおりですね。）

??? success "問3 の解答"
    ```python
    ATOMIC_MASS = {
        "H": 1.008, "C": 12.011, "N": 14.007, "O": 15.999,
        "Na": 22.990, "S": 32.06, "Cl": 35.453,
    }

    def molar_mass(composition):
        total = 0.0
        for symbol, count in composition.items():
            total += ATOMIC_MASS[symbol] * count
        return total

    acetic  = {"C": 2, "H": 4, "O": 2}
    na2co3  = {"Na": 2, "C": 1, "O": 3}

    print(f"酢酸        : {molar_mass(acetic):.3f} g/mol")
    print(f"炭酸ナトリウム: {molar_mass(na2co3):.3f} g/mol")
    ```

    出力:
    ```text
    酢酸        : 60.052 g/mol
    炭酸ナトリウム: 105.988 g/mol
    ```

---

## この回のまとめ

- `def 名前(引数):` で関数を作り、`return` で結果を返す
- `print` は表示するだけ、`return` は値を返す（部品として使い回せる）
- 引数は複数OK、`=` でデフォルト値を持てる
- docstring（`""" ... """`）で関数の説明を書く
- 関数の中で別の関数を呼べる＝部品を組み合わせて大きな機能に育てる

### 次回予告

[第7回：GitとGitHub](lesson-07.md) では、書いたコードを記録・共有する方法を学びます。仲間・研究室で1つのリポジトリを使い、お互いのコードを見せ合えるようにします。
