# 第9回　文字列操作：化学式を文字として扱う

!!! abstract "この回のゴール"
    - 文字列の**長さ・取り出し（インデックス／スライス）**を扱う
    - `upper` `lower` `replace` `strip` などの**文字列メソッド**を使う
    - `split` / `join` で**区切り**を扱う（CSVデータの準備）
    - 化学式や測定データの文字列を、自在に加工する
    - 所要時間の目安: 60分

`lesson09.py` を作って進めましょう。

---

## 1. 文字列の長さと取り出し

文字列も、リストと同じように**0番から**数えて取り出せます。

```python
formula = "H2SO4"     # 硫酸

print(len(formula))   # 文字数 → 5
print(formula[0])     # 最初の文字 → H
print(formula[-1])    # 最後の文字 → 4
```

出力:

```text
5
H
4
```

### スライス：一部を切り出す

`[開始:終了]` で範囲を取り出せます（終了の位置は**含まない**）。

```python
formula = "H2SO4"

print(formula[0:2])   # 0番から2番の手前まで → H2
print(formula[2:])    # 2番から最後まで      → SO4
print(formula[:2])    # 最初から2番の手前まで → H2
```

出力:

```text
H2
SO4
H2
```

---

## 2. 文字列メソッド：変換・整形

文字列には便利な「メソッド（`.名前()` で呼ぶ機能）」がたくさんあります。よく使うものを見ましょう。

```python
name = "  Ethanol  "     # 前後に余分な空白

print(name.strip())        # 前後の空白を除く → "Ethanol"
print(name.strip().upper())# 大文字に        → "ETHANOL"
print(name.strip().lower())# 小文字に        → "ethanol"

formula = "C2H5OH"
print(formula.replace("OH", "O-H"))  # 置換 → "C2H5O-H"
print(formula.count("H"))            # 文字 "H" の出現回数 → 2
print("S" in formula)                # "S" を含む？ → False
print(formula.startswith("C"))       # "C" で始まる？ → True
```

出力:

```text
Ethanol
ETHANOL
ethanol
C2H5O-H
2
False
True
```

!!! note "メソッドは「つなげて」書ける"
    `name.strip().upper()` のように、`.` でつなげて順に処理できます（左から順に実行）。まず余分な空白を取り、次に大文字化、という流れです。

!!! warning "`count` は「文字」を数える（原子の数ではない）"
    エタノール $C_2H_5OH$ の水素**原子**は6個ですが、`"C2H5OH".count("H")` は**文字 H の出現回数＝2**を返します。ここで扱っているのはあくまで「文字列」です。分子中の原子数を正確に数えるのは、第6部の RDKit の役目です。

---

## 3. split / join：区切りを扱う

実験データはよく「カンマ区切り」や「スペース区切り」で保存されます。`split()` で**バラして**、`join()` で**つなげます**。ここは第4部（pandasでCSVを読む）への大事な土台です。

```python
line = "6.1,6.4,6.0,6.3,6.2"      # カンマ区切りの測定値

parts = line.split(",")           # カンマで分割 → リストになる
print(parts)
print(len(parts), "個の値")
```

出力:

```text
['6.1', '6.4', '6.0', '6.3', '6.2']
5 個の値
```

!!! warning "split の結果は「文字列のリスト」"
    `parts` の中身は `'6.1'` のような**文字列**です。数として計算するには `float()` で変換します。
    ```python
    total = 0.0
    for p in parts:
        total += float(p)         # 1つずつ数に変換して足す
    print(f"合計: {total:.1f}")   # 合計: 31.0
    ```

`join()` は逆に、リストを1本の文字列につなげます。

```python
elements = ["Na", "Cl"]
print("-".join(elements))     # "Na-Cl"
print(" + ".join(elements))   # "Na + Cl"
```

出力:

```text
Na-Cl
Na + Cl
```

---

## 4. 応用：化学式から大文字を数える

大文字は「元素の始まり」を表すことが多いです（`Na` の `N`、`Cl` の `C`）。文字を1つずつ見て、大文字の数＝おおよその元素の登場回数を数えてみましょう。

```python
formula = "NaHCO3"      # 炭酸水素ナトリウム

count = 0
for ch in formula:
    if ch.isupper():           # 大文字かどうか
        count += 1

print(f"{formula} には大文字が {count} 個（元素の始まりの目安）")
```

出力:

```text
NaHCO3 には大文字が 4 個（元素の始まりの目安）
```

（N, H, C, O の4つ。Na の `a` は小文字なので数えません。）

!!! note "本格的な化学式の解析は RDKit で"
    「H2O から H:2, O:1 を自動で取り出す」といった正確な解析は、実は結構むずかしい処理です。第6部の **RDKit** を使えば正確・簡単にできます。今回は「文字列を自在に触る感覚」をつかむのが目的です。

---

## 演習問題

**問1.** 文字列 `formula = "C6H12O6"`（グルコース）について、(a) 文字数、(b) 最初の文字、(c) 最後の文字、(d) 3番目の文字（インデックス2）をそれぞれ表示してください。

**問2.** カンマ区切りの測定値 `line = "12.5,13.1,12.8,13.0,12.6"` を `split` でバラし、`float` に変換しながら for で合計と平均を計算して表示してください。

**問3.** 元素記号のリスト `["C", "H", "N", "O", "P", "S"]` を、`join` を使って `"C-H-N-O-P-S"` という1本の文字列にして表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    formula = "C6H12O6"
    print("文字数:", len(formula))
    print("最初:", formula[0])
    print("最後:", formula[-1])
    print("3番目:", formula[2])
    ```

    出力:
    ```text
    文字数: 7
    最初: C
    最後: 6
    3番目: H
    ```

??? success "問2 の解答"
    ```python
    line = "12.5,13.1,12.8,13.0,12.6"
    parts = line.split(",")

    total = 0.0
    for p in parts:
        total += float(p)
    average = total / len(parts)

    print(f"合計: {total:.1f}")
    print(f"平均: {average:.3f}")
    ```

    出力:
    ```text
    合計: 64.0
    平均: 12.800
    ```

??? success "問3 の解答"
    ```python
    elements = ["C", "H", "N", "O", "P", "S"]
    print("-".join(elements))
    ```

    出力:
    ```text
    C-H-N-O-P-S
    ```

---

## この回のまとめ

- 文字列は `len()`・`[i]`・`[a:b]`（スライス）で長さ・取り出しができる
- `strip` `upper` `lower` `replace` `count` `startswith`、`in` などのメソッドが便利
- `split` で区切り文字ごとにバラし、`join` でつなげる（CSVの土台）
- `split` の結果は**文字列**なので、計算には `float()` で変換
- 正確な化学式解析は第6部の RDKit で

### 次回予告

[第10回：エラーと例外処理](lesson-10.md) では、入力ミスや想定外があっても**止まらない・落ちない**プログラムの書き方を学びます。第2部前半の仕上げです。
