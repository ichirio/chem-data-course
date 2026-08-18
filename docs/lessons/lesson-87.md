# 第87回　RとPythonの連携・使い分け（reticulate）

!!! abstract "この回のゴール"
    - R と Python の使い分けを整理する
    - **reticulate** で R から Python を呼ぶ方法を知る
    - 2言語を組み合わせるワークフローをイメージする
    - 所要時間の目安: 60分

このコースは Python 主・R 副で進めてきました。実務では**両方を使い分け、時に連携させる**のが最強です。

---

## 1. 使い分けの整理

これまでの学びを踏まえた、実践的な使い分けです。

| やりたいこと | おすすめ | 理由 |
|---|---|---|
| データ処理・自動化・機械学習 | **Python**（pandas / scikit-learn） | 汎用性・エコシステム |
| 分子構造の扱い | **Python（RDKit）** or **R（rcdk）** | どちらも可（第6部・第88回〜） |
| 統計的検定・きれいな統計グラフ | **R**（t.test / ggplot2） | 統計が標準装備・作図が美しい |
| 帳票・レポート | **R（Quarto）** or Python | Quarto はどちらでも |

!!! note "どちらかに寄せてもよい"
    「全部 Python」「全部 R」でも仕事は回ります。大事なのは、**それぞれの得意を知り、必要なら行き来できる**こと。あなたが R に強いなら R 中心、Python に慣れたら Python 中心、で構いません。

---

## 2. reticulate：R から Python を呼ぶ

**reticulate** パッケージを使うと、R の中から Python を実行できます。「データ処理は Python、統計と作図は R」を1つのスクリプトでつなげます。

```r
library(reticulate)

# Python のモジュールを R から使う
np <- import("numpy")
arr <- np$array(c(1, 2, 3, 4))
np$mean(arr)          # Python の numpy.mean を R から呼ぶ
```

`import("numpy")` で Python のライブラリを読み込み、`np$mean(...)` のように `$` でその関数を呼びます。

!!! info "準備：reticulate"
    ```r
    install.packages("reticulate")
    ```
    reticulate は、使う Python を自動で探します（`py_config()` で確認）。第1回で作った conda 環境の Python を指定することもできます。

---

## 3. R Markdown / Quarto で両言語を混ぜる

Quarto（第86回）では、**1つの文書の中で R と Python のチャンクを混在**できます。

````markdown
```{python}
# Python で分子記述子を計算（RDKit）
from rdkit import Chem
from rdkit.Chem import Descriptors
mol = Chem.MolFromSmiles("CC(=O)Oc1ccccc1C(=O)O")
mw = Descriptors.MolWt(mol)
```

```{r}
# R で統計・作図（Python の結果を受け取ることも可能）
cat("分子量は Python で計算しました\n")
```
````

「RDKit（Python）で分子を処理 → その結果を R で統計解析・作図 → Quarto で1つのレポートに」——2言語のいいとこ取りができます。

!!! success "言語の壁を越える"
    reticulate や Quarto を使えば、「Python か R か」の二者択一に縛られません。**プロジェクトごとに最適な道具を、必要なだけ組み合わせる**。これがデータ分析の成熟した姿です。共同作業で片方は R 得意・片方は Python 得意でも、成果物は1つにまとめられます。

---

## 演習問題

**問1.** R と Python の使い分けを、自分の言葉で3つ挙げてください（例：統計検定は R、など）。

**問2.**（reticulate 導入済みなら）R で `library(reticulate)` を読み込み、`import("math")` して `math$sqrt(16)` を実行してください（Python の math.sqrt を R から呼ぶ）。

**問3.** どんな場面で「Python で処理 → R で統計・作図」の連携が役立ちそうか、化学の例を1つ考えてみてください。

---

## 解答

??? success "問1 の解答（例）"
    - **統計的検定（t検定・ANOVA）**は R（標準装備で簡単）。
    - **機械学習・大規模自動化**は Python（scikit-learn 等）。
    - **きれいな統計グラフ・帳票**は R（ggplot2・Quarto）。
    
    （正解は1つではありません。自分の得意と目的で決めてOK。）

??? success "問2 の解答"
    ```r
    library(reticulate)
    math <- import("math")
    math$sqrt(16)      # [1] 4
    ```
    Python の `math.sqrt(16)` が R から呼ばれ、4 が返ります。

??? success "問3 の解答（例）"
    「**RDKit（Python）で大量の化合物の分子記述子を計算** → その表を **R に渡して、物性との相関を統計解析し、ggplot2 で論文用の図を作る**」。分子処理は Python の RDKit が得意、統計と作図は R が得意なので、連携が活きます。

---

## この回のまとめ

- Python（汎用・分子・ML）と R（統計・作図・帳票）を使い分ける。
- **reticulate** で R から Python を呼べる（`import()` ＋ `$`）。
- Quarto では R と Python のチャンクを1文書に混在できる。
- 二者択一でなく、目的ごとに最適な道具を組み合わせる。

### 次回予告

[第88回：Rでケモインフォマティクス入門（rcdk）](lesson-88.md) では、R でも分子（構造式）を扱えることを学びます。第6部の RDKit の R 版です。
