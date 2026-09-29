# 第14回　ラムダと高階関数：データ変換の基本

!!! abstract "この回のゴール"
    - **ラムダ式**（使い捨ての小さな関数）を書く
    - `sorted` の `key` で、好きな基準で並べ替える
    - `map` / `filter` で変換・抽出する
    - 所要時間の目安: 60分
    - 使うテーマ：**ポリマー物性**の並べ替え・変換

今回は「ラムダ式」と、関数を引数に受け取る「高階関数」（`sorted` / `map` / `filter`）を学びます。ラムダは第37回で pandas の `apply(lambda x: ...)` として再び登場します。

`lesson14.py` を作りましょう。

---

## 1. ラムダ式：名前のない小さな関数

`def` で関数を作らなくても、その場で使う短い関数は **`lambda`** で書けます。

```python
# def を使う書き方
def square(x):
    return x ** 2

# lambda を使う書き方（同じ意味）
square2 = lambda x: x ** 2

print(square(5))    # 25
print(square2(5))   # 25
```

出力:

```text
25
25
```

読み方は **`lambda 引数: 返す値`**。`return` は書きません。単独で使うより、次の `sorted` などと**組み合わせて**使うのが本領です。なお、`square2 = lambda x: ...` のように**名前をつけて何度も使うなら `def` で書くのが Python の推奨**です（PEP 8）。ラムダは `sorted` の `key` など「その場で1回渡す」ときに使いましょう。

---

## 2. sorted の key：好きな基準で並べ替える

タプルのリストを、「2番目の値」で並べ替えたいとします。`sorted(..., key=...)` に「何で比べるか」をラムダで渡します。

```python
# (ポリマー名, 引張強さ [MPa])
data = [("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]

# 引張強さ（各要素の [1]）の大きい順に並べる
ranked = sorted(data, key=lambda x: x[1], reverse=True)
print(ranked)
```

出力:

```text
[('Nylon6', 80), ('PMMA', 70), ('PP', 35), ('PE', 25)]
```

`key=lambda x: x[1]` は「各要素 x の、2番目の値 `x[1]` を基準にして」という意味。`reverse=True` で大きい順（降順）になります。

!!! tip "key はいろいろ指定できる"
    ```python
    sorted(words, key=len)              # 文字数で並べ替え
    sorted(data, key=lambda x: x[0])    # 名前（1番目）で並べ替え
    ```

---

## 3. map：全要素をまとめて変換

`map(関数, リスト)` は、リストの各要素に関数を適用します。結果は `list()` で取り出します。

```python
tensile = [25, 35, 45]   # 引張強さ [MPa]

# 各値を2倍する
doubled = list(map(lambda x: x * 2, tensile))
print(doubled)
```

出力:

```text
[50, 70, 90]
```

!!! note "map と内包表記"
    `list(map(lambda x: x*2, tensile))` は、内包表記 `[x*2 for x in tensile]`（第13回）と同じ結果です。どちらでもOK。内包表記のほうが読みやすいことが多いですが、`map` は他の関数と組み合わせるときに便利です。

---

## 4. filter：条件で絞り込む

`filter(条件の関数, リスト)` は、条件が True の要素だけを残します。

```python
data = [("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]   # 引張強さ [MPa]

# 引張強さが 40 MPa より大きいものだけ
strong = list(filter(lambda p: p[1] > 40, data))
print(strong)
```

出力:

```text
[('PMMA', 70), ('Nylon6', 80)]
```

`lambda p: p[1] > 40` が各要素に対して True / False を返し、True のものだけが残ります（内包表記の `if` と同じ発想）。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「`[("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]` という (ポリマー名, 引張強さ[MPa]) のタプルのリストがあります。
    (1) 引張強さの大きい順に `sorted(..., key=lambda ...)` で並べ替え、
    (2) `map` で引張強さを GPa に換算したリスト（1 GPa = 1000 MPa）を作り、
    (3) `filter` で 40 MPa より大きいものだけ取り出してください。pandas は使わないでください。」

