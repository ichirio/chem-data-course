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

    化学の記号にならって `pH` のように書きたくなりますが、Python の慣習では変数は小文字始まりが基本です。

---

## 2. マジックナンバーを定数にする

コードの中に突然現れる数字（マジックナンバー）は、意味が分からず、変更も大変です。**名前をつけて定数**にします。

```python
# 悪い例：6.022e23 が何か、パッと見で分からない
molecules = moles * 6.022e23

# 良い例：名前で意味が明確。値の変更も1か所で済む
AVOGADRO = 6.022e23        # アボガドロ定数 [1/mol]
molecules = moles * AVOGADRO
```

---

## 3. コメントは「なぜ」を書く

コードを見れば分かる「何をしているか」ではなく、**「なぜそうするのか」**をコメントに書きます。

```python
# 悪いコメント：見れば分かることを書いている
temperature = temperature + 273.15   # temperature に 273.15 を足す

# 良いコメント：理由・背景を書いている
temperature = temperature + 273.15   # 摂氏からケルビンへ（絶対温度が必要なため）

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

---

## 解答

??? success "問1 の解答"
    ```python
    volume_L = 100
    molar_volume = 22.4          # 標準状態の1molあたり体積 [L/mol]
    moles = volume_L / molar_volume
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

    print(potential_energy(2.0, 10.0))   # 196.2
    ```

??? success "問3 の解答"
    ```python
    def moles_from_concentration(concentration, volume_L):
        """モル濃度 [mol/L] と体積 [L] から物質量 [mol] を返す"""
        return concentration * volume_L   # n = C × V

    print(moles_from_concentration(0.5, 2.0))   # 1.0
    ```

---

## この回のまとめ

- 変数名は**中身を表す言葉**で（snake_case）。1文字名は避ける。
- マジックナンバーは**定数**（大文字）に名前をつける。
- コメントは「何を」でなく**「なぜ」**を書く。関数説明は docstring。
- 読みやすさは未来の自分・仲間・AIへの贈り物。

### 次回予告

[第18回：デバッグの基礎](lesson-18.md) では、エラーやおかしな結果に出会ったとき、原因を見つけて直す方法を学びます。
