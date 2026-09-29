# 第17回　読みやすいコード：コメントと命名の作法

!!! abstract "この回のゴール"
    - 後で読んで分かる**名前のつけ方**を知る
    - コメントを「なぜ」を書くために使う
    - マジックナンバーを定数にする
    - 所要時間の目安: 60分

コードは「一度書いて終わり」ではありません。**数週間後の自分**や**一緒に学ぶ仲間**が読みます。読みやすさは、正しさと同じくらい大切です。

---

## 1. 名前で意図を伝える

変数名は「中身が何か」を表す言葉にします。1文字の名前は避けましょう。

```python
# 悪い例：何のことか分からない
x = 0.5
y = 18.015
z = x / y

# 良い例：名前で意味が分かる
mass_g = 0.5
molar_mass = 18.015
moles = mass_g / molar_mass
```

!!! note "命名の慣習（Python）"
    - 変数・関数 … 小文字＋アンダースコア（`molar_mass`, `atom_count`）。これを **snake_case** と呼びます。
    - 定数（変えない値）… 大文字（`AVOGADRO`, `ATOMIC_MASS`）。
    - クラス … 先頭大文字（`Molecule`）。

    化学の記号にならって `pH` や `Ea` のように大文字を混ぜたくなりますが、Python の慣習（PEP 8）では変数名はすべて小文字の snake_case（`ph`, `ea_j_per_mol`）が基本です。ただし `pH` のように分野で定着した表記は、読みやすさを優先して使うこともあります（第11回）。どちらにするかはプロジェクト内で統一しましょう。

---

## 2. マジックナンバーを定数にする

コードの中に突然現れる数字（マジックナンバー）は、意味が分からず、変更も大変です。**名前をつけて定数**にします。

```python
# 悪い例：6.022e23 が何か、パッと見で分からない
molecules = moles * 6.022e23

# 良い例：名前で意味が明確。値の変更も1か所で済む
AVOGADRO = 6.022e23        # アボガドロ定数 [1/mol]（定義値 6.02214076e23 を4桁に丸めた値）
molecules = moles * AVOGADRO
```

---

## 3. コメントは「なぜ」を書く

コードを見れば分かる「何をしているか」ではなく、**「なぜそうするのか」**をコメントに書きます。

```python
# 悪いコメント：見れば分かることを書いている
temperature = temperature + 273.15   # temperature に 273.15 を足す

# 良いコメント：理由・背景を書いている（変数名にも単位を入れる）
temperature_K = temperature_C + 273.15   # 気体の状態方程式には絶対温度が必要なため

# 良いコメント：注意点を残す
yield_pct = measured / theoretical * 100   # theoretical は 0 でない前提（第10回の検証済み）
```

!!! tip "docstring も活用（第6回）"
    関数の説明は、コメントより docstring（`"""..."""`）で書くのがおすすめ。`help(関数名)` で読み出せて、AIにも伝わりやすくなります。

---

## 4. before / after：読みやすく直す

同じ計算でも、書き方で読みやすさが大きく変わります。

```python
# before（読みにくい）
def f(m, mm):
    return m / mm * 6.022e23

# after（読みやすい）
AVOGADRO = 6.022e23

def count_molecules(mass_g, molar_mass):
    """質量[g]とモル質量[g/mol]から分子数を返す"""
    moles = mass_g / molar_mass
    return moles * AVOGADRO

print(count_molecules(18.0, 18.015))
```

出力:

```text
6.016985845129059e+23
```

同じ結果でも、`after` は「何をする関数か」が名前とdocstringで分かり、`6.022e23` の意味も明確です。

!!! success "読みやすさは未来への贈り物"
    「動けばいい」で終わらせず、**名前・定数・コメント**を整える。少しの手間が、後の自分と仲間を助けます。AIにコードを見せるときも、読みやすいコードのほうが的確な助けを得られます。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「次の関数を、動作を一切変えずに読みやすく書き直してください。
    変数名は snake_case で、物理量には単位を名前に含めてください（例: `volume_L`, `temperature_K`）。
    数値の定数は大文字の定数にして、単位と出典をコメントに書いてください。関数には docstring を付けてください。
    書き直す前と後で、同じ入力に対して同じ結果が出ることを確かめるテスト用の print も付けてください。」

!!! warning "AIの出力で確かめること"
    - 「読みやすくする」だけのはずが、**計算結果が変わっていないか**。書き直し前と後で同じ入力を与え、出力が一致するか print して比べる。
    - 定数の**値と、コメントに書いた単位が一致しているか**。例: 気体定数 8.314 は J/(mol·K)、0.082057 は L·atm/(mol·K)。コメントが「うそ」になっていないか。
    - 温度が摂氏のまま絶対温度の式に入っていないか。変数名（`_C` / `_K`）と実際の計算が合っているか。
    - コメントが「何をしているか」の言い換えだけになっていないか（「なぜ」が書かれているか）。

