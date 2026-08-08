# 第13回　内包表記：スマートにリストをつくる

!!! abstract "この回のゴール"
    - **リスト内包表記**で、短く読みやすくリストを作る
    - 条件つき内包表記でフィルタする
    - 辞書内包表記も知る
    - 所要時間の目安: 60分
    - 使うテーマ：**元素・分子量**の一括計算

「for ループで1つずつ追加」は、内包表記を使うと**1行**で書けます。Python らしい書き方で、慣れると手放せません。

`lesson13.py` を作りましょう。

---

## 1. 基本：for を1行にたたむ

まず、これまでの書き方。

```python
# これまで（for で append）
temps_C = [0, 25, 50, 100]
temps_K = []
for t in temps_C:
    temps_K.append(t + 273.15)
print(temps_K)
```

これを内包表記で書くと、こうなります。

```python
temps_C = [0, 25, 50, 100]
temps_K = [t + 273.15 for t in temps_C]   # 1行！
print(temps_K)
```

出力（どちらも同じ）:

```text
[273.15, 298.15, 323.15, 373.15]
```

読み方は **`[やること for 変数 in 元のリスト]`**。「元のリストの各要素に対して、やることをした結果を並べたリスト」です。

---

## 2. 条件つき：if でフィルタする

`if` を付けると、条件に合うものだけを残せます。

```python
elements = ["H", "He", "Li", "Be", "B", "C", "N", "O", "F", "Ne"]

# 元素記号が1文字のものだけ
single = [e for e in elements if len(e) == 1]
print(single)
```

出力:

```text
['H', 'B', 'C', 'N', 'O', 'F']
```

読み方は **`[要素 for 要素 in リスト if 条件]`**。「条件を満たす要素だけを並べる」です。

---

## 3. 化学で使う：分子量を一気に計算

複数の分子の分子量を、内包表記でまとめて計算します。

```python
ATOMIC_MASS = {"H": 1.008, "C": 12.011, "O": 15.999, "N": 14.007}

molecules = {
    "water":    {"H": 2, "O": 1},
    "methanol": {"C": 1, "H": 4, "O": 1},
    "ammonia":  {"N": 1, "H": 3},
}

def molar_mass(comp):
    return round(sum(ATOMIC_MASS[s] * n for s, n in comp.items()), 3)

# 各分子の分子量をまとめて計算
masses = [molar_mass(comp) for comp in molecules.values()]
print(masses)
```

出力:

```text
[18.015, 32.042, 17.031]
```

!!! note "`sum(... for ...)` も内包表記の仲間"
    `sum(ATOMIC_MASS[s] * n for s, n in comp.items())` は、**その場で計算して合計**する書き方（ジェネレータ式）。第6回では for ループで足していましたが、これで1行に凝縮できます。

---

## 4. 辞書内包表記

`{キー: 値 for ...}` と書くと、辞書も内包表記で作れます。

```python
ATOMIC_MASS = {"H": 1.008, "C": 12.011, "O": 15.999, "N": 14.007}

molecules = {"water": {"H": 2, "O": 1}, "methanol": {"C": 1, "H": 4, "O": 1}, "ammonia": {"N": 1, "H": 3}}

def molar_mass(comp):
    return round(sum(ATOMIC_MASS[s] * n for s, n in comp.items()), 3)

# 名前 → 分子量 の辞書を一気に作る
mass_table = {name: molar_mass(comp) for name, comp in molecules.items()}
print(mass_table)
```

出力:

```text
{'water': 18.015, 'methanol': 32.042, 'ammonia': 17.031}
```

!!! warning "読みやすさ優先"
    内包表記は便利ですが、詰め込みすぎると読みにくくなります。条件が複雑なときや処理が長いときは、無理せず普通の for ループにしましょう。「短い」より「読める」が大事です。

---

## 演習問題

**問1.** リスト `masses = [18.015, 46.069, 180.156, 60.052]` の各要素を2倍したリストを、内包表記で作って表示してください。

**問2.** 元素記号のリスト `["Na", "Cl", "H", "Mg", "O", "Fe"]` から、**2文字の元素記号だけ**を内包表記で取り出してください。

**問3.** 温度のリスト `celsius = [-40, 0, 25, 37, 100]` を、内包表記でケルビン（+273.15）に変換したリストを作ってください。さらに、**0℃以上のものだけ**をケルビンに変換したリストも作ってください（`if` 付き）。

---

## 解答

??? success "問1 の解答"
    ```python
    masses = [18.015, 46.069, 180.156, 60.052]
    doubled = [m * 2 for m in masses]
    print(doubled)
    ```

    出力:
    ```text
    [36.03, 92.138, 360.312, 120.104]
    ```

??? success "問2 の解答"
    ```python
    elements = ["Na", "Cl", "H", "Mg", "O", "Fe"]
    two_letter = [e for e in elements if len(e) == 2]
    print(two_letter)
    ```

    出力:
    ```text
    ['Na', 'Cl', 'Mg', 'Fe']
    ```

??? success "問3 の解答"
    ```python
    celsius = [-40, 0, 25, 37, 100]
    kelvin = [c + 273.15 for c in celsius]
    kelvin_pos = [c + 273.15 for c in celsius if c >= 0]
    print(kelvin)
    print(kelvin_pos)
    ```

    出力:
    ```text
    [233.15, 273.15, 298.15, 310.15, 373.15]
    [273.15, 298.15, 310.15, 373.15]
    ```

---

## この回のまとめ

- リスト内包表記 `[やること for x in リスト]` で for ループを1行に。
- 条件つき `[x for x in リスト if 条件]` でフィルタ。
- 辞書内包表記 `{k: v for ...}`、合計は `sum(... for ...)`。
- 便利だが**読みやすさ優先**。複雑なら普通の for に。

### 次回予告

[第14回：ラムダと高階関数](lesson-14.md) では、`sorted` や `map` と組み合わせて、データを自在に並べ替え・変換する方法を学びます。
