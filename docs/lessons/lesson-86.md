# 第86回　R Markdown / Quartoで帳票・レポート作成

!!! abstract "この回のゴール"
    - コード・図・文章を1つにまとめた文書（帳票）を作る
    - R Markdown / Quarto の仕組みを知る
    - HTML / PDF レポートに書き出す
    - 所要時間の目安: 60分
    - 使うテーマ：**分析結果の報告書**

分析したら、結果を**報告書（帳票）**にまとめます。**Quarto**（および R Markdown）を使うと、「コード＋実行結果＋図＋文章」を1つの文書にして、HTML や PDF に出力できます。手作業のコピペが不要になり、再現性も保たれます。

!!! info "Quarto について"
    Quarto は R Markdown の後継で、R でも Python でも使える文書システムです。第1回で入れた環境に含まれることが多く、なければ [quarto.org](https://quarto.org/) から導入します。VS Code の拡張でも使えます。

---

## 1. Quarto 文書の構造

Quarto 文書（`.qmd` ファイル）は、3つの要素でできています。

1. **YAML ヘッダ**（`---` で囲む）… タイトルや出力形式の設定
2. **文章**（Markdown）… 見出し・説明文
3. **コードチャンク**（\`\`\`{r} で囲む）… 実行される R コード

```markdown
---
title: "触媒スクリーニング報告書"
author: "山田太郎"
format: html
---

## 実験の概要

3種類の触媒について収率を測定した。

## データと集計

```{r}
library(tidyverse)
df <- tibble(
  catalyst = c("Pd", "Pt", "Ni"),
  yield = c(85, 78, 62)
)
df %>% summarise(mean_yield = mean(yield))
```

## グラフ

```{r}
ggplot(df, aes(catalyst, yield, fill = catalyst)) +
  geom_col() + theme_minimal()
```

以上より、Pd の収率が最も高かった。
```

---

## 2. レポートに変換する

`.qmd` ファイルを HTML に変換（レンダリング）します。ターミナルで:

```bash
quarto render report.qmd
```

すると `report.html` ができ、**コードの実行結果や図が埋め込まれた報告書**が完成します。ブラウザで開けます。PDF にしたいときは、YAML の `format: html` を `format: pdf` に変えます（PDF出力にはLaTeX環境が必要）。

!!! success "帳票作成が自動化される"
    データが更新されたら、`quarto render` を再実行するだけで報告書が最新になります。「表をコピペ→図を貼り付け→数字を手打ち」という手作業が消え、**ミスがなくなり、再現性が保たれます**。定例レポートや実験報告書に絶大な効果があります。

---

## 3. VS Code / RStudio での使い方

- **VS Code**：Quarto 拡張を入れ、`.qmd` を開いて「Render」ボタン。
- **RStudio**：`.qmd` や `.Rmd` を開いて「Knit / Render」ボタン。
- コードチャンクは1つずつ実行して確認しながら書けます（第16回の対話的スタイル）。

!!! note "R Markdown（.Rmd）との関係"
    従来からある **R Markdown（.Rmd）** も同じ発想で、`rmarkdown::render("report.Rmd")` で変換します。Quarto はその進化版で、書き方もほぼ同じ。新しく始めるなら Quarto がおすすめです。

---

## 演習問題

**問1.** `report.qmd` を作り、YAMLヘッダ（title, format: html）と見出し1つ、簡単な R コードチャンク（`1 + 1` など）を書いて、`quarto render report.qmd` で HTML に変換してください。

**問2.** レポートに、tibble を作って `summarise` で平均を出すコードチャンクと、その結果への説明文を加えてください。

**問3.** レポートに ggplot2 のグラフを描くコードチャンクを加え、レンダリングして図が埋め込まれることを確認してください。

---

## 解答

??? success "問1〜問3 の解答（統合例）"
    次の内容で `report.qmd` を作成します。
    ````markdown
    ---
    title: "実験レポート"
    format: html
    ---

    # 収率の分析

    3つの触媒の収率をまとめる。

    ```{r}
    library(tidyverse)
    df <- tibble(catalyst = c("Pd","Pt","Ni"), yield = c(85, 78, 62))
    df %>% summarise(mean_yield = mean(yield))
    ```

    平均収率は上記のとおり。以下にグラフを示す。

    ```{r}
    ggplot(df, aes(catalyst, yield, fill = catalyst)) +
      geom_col() + theme_minimal()
    ```
    ````
    ターミナルで `quarto render report.qmd` を実行すると、集計結果とグラフが埋め込まれた `report.html` が生成されます。ブラウザで開いて確認しましょう。

---

## この回のまとめ

- Quarto（`.qmd`）で「コード＋結果＋図＋文章」を1文書に。
- 構造：YAMLヘッダ ＋ Markdown文章 ＋ `{r}` コードチャンク。
- `quarto render report.qmd` で HTML/PDF に変換。
- データ更新→再レンダリングで帳票が自動更新。手作業とミスが消える。

### 次回予告

[第87回：RとPythonの連携・使い分け](lesson-87.md) では、RとPythonを行き来する方法（reticulate）と、両者の使い分けを学びます。
