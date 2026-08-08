# 第10回　エラーと例外処理：入力ミスに強いプログラム

!!! abstract "この回のゴール"
    - よくある**エラーの種類**を読めるようになる
    - `try` / `except` で、エラーが起きても**止まらない**ようにする
    - 特定のエラーだけを捕まえる／`raise` で自分からエラーを出す
    - ユーザー入力や実験データの「想定外」に強いコードを書く
    - 所要時間の目安: 60分

`lesson10.py` を作って進めましょう。

---

## 1. エラーは「悪」ではない — メッセージを読もう

プログラムが止まると赤い文字（エラー）が出ます。これは**「ここで、こういう理由で困りました」という案内**です。こわがらず、最後の行を読むのがコツです。

```python
mass = float("abc")     # 数にできない文字を変換しようとした
```

出力（一部）:

```text
ValueError: could not convert string to float: 'abc'
```

よく出会うエラーを知っておきましょう。

| エラー名 | 意味 | 例 |
|---|---|---|
| `NameError` | 未定義の名前を使った | 変数名のタイプミス |
| `TypeError` | 型が合わない | `"3" + 4`（文字＋数） |
| `ValueError` | 型は合うが値がダメ | `float("abc")` |
| `ZeroDivisionError` | 0で割った | `1 / 0` |
| `KeyError` | 辞書に無いキー | `atomic_mass["Xx"]` |
| `IndexError` | リストの範囲外 | `data[999]` |

!!! tip "エラーが出たらAIに丸ごと見せる"
    エラーメッセージを**全部コピー**してAI（Claude / ChatGPT）に貼ると、原因と直し方を教えてくれます。エラー文は「相談のための情報」でもあります。

---

## 2. try / except：止まらないようにする

`try:` の中でエラーが起きたら、プログラムを止めずに `except:` に飛びます。

```python
text = "abc"

try:
    mass = float(text)
    print(f"質量は {mass} g")
except ValueError:
    print("！ 数値を入力してください（例：18.0）")

print("プログラムは続いています")   # ← ちゃんと最後まで動く
```

出力:

```text
！ 数値を入力してください（例：18.0）
プログラムは続いています
```

もし `try/except` が無ければ、`float("abc")` の時点でプログラムは**そこで止まって**しまいます。`except` があるおかげで、やさしいメッセージを出して先へ進めます。

---

## 3. 特定のエラーだけを捕まえる

`except` には**エラーの種類**を書けます。種類ごとに対応を変えられます。

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("！ 0では割れません")
        return None

print(safe_divide(10, 2))    # 5.0
print(safe_divide(10, 0))    # エラーにならず、メッセージ＋None
```

出力:

```text
5.0
！ 0では割れません
None
```

!!! warning "何でも捕まえる `except:` は避ける"
    種類を書かない `except:` は、**本当のバグまで隠して**しまい、原因が分からなくなります。「どんなエラーを想定しているか」を書く（`except ValueError:` など）のが良い習慣です。

---

## 4. 化学の例：知らない元素を安全に扱う

辞書に無い元素を引くと `KeyError` で止まります。`try/except` で「未対応です」とやさしく返しましょう。

```python
ATOMIC_MASS = {"H": 1.008, "C": 12.011, "O": 15.999}

def get_mass(symbol):
    try:
        return ATOMIC_MASS[symbol]
    except KeyError:
        print(f"！ '{symbol}' は原子量表に登録されていません")
        return None

print(get_mass("O"))     # 15.999
print(get_mass("Xx"))    # メッセージ＋None（止まらない）
```

出力:

```text
15.999
！ 'Xx' は原子量表に登録されていません
None
```

---

## 5. raise：自分からエラーを出す

「ありえない値」が来たら、**わざとエラーを出して**気づかせるのも大事な作法です。`raise` を使います。

```python
def moles(mass_g, molar_mass):
    if mass_g < 0:
        raise ValueError("質量は0以上でなければなりません")
    return mass_g / molar_mass

