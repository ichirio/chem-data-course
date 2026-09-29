# 第20回　まとめ演習：実験データ整理ツールをつくる

!!! abstract "この回のゴール"
    - 第2部で学んだ **クラス・内包表記・関数・ファイル** を全部使う
    - 実験記録を整理して集計し、レポートに書き出すツールを作る
    - 所要時間の目安: 60分（第2部の総仕上げ）
    - 使うテーマ：**触媒実験の記録整理**

`experiment_organizer.py` を作りましょう。複数回の実験記録を、クラスでまとめ、集計し、ファイルに保存するツールを作ります。第1部・第2部の集大成です。（データは架空です。同じ基質の反応で、Pd・Pt・Ni の3種の金属触媒を比べた場面を想定しています。）

---

## 1. 設計

!!! example "作るもの：実験データ整理ツール"
    1. 1つの実験を **`Experiment` クラス**で表す（名前・触媒・複数回の収率）
    2. 各実験の**平均収率・最高収率**をメソッドで計算する
    3. 全実験の集計を**内包表記**でまとめる
    4. 結果を画面に表示し、**ファイルに保存**する

必要な部品はすべて第2部までで学びました。

---

## 2. 完成版コード（そのままコピペで動きます）

```python title="experiment_organizer.py"
# ===== 実験データ整理ツール =====

class Experiment:
    """1つの実験条件（同じ条件で繰り返した収率測定を持つ）"""

    def __init__(self, name, catalyst, yields):
        self.name = name
        self.catalyst = catalyst
        self.yields = yields          # 収率のリスト [%]

    def average(self):
        """平均収率を返す"""
        return round(sum(self.yields) / len(self.yields), 2)

    def best(self):
        """最高収率を返す"""
        return max(self.yields)


# --- 実験記録を作る ---
experiments = [
    Experiment("exp1", "Pd", [88.5, 90.1, 89.3]),
    Experiment("exp2", "Pt", [70.2, 72.5, 71.0]),
    Experiment("exp3", "Ni", [58.0, 60.5]),
]

# --- 集計（内包表記でまとめる）---
summary = [(e.name, e.catalyst, e.average(), e.best()) for e in experiments]

# --- 画面に表示 ---
print("=== 実験サマリー ===")
for name, catalyst, avg, best in summary:
    print(f"{name} ({catalyst}): 平均 {avg}%, 最高 {best}%")

# --- 最も平均が高い実験を探す ---
best_exp = max(experiments, key=lambda e: e.average())
print(f"\n最良の平均収率: {best_exp.name}（{best_exp.catalyst}）{best_exp.average()}%")

# --- ファイルに保存 ---
with open("experiment_report.csv", "w", encoding="utf-8") as f:
    f.write("name,catalyst,average,best\n")
    for name, catalyst, avg, best in summary:
        f.write(f"{name},{catalyst},{avg},{best}\n")

print("\nexperiment_report.csv に保存しました")
```

### 実行結果

```text
=== 実験サマリー ===
exp1 (Pd): 平均 89.3%, 最高 90.1%
exp2 (Pt): 平均 71.23%, 最高 72.5%
exp3 (Ni): 平均 59.25%, 最高 60.5%

最良の平均収率: exp1（Pd）89.3%

experiment_report.csv に保存しました
```

保存された `experiment_report.csv` は、第4部（第32回）で学ぶ `pd.read_csv()` でそのまま読み込んで、さらに集計・可視化できます。

!!! success "全部つながった！"
    使った道具を数えてみましょう——**クラス**（Experiment）、**メソッド**（average/best）、**内包表記**（summary）、**ラムダ**（key）、**ファイル書き込み**、**f-string**、**関数**。第1部・第2部で学んだことが、1つのツールとして結実しました。

---

## 3. 部品ごとに読み解く

長く見えても、部品に分ければ簡単です。

