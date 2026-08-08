# 第20回　まとめ演習：実験データ整理ツールをつくる

!!! abstract "この回のゴール"
    - 第2部で学んだ **クラス・内包表記・関数・ファイル** を全部使う
    - 実験記録を整理して集計し、レポートに書き出すツールを作る
    - 所要時間の目安: 60分（第2部の総仕上げ）
    - 使うテーマ：**触媒実験の記録整理**

`experiment_organizer.py` を作りましょう。複数回の実験記録を、クラスでまとめ、集計し、ファイルに保存するツールを作ります。第1部・第2部の集大成です。

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
    """1回の実験（複数回の収率測定を持つ）"""

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

保存された `experiment_report.csv` は、そのまま第4部の `pd.read_csv()` で読み込んで、さらに集計・可視化できます。

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

## 演習問題

**問1.** `Experiment` クラスに、収率の**ばらつき（標準偏差）**を返すメソッド `stdev` を追加してください（ヒント：`import statistics` して `statistics.stdev(self.yields)`。ただし測定が1回だとエラーになるので、2回以上ある前提で）。

**問2.** 全実験の中から、**最高収率（best）が最も高い実験**を `max(..., key=...)` で探して表示してください。

**問3.** 新しい実験 `Experiment("exp4", "Pd", [92.0, 93.5, 91.8])` を `experiments` に追加し、ツールを実行し直して、サマリーと保存ファイルに反映されることを確認してください。

---

## 解答

??? success "問1 の解答"
    ```python
    import statistics

    # Experiment クラスにメソッドを追加
    def stdev(self):
        """収率の標準偏差を返す（測定2回以上が前提）"""
        return round(statistics.stdev(self.yields), 2)

    # 例：exp1 に対して
    exp1 = Experiment("exp1", "Pd", [88.5, 90.1, 89.3])
    print("標準偏差:", round(statistics.stdev(exp1.yields), 2))
    ```

    出力:
    ```text
    標準偏差: 0.8
    ```

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

---

## 第1部・第2部の修了、おめでとう！

ここまでで、プログラミングの土台が完成しました。**データを持ち、条件で分け、繰り返し、部品（関数・クラス）にまとめ、ファイルに残す。** この基礎の上に、第4部（集計）・第5部（可視化）が乗っています。あなたはもう、化学データを扱うための道具を一通り手にしています。

### 次回予告

このあとは、飛ばしていた **第3部：NumPy（数値計算）** を追加していきます。配列を使った高速な計算で、濃度計算や滴定曲線などを扱います。ここまで本当によく頑張りました！
