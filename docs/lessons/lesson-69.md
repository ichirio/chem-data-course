# 第69回　単位・物理定数ライブラリ（pint）

!!! abstract "この回のゴール"
    - **pint** で「単位つきの数値」を扱う
    - 単位換算を安全に行う（次元の不一致を検出）
    - 温度など特殊な単位を換算する
    - 所要時間の目安: 60分
    - 使うテーマ：**単位を間違えない計算**

第27回では単位換算を手で計算しました。**pint** を使うと「単位そのもの」を数値に付けられ、換算も次元チェックも自動になります。単位ミス（火星探査機を失った有名な事故も単位ミス！）を防げます。

!!! info "準備：pint"
    ```bash
    pip install pint       # conda環境でも venv でも
    ```

`lesson69.py` を作りましょう。

---

## 1. 単位つきの数値を作る

`UnitRegistry` を作り、`数値 * u.単位` で単位つきの量（Quantity）を作ります。

```python
import pint

u = pint.UnitRegistry()

distance = 5 * u.meter
time = 2 * u.second
speed = distance / time

print(speed)                     # 単位つきで表示
print(speed.to("km/hour"))       # km/h に換算
```

出力:

```text
2.5 meter / second
9.0 kilometer / hour
```

数値だけでなく**単位も一緒に計算・換算**されます。`.to("単位")` で好きな単位に変換できます。

---

## 2. 化学でよく使う換算

```python
import pint

u = pint.UnitRegistry()

# 密度の換算
density = 5 * u.gram / u.centimeter**3
print("密度:", round(density.to("kg/m^3").magnitude, 1), "kg/m^3")

# 圧力の換算
pressure = 1 * u.atm
print("圧力:", pressure.to("kPa"))

# エネルギーの換算
energy = 1 * u.calorie
print("エネルギー:", energy.to("joule"))
```

出力:

```text
密度: 5000.0 kg/m^3
圧力: 101.325 kilopascal
エネルギー: 4.184 joule
```

`.magnitude` は「単位を外して数値だけ」を取り出す属性です（表示を整えるのに便利）。

---

## 3. 温度の換算（特別な扱い）

温度は「0点がずれている」特殊な単位なので、`u.Quantity(値, 単位)` の形で作ります。

```python
import pint

u = pint.UnitRegistry()
Q = u.Quantity

print(Q(25, u.degC).to(u.kelvin))     # 摂氏 → ケルビン
print(Q(100, u.degC).to(u.degF))      # 摂氏 → 華氏
```

出力:

```text
298.15 kelvin
211.99999999999991 degree_Fahrenheit
```

（華氏は浮動小数の誤差で 212 ちょうどになりませんが、実質212℉です。第29回の数値誤差。`round(..., 1)` で整えられます。）

---

## 4. 次元の不一致を検出してくれる

pint の最大の利点は、**単位が合わない計算をエラーにしてくれる**ことです。

```python
import pint

u = pint.UnitRegistry()

length = 5 * u.meter
mass = 3 * u.kilogram

try:
    result = length + mass       # 長さ + 質量 は不可能
except pint.DimensionalityError as e:
    print("エラー検出:", "長さと質量は足せません")
```

出力:

```text
エラー検出: 長さと質量は足せません
```

「メートルとキログラムを足す」ような**物理的にありえない計算**を、実行前に止めてくれます。単位を意識した安全な計算ができます。

!!! success "単位ミスを根絶する"
    実験・計算で単位を取り違えるミスは、研究でよくある落とし穴です。pint を使えば、単位を数値と一体で扱い、換算も次元チェックも自動化できます。重要な計算ほど、pint で安全に。

---

## 演習問題

**問1.** `pint` で、`10 * u.mole / u.liter`（10 mol/L）を作り、`mol/m^3` に換算してください（1 mol/L = 1000 mol/m³）。

**問2.** 圧力 `2 * u.atm` を、(a) kPa、(b) mmHg に換算してください。

**問3.** `u.Quantity(37, u.degC)`（体温37℃）を、ケルビンと華氏に換算してください。

---

## 解答

??? success "問1 の解答"
    ```python
    import pint
    u = pint.UnitRegistry()
    c = 10 * u.mole / u.liter
    print(c.to("mol/m^3"))
    ```

    出力:
    ```text
    9999.999999999998 mole / meter ** 3
    ```
    （実質10000 mol/m³。末尾は第29回の浮動小数の誤差です。）

??? success "問2 の解答"
    ```python
    import pint
    u = pint.UnitRegistry()
    p = 2 * u.atm
    print(p.to("kPa"))
    print(p.to("mmHg"))
    ```

    出力:
    ```text
    202.65 kilopascal
    1519.9997834512228 millimeter_Hg
    ```
    （実質1520 mmHg。2気圧＝1520 Torr です。mmHg と Torr はごくわずかに定義が異なります。）

??? success "問3 の解答"
    ```python
    import pint
    u = pint.UnitRegistry()
    Q = u.Quantity
    print(Q(37, u.degC).to(u.kelvin))
    print(Q(37, u.degC).to(u.degF))
    ```

    出力:
    ```text
    310.15 kelvin
    98.59999999999994 degree_Fahrenheit
    ```
    体温37℃＝98.6℉（末尾は浮動小数の誤差）。よく聞く値ですね。

---

## この回のまとめ

- pint は「単位つきの数値」を扱うライブラリ。単位ミスを防ぐ。
- `u = pint.UnitRegistry()`、`数値 * u.単位` で量を作る。`.to("単位")` で換算。
- 温度は `u.Quantity(値, u.degC)` の形で（0点がずれる特殊な単位）。
- 次元が合わない計算は `DimensionalityError` で止まる。

### 次回予告

[第70回：まとめ演習（医薬品分子の性質を比較する）](lesson-70.md) では、第6部の総仕上げとして、複数の医薬品分子を RDKit で総合的に比較します。
