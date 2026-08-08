# 第14回　ラムダと高階関数：データ変換の基本

!!! abstract "この回のゴール"
    - **ラムダ式**（使い捨ての小さな関数）を書く
    - `sorted` の `key` で、好きな基準で並べ替える
    - `map` / `filter` で変換・抽出する
    - 所要時間の目安: 60分
    - 使うテーマ：**ポリマー物性**の並べ替え・変換

第37回で `apply(lambda x: ...)` を使いました。今回はその「ラムダ」を正面から学びます。

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

読み方は **`lambda 引数: 返す値`**。`return` は書きません。単独で使うより、次の `sorted` などと**組み合わせて**使うのが本領です。

---

## 2. sorted の key：好きな基準で並べ替える

タプルのリストを、「2番目の値」で並べ替えたいとします。`sorted(..., key=...)` に「何で比べるか」をラムダで渡します。

```python
# (ポリマー名, 引張強さ)
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
tensile = [25, 35, 45]

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
data = [("PE", 25), ("PMMA", 70), ("PP", 35), ("Nylon6", 80)]

# 引張強さが 40 より大きいものだけ
strong = list(filter(lambda p: p[1] > 40, data))
print(strong)
```

出力:

```text
[('PMMA', 70), ('Nylon6', 80)]
```

`lambda p: p[1] > 40` が各要素に対して True / False を返し、True のものだけが残ります（内包表記の `if` と同じ発想）。

---

## 演習問題

**問1.** `lambda` で「摂氏を華氏に変換する」関数 `to_f = lambda c: c * 9/5 + 32` を作り、`to_f(100)` と `to_f(0)` を表示してください。

**問2.** `(分子名, 分子量)` のリスト `mols = [("water", 18.0), ("glucose", 180.2), ("ethanol", 46.1)]` を、**分子量の大きい順**に `sorted` で並べ替えてください。

**問3.** `filter` を使って、`mols` から**分子量が50より大きい**分子だけを取り出してください。

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
    heavy = list(filter(lambda m: m[1] > 50, mols))
    print(heavy)
    ```

    出力:
    ```text
    [('glucose', 180.2)]
    ```

---

## この回のまとめ

- `lambda 引数: 返す値` … その場で使う名前なしの小さな関数。
- `sorted(データ, key=lambda x: x[1], reverse=True)` … 好きな基準で並べ替え。
- `map(関数, リスト)` … 全要素を変換（内包表記でも書ける）。
- `filter(条件, リスト)` … 条件に合う要素だけ抽出。

### 次回予告

[第15回：クラス入門](lesson-15.md) では、分子を「オブジェクト」として表す方法を学びます。データと処理をひとまとめにできます。