---

## 演習問題

**問1.** 次の読みにくいコードを、意味の分かる名前に直してください。
```python
a = 100
b = 22.4
c = a / b
print(c)
```
（ヒント：気体の体積[L]と、1molあたりの体積[L/mol]から物質量[mol]を求めている）

**問2.** コード中の `9.81` をマジックナンバーにせず、`GRAVITY` という定数にして使うように書き換えてください（用途は自由。例：位置エネルギー `m * GRAVITY * h`）。

**問3.** 次の関数に、良い名前・docstring・意味のあるコメントをつけて読みやすくしてください。
```python
def g(c, v):
    return c * v
```
（ヒント：モル濃度 c [mol/L] と体積 v [L] から物質量 [mol] を求めている）

**問4.** 次は AI が「読みやすく書き直した」理想気体の圧力計算です。名前もコメントも docstring も整っていますが、1 mol・22.4 L・0 °C で 0.0 atm という結果になりました。問題点を2つ指摘し、直してください。
```python
GAS_CONSTANT = 8.314          # 気体定数 [L·atm/(mol·K)]

def pressure_atm(moles, volume_L, temperature_C):
    """理想気体の圧力 [atm] を返す"""
    return moles * GAS_CONSTANT * temperature_C / volume_L

print(pressure_atm(1.0, 22.4, 0))
print(pressure_atm(1.0, 24.5, 25))
```

出力:
```text
0.0
8.483673469387755
```

---

## 解答

??? success "問1 の解答"
    ```python
    volume_L = 100
    MOLAR_VOLUME_L = 22.4        # 理想気体 1 mol の体積 [L/mol]（0 °C, 1.013×10^5 Pa）
    moles = volume_L / MOLAR_VOLUME_L
    print(moles)
    ```

    出力:
    ```text
    4.464285714285714
    ```

??? success "問2 の解答"
    ```python
    GRAVITY = 9.81               # 重力加速度 [m/s^2]

    def potential_energy(mass_kg, height_m):
        """位置エネルギー [J] を返す"""
        return mass_kg * GRAVITY * height_m

    print(f"{potential_energy(2.0, 10.0):.1f} J")   # 196.2 J
    ```

??? success "問3 の解答"
    ```python
    def moles_from_concentration(concentration, volume_L):
        """モル濃度 [mol/L] と体積 [L] から物質量 [mol] を返す"""
        return concentration * volume_L   # n = C × V

    print(moles_from_concentration(0.5, 2.0))   # 1.0
    ```

??? success "問4 の解答"
    **誤り1：定数の値とコメントの単位が合っていません。** 8.314 は J/(mol·K) の値です。L·atm/(mol·K) で計算するなら 0.082057 を使います。コメントが整っていても、**値と単位の対応**は自分で確かめる必要があります。

    **誤り2：温度を摂氏のまま使っています。** 状態方程式 PV = nRT の T は絶対温度 [K] です。0 °C を 0 として掛けたため、圧力が 0 になりました。

    **確かめ方：** 0 °C・1 mol・22.4 L なら約 1 atm になるはず、という既知の値と比べます。

    ```python
    GAS_CONSTANT_L_ATM = 0.082057   # 気体定数 [L·atm/(mol·K)]
    ZERO_CELSIUS_K = 273.15         # 0 °C の絶対温度 [K]

    def pressure_atm(moles, volume_L, temperature_C):
        """理想気体の圧力 [atm] を返す（温度は摂氏で受け取り、内部でケルビンに換算）"""
        temperature_K = temperature_C + ZERO_CELSIUS_K   # 気体の状態方程式は絶対温度が必要
        return moles * GAS_CONSTANT_L_ATM * temperature_K / volume_L

    print(f"{pressure_atm(1.0, 22.4, 0):.3f}")
    print(f"{pressure_atm(1.0, 24.5, 25):.3f}")
    ```

    出力:
    ```text
    1.001
    0.999
    ```
    どちらも約 1 atm になり、既知の値（0 °C で 22.4 L/mol、25 °C で約 24.5 L/mol）と整合します。

---

## この回のまとめ

- 変数名は**中身を表す言葉**で（snake_case）。1文字名は避ける。
- マジックナンバーは**定数**（大文字）に名前をつける。
- コメントは「何を」でなく**「なぜ」**を書く。関数説明は docstring。
- 読みやすさは未来の自分・仲間・AIへの贈り物。

### 次回予告

[第18回：デバッグの基礎](lesson-18.md) では、エラーやおかしな結果に出会ったとき、原因を見つけて直す方法を学びます。