# 正常なとき
print(moles(36.0, 18.015))    # 1.998...

# ありえない入力 → try で受け止める
try:
    print(moles(-5, 18.015))
except ValueError as e:
    print("エラーを検知:", e)
```

出力:

```text
1.9983347210657785
エラーを検知: 質量は0以上でなければなりません
```

!!! note "`as e` でメッセージを受け取る"
    `except ValueError as e:` と書くと、エラーの中身を `e` で受け取れます。`print(e)` でメッセージを表示できます。

---

## 6. 実践：入力を安全に受け取る

第8回のアプリでも役立つ、「数値を入れてもらうまで聞き直す」パターンです。

```python
def ask_number(prompt):
    """数値が入力されるまで聞き直して、float で返す"""
    while True:
        text = input(prompt)
        try:
            return float(text)
        except ValueError:
            print("  数値で入力してください（例：18.0）")

# 使い方（実行すると対話になります）
# mass = ask_number("質量[g]を入力 > ")
# print(f"受け取った質量: {mass} g")
```

実行例:

```text
質量[g]を入力 > いち
  数値で入力してください（例：18.0）
質量[g]を入力 > 18.0
受け取った質量: 18.0 g
```

!!! success "「落ちないプログラム」は親切なプログラム"
    実験データには、空欄・文字の混入・測定ミスがつきものです。`try/except` を使えるようになると、**現実の汚れたデータ**にも耐えるコードが書けます。第4部（pandas）でも大活躍します。

---

## 演習問題

**問1.** 文字列を整数に変換する処理を `try/except` で包み、`int("五")` のように変換できないときは「整数を入力してください」と表示するコードを書いてください（プログラムは止まらないこと）。

**問2.** 2つの数を割る関数 `safe_divide(a, b)` を作り、`b` が 0 のときは `ZeroDivisionError` を捕まえて「0では割れません」と表示し、`None` を返すようにしてください。`safe_divide(10, 0)` で確認しましょう。

**問3.** 温度（ケルビン）を受け取る関数 `check_kelvin(k)` を作り、`k` が**負の値**なら `raise ValueError("絶対温度は負になりません")` でエラーを出してください。`check_kelvin(-10)` を `try/except` で受け止め、メッセージを表示しましょう。

---

## 解答

??? success "問1 の解答"
    ```python
    text = "五"
    try:
        number = int(text)
        print(f"整数: {number}")
    except ValueError:
        print("整数を入力してください")

    print("（プログラムは続いています）")
    ```

    出力:
    ```text
    整数を入力してください
    （プログラムは続いています）
    ```

??? success "問2 の解答"
    ```python
    def safe_divide(a, b):
        try:
            return a / b
        except ZeroDivisionError:
            print("0では割れません")
            return None

    result = safe_divide(10, 0)
    print("結果:", result)
    ```

    出力:
    ```text
    0では割れません
    結果: None
    ```

??? success "問3 の解答"
    ```python
    def check_kelvin(k):
        if k < 0:
            raise ValueError("絶対温度は負になりません")
        return k

    try:
        check_kelvin(-10)
    except ValueError as e:
        print("エラーを検知:", e)
    ```

    出力:
    ```text
    エラーを検知: 絶対温度は負になりません
    ```

---

## この回のまとめ

- エラーは「困った理由の案内」。**最後の行を読む**（種類とメッセージ）
- `try:` … `except エラー名:` で、止まらずに対処できる
- `except` には**想定する種類を書く**（`except:` だけは避ける）
- `except ... as e:` でエラーの中身を受け取れる
- `raise` で、ありえない値のときに自分からエラーを出す
- 「落ちないプログラム」は、現実の汚れたデータに強い

### 次回予告

[第11回：モジュールとインポート](../roadmap.md) では、`math` などの**標準ライブラリ**を取り込んで、平方根・対数（pH計算に必須！）などを使えるようにします。第1部・第2部前半で土台は完成です。ここまで本当によく頑張りました！
