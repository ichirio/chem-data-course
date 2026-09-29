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

pint の `calorie` は熱化学カロリー（1 cal = 4.184 J）です。栄養学などで使う国際蒸気表カロリーは `u.international_calorie`（4.1868 J）です。

---

## 3. 温度の換算（特別な扱い）

温度は「0点がずれている」特殊な単位なので、`u.Quantity(値, 単位)` の形で作ります（`25 * u.degC` と書くと `OffsetUnitCalculusError` になります）。温度**差**を足すときは `u.delta_degC` を使います。

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

「メートルとキログラムを足す」ような**物理的にありえない計算**を、その計算をした瞬間にエラー（`DimensionalityError`）で止めてくれます。単位を意識した安全な計算ができます。

!!! success "単位ミスを根絶する"
    実験・計算で単位を取り違えるミスは、研究でよくある落とし穴です。pint を使えば、単位を数値と一体で扱い、換算も次元チェックも自動化できます。重要な計算ほど、pint で安全に。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「pint で、理想気体 0.50 mol・10.0 L・25 ℃ の圧力を kPa で求めてください。
    すべての量を単位付き（`u.mole`、`u.liter`）で作り、温度は `u.Quantity(25, u.degC).to(u.kelvin)` のようにケルビンに換算してから使ってください。
    `.magnitude` で単位を外すのは最後の表示のときだけにし、途中の値も単位付きで print してください。」

!!! warning "AIの出力で確かめること"
    - **摂氏をケルビンとして使っていないか**：`T = 25 * u.kelvin  # 25 ℃` のようにコメントと単位が食い違うコードは、pint でも検出できません（単位としては正しいので）。T を print して 298.15 K になっているか確認します。
    - 途中で `.magnitude` を取って単位を捨てていないか。捨てた後の計算は、pint の次元チェックが効きません。
    - 最終結果の桁が常識的か：0.5 mol・10 L・室温なら約 1.2 atm（約 124 kPa）。
    - 温度差（例：10 K 上がった）と温度そのものを区別しているか（`u.delta_degC`）。

---

## 演習問題

**問1.** `pint` で、`10 * u.mole / u.liter`（10 mol/L）を作り、`mol/m^3` に換算してください（1 mol/L = 1000 mol/m³）。

**問2.** 圧力 `2 * u.atm` を、(a) kPa、(b) mmHg に換算してください。

**問3.** `u.Quantity(37, u.degC)`（体温37℃）を、ケルビンと華氏に換算してください。

**問4.** 次は AI が書いた「0.50 mol の理想気体を 10.0 L の容器に入れ、室温 25 ℃ にしたときの圧力」を求めるコードです。問題点を指摘し、直してください。

```python
import pint

u = pint.UnitRegistry()
R = 0.082057 * u.liter * u.atm / (u.mole * u.kelvin)   # 気体定数

n = 0.50 * u.mole
V = 10.0 * u.liter
T = 25 * u.kelvin          # 室温 25 ℃
P = n * R * T / V
print(P.to("kPa"))
```

AI の出力:

```text
10.39303190625 kilopascal
```

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

??? success "問4 の解答"
    誤りは温度の単位です。コメントは「25 ℃」なのに、`25 * u.kelvin`（25 K ＝ −248 ℃）として計算しています。単位としては正しいので pint もエラーを出しません。「0.5 mol を 10 L に入れて約 0.1 atm」は小さすぎる、と桁で気づけます。摂氏は `u.Quantity(25, u.degC)` で作ってケルビンに換算します。

    ```python
    import pint

    u = pint.UnitRegistry()
    Q = u.Quantity
    R = 0.082057 * u.liter * u.atm / (u.mole * u.kelvin)

    n = 0.50 * u.mole
    V = 10.0 * u.liter
    T = Q(25, u.degC).to(u.kelvin)     # ℃ は Quantity で作ってから K に換算
    P = n * R * T / V
    print(T)
    print(round(P.to("kPa"), 1))
    ```

    出力:
    ```text
    298.15 kelvin
    123.9 kilopascal
    ```

---

## この回のまとめ

- pint は「単位つきの数値」を扱うライブラリ。単位ミスを防ぐ。
- `u = pint.UnitRegistry()`、`数値 * u.単位` で量を作る。`.to("単位")` で換算。
- 温度は `u.Quantity(値, u.degC)` の形で（0点がずれる特殊な単位）。
- 次元が合わない計算は `DimensionalityError` で止まる。

### 次回予告

[第70回：まとめ演習（医薬品分子の性質を比較する）](lesson-70.md) では、第6部の総仕上げとして、複数の医薬品分子を RDKit で総合的に比較します。
