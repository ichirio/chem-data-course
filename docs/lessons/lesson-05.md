# 第5回　繰り返し（for / while）：たくさんの計算を自動化する

!!! abstract "この回のゴール"
    - `for` と `range()` で「決まった回数」繰り返す
    - リストを合計・平均するパターンを身につける
    - `while` で「条件が続くあいだ」繰り返す
    - `break` / `continue` で流れを制御する
    - 実験データの集計や希釈系列の計算を自動化する
    - 所要時間の目安: 60分

`lesson05.py` を作って進めましょう。

---

## 1. range()：決まった回数くり返す

第3回ではリストを for で回しました。今回は `range()` で「回数」を指定します。

```python
for i in range(5):      # 0, 1, 2, 3, 4 の5回
    print(i)
```

出力:

```text
0
1
2
3
4
```

`range` はいろいろな指定ができます。

```python
for i in range(1, 6):       # 1 から 5 まで（6は含まない）
    print(i, end=" ")       # end=" " で改行せず横に並べる
print()                     # 最後に改行

for i in range(0, 101, 10): # 0 から 100 まで 10きざみ
    print(i, end=" ")
print()
```

出力:

```text
1 2 3 4 5 
0 10 20 30 40 50 60 70 80 90 100 
```

!!! note "range のきまり"
    - `range(n)` … 0 から n-1 まで（**n は含まない**）
    - `range(a, b)` … a から b-1 まで
    - `range(a, b, c)` … a から b未満まで c きざみ

---

## 2. 合計と平均：実験データを集計する

繰り返しの一番の定番が「ためていく」計算です。まず入れ物を `0` にしておき、ループで足していきます。

```python
# ある実験を5回くり返して測った pH
measurements = [6.1, 6.4, 6.0, 6.3, 6.2]

total = 0.0
for x in measurements:
    total = total + x        # total += x とも書ける

average = total / len(measurements)

print(f"合計: {total}")
print(f"平均: {average}")
print(f"回数: {len(measurements)}")
```

出力:

```text
合計: 31.0
平均: 6.2
回数: 5
```

!!! tip "`total += x` は便利な書き方"
    `total = total + x` は `total += x` と短く書けます。同様に `-=` `*=` `/=` もあります。よく使うので慣れておきましょう。

!!! note "実は合計・平均は一発でも出せる"
    Python には `sum()` と `len()` があるので `sum(measurements) / len(measurements)` でも平均は出ます。
    でも「ループでためる」考え方は応用が広いので、まずは自分で書けるようになりましょう。

---

## 3. 条件つきで数える：for と if の合わせ技

「基準を超えたデータが何個あるか」を数えてみます。第4回の `if` と組み合わせます。

```python
measurements = [6.1, 6.4, 6.0, 6.3, 6.2]
threshold = 6.2

count = 0
for x in measurements:
    if x >= threshold:
        count += 1

print(f"{threshold} 以上のデータは {count} 個です")
```

出力:

```text
6.2 以上のデータは 3 個です
```

---

## 4. while：条件が続くあいだ繰り返す

回数が決まっていないときは `while`（「〜のあいだ、ずっと」）を使います。
例として**希釈系列**：ある濃度から半分ずつ薄めていき、0.1 mol/L を下回るまで続けます。

```python
concentration = 1.0     # はじめの濃度 [mol/L]
step = 0

while concentration >= 0.1:
    print(f"{step}回目: {concentration:.4f} mol/L")
    concentration = concentration / 2    # 半分に薄める
    step += 1

print(f"→ {step}回の希釈で 0.1 mol/L を下回りました")
```

出力:

```text
0回目: 1.0000 mol/L
1回目: 0.5000 mol/L
2回目: 0.2500 mol/L
3回目: 0.1250 mol/L
→ 4回の希釈で 0.1 mol/L を下回りました
```