!!! warning "AIの出力で確かめること"
    - `key=lambda x: x[1]` の添字が、比べたい値（引張強さ）を指しているか。`x[0]` だと名前のアルファベット順になります。並べ替え結果の先頭が最大値（Nylon6, 80）か確認する。
    - 「大きい順」なら `reverse=True` が付いているか。
    - 単位換算の係数が正しいか（MPa → GPa は 1/1000）。25 MPa → 0.025 GPa になるか1つ手計算で確かめる。
    - `map` / `filter` の結果を `list()` で取り出しているか。`print(map(...))` だと `<map object at ...>` と表示されるだけです。
    - 同じ `map` / `filter` の結果を2回使っていないか（1回取り出すと空になります）。

---

## 演習問題

**問1.** `lambda` で「摂氏を華氏に変換する」関数 `to_f = lambda c: c * 9/5 + 32` を作り、`to_f(100)` と `to_f(0)` を表示してください。

**問2.** `(分子名, 分子量)` のリスト `mols = [("water", 18.0), ("glucose", 180.2), ("ethanol", 46.1)]` を、**分子量の大きい順**に `sorted` で並べ替えてください。

**問3.** `filter` を使って、`mols` から**分子量が50より大きい**分子だけを取り出してください。

**問4.** 次は AI が書いたコードです。ポリマーを引張強さの強い順に並べ、引張強さを GPa に換算するつもりです。問題点を2つ指摘し、直してください。
```python
# (ポリマー名, 引張強さ [MPa])
data = [("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]

ranked = sorted(data, key=lambda x: x[0], reverse=True)   # 強い順
print(ranked)

gpa = list(map(lambda x: x[1] / 100, data))               # MPa → GPa
print(gpa)
```

出力:
```text
[('PP', 35), ('PMMA', 70), ('PE', 25), ('Nylon6', 80)]
[0.25, 0.7, 0.35, 0.8]
```

---

## 解答

??? success "問1 の解答"
    ```python
    to_f = lambda c: c * 9/5 + 32
    print(to_f(100))   # 212.0
    print(to_f(0))     # 32.0
    ```

    出力:
    ```text
    212.0
    32.0
    ```

??? success "問2 の解答"
    ```python
    mols = [("water", 18.0), ("glucose", 180.2), ("ethanol", 46.1)]
    ranked = sorted(mols, key=lambda m: m[1], reverse=True)
    print(ranked)
    ```

    出力:
    ```text
    [('glucose', 180.2), ('ethanol', 46.1), ('water', 18.0)]
    ```

??? success "問3 の解答"
    ```python
    mols = [("water", 18.0), ("glucose", 180.2), ("ethanol", 46.1)]
    heavy = list(filter(lambda m: m[1] > 50, mols))
    print(heavy)
    ```

    出力:
    ```text
    [('glucose', 180.2)]
    ```

??? success "問4 の解答"
    **誤り1：`key=lambda x: x[0]` は名前（1番目）で並べています。** 出力が PP → PMMA → PE → Nylon6 と、アルファベットの逆順になっていることで分かります。引張強さは `x[1]` です。

    **誤り2：MPa → GPa は 1000 で割ります（1 GPa = 1000 MPa）。** 100 で割ると10倍大きな値になります。PE の 25 MPa が 0.25 GPa になっていたら誤りで、正しくは 0.025 GPa です。

    ```python
    # (ポリマー名, 引張強さ [MPa])
    data = [("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]

    ranked = sorted(data, key=lambda x: x[1], reverse=True)   # 強い順
    print(ranked)

    gpa = list(map(lambda x: x[1] / 1000, data))              # MPa → GPa
    print(gpa)
    ```

    出力:
    ```text
    [('Nylon6', 80), ('PMMA', 70), ('PP', 35), ('PE', 25)]
    [0.025, 0.07, 0.035, 0.08]
    ```

---

## この回のまとめ

- `lambda 引数: 返す値` … その場で使う名前なしの小さな関数。
- `sorted(データ, key=lambda x: x[1], reverse=True)` … 好きな基準で並べ替え。
- `map(関数, リスト)` … 全要素を変換（内包表記でも書ける）。
- `filter(条件, リスト)` … 条件に合う要素だけ抽出。

### 次回予告

[第15回：クラス入門](lesson-15.md) では、分子を「オブジェクト」として表す方法を学びます。データと処理をひとまとめにできます。