1. `class Experiment` … データ（名前・触媒・収率）と処理（average・best）をまとめた設計図（第15回）
2. `summary = [... for e in experiments]` … 各実験を集計してまとめる（第13回）
3. `max(experiments, key=lambda e: e.average())` … 平均が最大の実験を探す（第14回）
4. `with open(...) as f:` … 結果をCSVに保存（第12回）

!!! tip "自分のデータでやってみる"
    `experiments` のリストを、あなた自身の実験・観察データに置き換えてみましょう。動くツールを"自分ごと"にすると、理解が一気に深まります。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「実験記録を整理する `Experiment` クラスを書いてください。属性は name・catalyst・yields（収率[%]のリスト、測定回数は実験ごとに2〜3回で異なる）です。
    メソッドは average（平均、小数第2位で丸め）・best（最高）・stdev（標本標準偏差）。測定回数は `len` で数え、固定値を使わないこと。
    全実験の (name, catalyst, average, best) を内包表記でまとめ、ヘッダ `name,catalyst,average,best` つきの CSV に `encoding="utf-8"` で保存してください。pandas は使わないでください。」

!!! warning "AIの出力で確かめること"
    - 平均の割る数が `len(self.yields)` になっているか。`/ 3` のような固定値だと、測定が2回の実験で平均がおかしくなります。**平均が必ず最小値〜最大値の間に入るか**を print して確かめる。
    - 追加したメソッドが**クラスの中**（字下げされた位置）に書かれているか。`exp1.stdev()` と呼んで動くか試す。
    - `max(..., key=lambda e: e.average())` のように `()` を付けて呼んでいるか。「最良の実験」が画面のサマリーと一致しているか目で確かめる。
    - 保存した CSV を開き、行数が実験数＋1（ヘッダ）になっているか。`"a"` モードで何度も実行して行が重複していないか。
    - 収率が 0〜100 % の範囲に入っているか（100 % を超える値が紛れ込んでいないか）。

---

## 演習問題

**問1.** `Experiment` クラスに、収率の**ばらつき（標準偏差）**を返すメソッド `stdev` を追加してください（ヒント：`import statistics` して `statistics.stdev(self.yields)`。ただし測定が1回だとエラーになるので、2回以上ある前提で）。

**問2.** 全実験の中から、**最高収率（best）が最も高い実験**を `max(..., key=...)` で探して表示してください。

**問3.** 新しい実験 `Experiment("exp4", "Pd", [92.0, 93.5, 91.8])` を `experiments` に追加し、ツールを実行し直して、サマリーと保存ファイルに反映されることを確認してください。

**問4.** 次は AI が書いた `Experiment` クラスの一部と実行結果です。エラーは出ませんが、1つの実験の平均収率がおかしな値になっています。どれがおかしいかを見抜き、原因を指摘して直してください。また、同じ種類の誤りを自動で見つけるための確認も加えてください。
```python
class Experiment:
    def __init__(self, name, catalyst, yields):
        self.name = name
        self.catalyst = catalyst
        self.yields = yields          # 収率のリスト [%]

    def average(self):
        """平均収率を返す"""
        return round(sum(self.yields) / 3, 2)   # 3回測定の平均


experiments = [
    Experiment("exp1", "Pd", [88.5, 90.1, 89.3]),
    Experiment("exp2", "Pt", [70.2, 72.5, 71.0]),
    Experiment("exp3", "Ni", [58.0, 60.5]),
]
for e in experiments:
    print(f"{e.name} ({e.catalyst}): 平均 {e.average()}%")
```

出力:
```text
exp1 (Pd): 平均 89.3%
exp2 (Pt): 平均 71.23%
exp3 (Ni): 平均 39.5%
```

---

## 解答

