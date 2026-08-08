# 第12回　ファイルの読み書き：実験データをテキストで保存する

!!! abstract "この回のゴール"
    - `open()` と `with` でファイルを安全に開く
    - テキストファイルに**書き込む**／**読み込む**
    - 1行ずつ処理する
    - 所要時間の目安: 60分
    - 使うテーマ：**実験メモ**の保存と読み戻し

計算結果や測定値を、次回また使えるようファイルに残しましょう。pandas を使わない素のファイル操作は、あらゆる場面の土台です。

`lesson12.py` を作りましょう。

---

## 1. ファイルに書き込む

`with open(...) as f:` でファイルを開き、`f.write()` で書き込みます。`"w"` は「書き込みモード（write）」です。

```python
with open("experiment.txt", "w", encoding="utf-8") as f:
    f.write("実験ログ\n")            # \n は改行
    f.write("run1 収率 88.5%\n")
    f.write("run2 収率 91.2%\n")

print("保存しました")
```

出力:

```text
保存しました
```

実行すると `experiment.txt` ができ、3行のテキストが入っています。

!!! warning "`with` を使う理由 / `\n` を忘れない"
    - `with open(...) as f:` は、ブロックを抜けると**自動でファイルを閉じます**（閉じ忘れによる書き込み失敗を防ぐ）。`with` を使うのが定石です。
    - 改行は自動で入りません。行末に **`\n`** を書かないと、全部が1行につながってしまいます。
    - `"w"` は**上書き**。既存の内容は消えます。追記したいときは `"a"`（append）。

---

## 2. ファイルを読み込む

`"r"`（read）で開き、`f.read()` で全体を、`f.readlines()` で行のリストを取得します。

```python
with open("experiment.txt", "r", encoding="utf-8") as f:
    content = f.read()

print(content)
```

出力:

```text
実験ログ
run1 収率 88.5%
run2 収率 91.2%
```

---

## 3. 1行ずつ処理する

ファイルは for ループで**1行ずつ**回せます。データを1行ずつ解析するときの基本形です。

```python
with open("experiment.txt", "r", encoding="utf-8") as f:
    for line in f:
        clean = line.strip()          # 行末の改行を除く
        print("読み取り:", clean)
```

出力:

```text
読み取り: 実験ログ
読み取り: run1 収率 88.5%
読み取り: run2 収率 91.2%
```

!!! note "`strip()` で改行を除く"
    ファイルから読んだ各行の末尾には改行 `\n` が付いています。`line.strip()` で前後の空白・改行を取り除いてから使うのが定番です（第9回の文字列操作）。

---

## 4. CSV風データを書き出す（pandasを使わずに）

第4部では pandas で CSV を扱いましたが、素の Python でも書けます。仕組みを知っておくと応用が利きます。

```python
data = [("run1", 88.5), ("run2", 91.2), ("run3", 90.0)]

with open("yields.csv", "w", encoding="utf-8") as f:
    f.write("run,yield_pct\n")          # ヘッダ
    for name, y in data:
        f.write(f"{name},{y}\n")         # カンマ区切りで1行ずつ

print("yields.csv を書き出しました")
```

出力:

```text
yields.csv を書き出しました
```

できた `yields.csv` は、そのまま `pd.read_csv("yields.csv")` で読み込めます。

---

## 演習問題

**問1.** テキストファイル `memo.txt` を作り、好きな3行（例：実験条件のメモ）を書き込んでください。改行 `\n` を忘れずに。

**問2.** `memo.txt` を読み込み、`f.read()` で全体を表示してください。次に、for ループで1行ずつ `strip()` して表示してください。

**問3.** 測定値のリスト `data = [("A", 12.5), ("B", 13.1), ("C", 12.8)]` を、`sample,value` というヘッダつきの CSV ファイル `measurements.csv` に書き出してください。書き出した後、テキストとして読み込んで中身を確認しましょう。

---

## 解答

??? success "問1 の解答"
    ```python
    with open("memo.txt", "w", encoding="utf-8") as f:
        f.write("触媒: Pd\n")
        f.write("温度: 80C\n")
        f.write("溶媒: エタノール\n")
    print("memo.txt を保存しました")
    ```

??? success "問2 の解答"
    ```python
    with open("memo.txt", "r", encoding="utf-8") as f:
        print(f.read())

    with open("memo.txt", "r", encoding="utf-8") as f:
        for line in f:
            print("行:", line.strip())
    ```

    出力:
    ```text
    触媒: Pd
    温度: 80C
    溶媒: エタノール

    行: 触媒: Pd
    行: 温度: 80C
    行: 溶媒: エタノール
    ```

??? success "問3 の解答"
    ```python
    data = [("A", 12.5), ("B", 13.1), ("C", 12.8)]

    with open("measurements.csv", "w", encoding="utf-8") as f:
        f.write("sample,value\n")
        for name, v in data:
            f.write(f"{name},{v}\n")

    with open("measurements.csv", "r", encoding="utf-8") as f:
        print(f.read())
    ```

    出力:
    ```text
    sample,value
    A,12.5
    B,13.1
    C,12.8
    ```

---

## この回のまとめ

- `with open("名前", "モード", encoding="utf-8") as f:` で安全に開く。
- モード：`"w"`（上書き）/ `"a"`（追記）/ `"r"`（読み込み）。
- 書き込みは `f.write("...\n")`（改行 `\n` を明記）。
- 読み込みは `f.read()`（全体）/ for ループ（1行ずつ、`strip()` で改行除去）。
- 素の Python でも CSV は書ける。仕組みを知ると応用が利く。

### 次回予告

[第13回：内包表記](lesson-13.md) では、リストをスマートに作る「内包表記」を学びます。たくさんの分子量を一気に計算できます。