!!! danger "無限ループに注意"
    `while` は条件が `True` の間ずっと回り続けます。**ループの中で条件がいつか False になる**ように、必ず値を変えてください（上の例では毎回 `concentration` を半分にしている）。もし止まらなくなったら、ターミナルで `Ctrl` + `C` を押すと止められます。

---

## 5. break と continue：流れを変える

- `break` … ループを**その場で抜ける**
- `continue` … 残りを飛ばして**次のくり返しへ**

```python
values = [5.0, 6.2, -1.0, 7.1, 6.8]

for x in values:
    if x < 0:
        print(f"{x} は異常値なので、ここで測定を打ち切ります")
        break                # ループを抜ける
    print(f"測定値 {x} を記録")
```

出力:

```text
測定値 5.0 を記録
測定値 6.2 を記録
-1.0 は異常値なので、ここで測定を打ち切ります
```

`continue` は「その回だけ飛ばす」ときに使います。

```python
values = [5.0, 6.2, -1.0, 7.1]

total = 0.0
for x in values:
    if x < 0:
        continue             # 異常値は足さずに次へ
    total += x

print(f"異常値を除いた合計: {total}")
```

出力:

```text
異常値を除いた合計: 18.3
```

---

## 演習問題

**問1.** 測定値のリスト `data = [12.5, 13.1, 12.8, 13.0, 12.6, 12.9]` について、for ループで**合計と平均**を計算して表示してください（`sum()` を使わず、自分でためること）。

**問2.** `while` を使って、はじめの濃度 `2.0 mol/L` から**半分ずつ**薄めていき、`0.05 mol/L` を下回るまで各ステップの濃度を表示してください。何回で下回りましたか？

**問3.** リスト `data = [12.5, 13.1, 12.8, 13.0, 12.6, 12.9]` のうち、**平均より大きい値が何個あるか**を数えて表示してください（ヒント：まず平均を求め、次に for と if で数える）。

---

## 解答

??? success "問1 の解答"
    ```python
    data = [12.5, 13.1, 12.8, 13.0, 12.6, 12.9]

    total = 0.0
    for x in data:
        total += x
    average = total / len(data)

    print(f"合計: {total:.1f}")
    print(f"平均: {average:.3f}")
    ```

    出力:
    ```text
    合計: 76.9
    平均: 12.817
    ```

??? success "問2 の解答"
    ```python
    concentration = 2.0
    step = 0

    while concentration >= 0.05:
        print(f"{step}回目: {concentration:.4f} mol/L")
        concentration /= 2
        step += 1

    print(f"→ {step}回で 0.05 mol/L を下回りました")
    ```

    出力:
    ```text
    0回目: 2.0000 mol/L
    1回目: 1.0000 mol/L
    2回目: 0.5000 mol/L
    3回目: 0.2500 mol/L
    4回目: 0.1250 mol/L
    5回目: 0.0625 mol/L
    → 6回で 0.05 mol/L を下回りました
    ```

??? success "問3 の解答"
    ```python
    data = [12.5, 13.1, 12.8, 13.0, 12.6, 12.9]

    average = sum(data) / len(data)   # ここは sum を使ってOK

    count = 0
    for x in data:
        if x > average:
            count += 1

    print(f"平均 {average:.3f} より大きい値は {count} 個です")
    ```

    出力:
    ```text
    平均 12.817 より大きい値は 3 個です
    ```

---

## この回のまとめ

- `for i in range(n)` … 決まった回数くり返す（n は含まない）
- 「入れ物を0にして、ループで `+=` してためる」が集計の基本形
- for と if を組み合わせて「条件に合うものを数える／選ぶ」
- `while 条件:` … 回数が決まらないとき。**条件を必ずいつか False にする**
- `break`（抜ける）／ `continue`（次へ飛ばす）

### 次回予告

[第6回：関数をつくる](lesson-06.md) では、これまで何度も書いた「分子量の計算」を**関数（部品）**にまとめ、どんな分子でも一発で計算できるようにします。