??? success "問1 の解答"
    ```python
    import statistics


    class Experiment:
        """1つの実験条件（同じ条件で繰り返した収率測定を持つ）"""

        def __init__(self, name, catalyst, yields):
            self.name = name
            self.catalyst = catalyst
            self.yields = yields          # 収率のリスト [%]

        def average(self):
            """平均収率を返す"""
            return round(sum(self.yields) / len(self.yields), 2)

        def best(self):
            """最高収率を返す"""
            return max(self.yields)

        def stdev(self):
            """収率の標準偏差（標本標準偏差, n-1 で割る）を返す（測定2回以上が前提）"""
            return round(statistics.stdev(self.yields), 2)


    exp1 = Experiment("exp1", "Pd", [88.5, 90.1, 89.3])
    print("標準偏差:", exp1.stdev())
    ```

    出力:
    ```text
    標準偏差: 0.8
    ```
    メソッドは**クラスの中に**（`def` をクラスと同じ字下げの1段内側に）書きます。`statistics.stdev` は n−1 で割る**標本標準偏差**です（n で割る母標準偏差は `statistics.pstdev`）。

??? success "問2 の解答"
    ```python
    best_by_max = max(experiments, key=lambda e: e.best())
    print(f"最高収率が一番高い: {best_by_max.name}（{best_by_max.catalyst}）{best_by_max.best()}%")
    ```

    出力（exp4 追加前）:
    ```text
    最高収率が一番高い: exp1（Pd）90.1%
    ```

??? success "問3 の解答"
    ```python
    experiments.append(Experiment("exp4", "Pd", [92.0, 93.5, 91.8]))
    # そのまま summary 以降を再実行する
    ```
    サマリーに `exp4 (Pd): 平均 92.43%, 最高 93.5%` の行が加わり、`experiment_report.csv` にも1行増えます。最良の平均収率も exp4 に変わります。

??? success "問4 の解答"
    **おかしいのは exp3 です。** 測定値は 58.0 と 60.5 なのに、平均が 39.5 % と、**最小値より小さく**なっています。平均は必ず最小値〜最大値の間に入るはずです。

    **原因：割る数が `3` に固定されています。** exp3 は測定が2回しかないので、(58.0 + 60.5) / 3 = 39.5 になりました。exp1・exp2 はたまたま3回なので正しく見え、見落としやすい誤りです。測定回数は `len(self.yields)` で数えます。

    ```python
    class Experiment:
        def __init__(self, name, catalyst, yields):
            self.name = name
            self.catalyst = catalyst
            self.yields = yields          # 収率のリスト [%]

        def average(self):
            """平均収率を返す（測定回数は len で数える）"""
            return round(sum(self.yields) / len(self.yields), 2)


    experiments = [
        Experiment("exp1", "Pd", [88.5, 90.1, 89.3]),
        Experiment("exp2", "Pt", [70.2, 72.5, 71.0]),
        Experiment("exp3", "Ni", [58.0, 60.5]),
    ]
    for e in experiments:
        avg = e.average()
        ok = min(e.yields) <= avg <= max(e.yields)   # 平均は必ず最小〜最大の間
        print(f"{e.name} ({e.catalyst}): n={len(e.yields)}, 平均 {avg}%, 範囲内: {ok}")
    ```

    出力:
    ```text
    exp1 (Pd): n=3, 平均 89.3%, 範囲内: True
    exp2 (Pt): n=3, 平均 71.23%, 範囲内: True
    exp3 (Ni): n=2, 平均 59.25%, 範囲内: True
    ```
    元のコードにこの「範囲内」チェックを付けていれば、exp3 で `False` が出て誤りに気づけます。

---

## 第1部・第2部の修了、おめでとう！

ここまでで、プログラミングの土台が完成しました。**データを持ち、条件で分け、繰り返し、部品（関数・クラス）にまとめ、ファイルに残す。** この基礎の上に、第3部（NumPy）・第4部（pandas での集計）・第5部（可視化）が続きます。あなたはもう、化学データを扱うための道具を一通り手にしています。

### 次回予告

[第21回：NumPy入門](lesson-21.md) から **第3部：数値計算と NumPy** が始まります。配列を使った高速な計算で、濃度計算や滴定曲線などを扱います。ここまで本当によく頑張りました！
