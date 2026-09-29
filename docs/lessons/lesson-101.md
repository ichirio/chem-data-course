# 第101回　Quartoでレポート・論文を書く（応用）

!!! abstract "この回のゴール"
    - Quarto でコード・図・文章・数式・文献を1つの文書にまとめる
    - 図表番号・相互参照・引用を扱う
    - 研究レポートや論文の形にする
    - 所要時間の目安: 60分

!!! info "第9部スタート：研究・実務に活かす"
    いよいよ最終章です。ここまで学んだ**データ処理・可視化・統計・分子処理・機械学習**を、実際の研究・実務でどう活かすか——レポート作成・再現性・AI活用・データ管理——を学び、総合プロジェクトで締めくくります。

第86回で Quarto の基本を学びました。ここでは、研究レポート・論文にふさわしい機能（図表番号・相互参照・引用）を扱います。

---

## 1. 図表に番号と相互参照をつける

論文では「図1」「表2」のように番号で参照します。Quarto では、コードチャンクに**ラベル**を付けると自動で番号が振られ、本文から参照できます。

````markdown
```{python}
#| label: fig-calibration
#| fig-cap: "検量線：濃度と吸光度の関係"
import matplotlib.pyplot as plt
plt.scatter([0,2,4,6,8,10], [0.02,0.21,0.40,0.59,0.80,0.99])
plt.xlabel("Concentration (mM)"); plt.ylabel("Absorbance")
plt.show()
```

本文中で @fig-calibration のように書くと「図1」に置き換わります。
````

- `#| label: fig-xxx` … 図のラベル（`fig-` で始めると図として扱われる）
- `#| fig-cap: "..."` … 図のキャプション
- 本文の `@fig-xxx` … 「図1」などに自動変換され、番号がずれても自動で追随

!!! note "「図 1」と表示させるには"
    Quarto の既定は英語なので、そのままでは「Figure 1」と表示されます。文書冒頭の YAML に `lang: ja` を書くと「図 1」「表 1」になります。

    ```yaml
    ---
    title: "検量線の作成"
    lang: ja
    ---
    ```

表も同様に `#| label: tbl-xxx` と `#| tbl-cap: "..."` で扱えます（pandas の DataFrame をチャンクの最後に置くと表として出力されます）。**番号を手で管理する必要がなくなり、ミスが消えます**。

---

## 2. 数式を書く

化学の式や統計の式は、LaTeX 記法で書けます（`$...$` で囲む）。

```markdown
反応速度は一次反応 $C = C_0 e^{-kt}$ に従う。

$$ \text{物質量} = \frac{\text{質量}}{\text{モル質量}} $$
```

`$C = C_0 e^{-kt}$` は本文中の数式、`$$...$$` は独立した数式ブロックになります。第2回（物質量）や第55回（反応速度）で扱った式を、そのまま論文に書けます。

---

## 3. 文献を引用する

`.bib`（BibTeX）ファイルに文献情報を用意し、`[@キー]` で引用すると、本文中の引用表記と参考文献リストが自動生成されます。既定は「(Smith 2020)」のような著者-年形式です。化学系の雑誌のような番号形式にしたいときは、YAML に `csl: american-chemical-society.csl` のように引用スタイル（CSL ファイル、Zotero Style Repository などで入手）を指定します。

```markdown
---
title: "触媒スクリーニング"
bibliography: references.bib
---

先行研究では Pd 触媒が有効とされている [@smith2020]。
```

文書末に参考文献リストが自動で作られ、引用の表記（番号や著者-年）も自動管理されます。.bib にない `[@キー]` を書くと、レンダリング時に警告が出て、本文には `(キー?)` のような未解決の表示が残ります。文献管理ソフト（Zotero など）と連携すれば、さらに効率的です。

!!! success "研究文書の自動化"
    **図表番号・相互参照・数式・引用**——論文で手間のかかる部分が、Quarto なら自動化されます。データが変われば図も数字も自動更新（第86回）。「コードから論文まで、1つの文書で・再現可能に」が実現します。卒業論文や学会発表資料にも使えます。

---

## 4. 出力形式いろいろ

同じ `.qmd` から、複数の形式に出力できます。

```bash
quarto render report.qmd --to html    # Webページ
quarto render report.qmd --to pdf     # PDF（LaTeX環境が必要：quarto install tinytex）
quarto render report.qmd --to docx    # Word文書
```

HTML で共有、PDF で提出、Word で共同編集、と使い分けられます。日本語の PDF は、フォントの設定が必要になることがあります（YAML で `pdf-engine: lualatex` と `documentclass: ltjsarticle` を指定するのが手軽です）。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「Quarto の .qmd に入れる Python チャンクを書いてください。`data/calibration.csv`（列 `conc_mg_L`, `absorbance`）を pandas で読み、
    NumPy の polyfit で検量線の傾き・切片・R² を計算し、matplotlib で散布図と回帰直線を描きます。
    チャンクには `#| label: fig-calibration` と `#| fig-cap:` を付け、`#| echo: false` にしてください。
    本文で使う傾きと R² は、手で書かずにインライン式（`` `{python} f"{slope:.4f}"` ``）で埋め込む形にしてください。」

!!! warning "AIの出力で確かめること"
    - **ラベルの接頭辞**：図のラベルが `fig-`、表が `tbl-` で始まっているか。`#| label: calibration` では図番号が付かず、`@calibration` は「文献の引用」と解釈されて未解決の警告になる。
    - **本文の数値が手書きになっていないか**：「傾きは 0.21」のように数値を直接書くと、データを差し替えても更新されない。チャンクで計算した変数がインライン式で埋め込まれているか、レンダリング後の値をチャンクの `print` と照合する。
    - **チャンクの実行順序**：Quarto は上から順に1つの Python で実行する。後ろのチャンクで定義する変数を前で使っていないか、`quarto render` を最初からやり直して確かめる（Jupyter でセルを行き来して動いたコードでも、上から実行すると失敗することがある）。
    - **どの Python で実行されるか**：`quarto check jupyter` で、ふだん使っている環境（pandas・RDKit 入り）の Python が使われているか確認する。
    - **パスと単位**：CSV のパスが .qmd からの相対パスになっているか。軸ラベルに単位（mg/L）があるか。

---

## 演習問題

**問1.** 第86回で作った `report.qmd` に、図のチャンクへ `#| label: fig-xxx` と `#| fig-cap:` を付け、本文から `@fig-xxx` で参照してみてください。レンダリングして図番号が自動で付くことを確認しましょう。

**問2.** レポートに数式を1つ（例：`$C = C_0 e^{-kt}$`）入れて、正しく表示されることを確認してください。

**問3.** `quarto render report.qmd --to docx` で Word 形式に出力してみてください（可能な環境なら）。同じ内容が別形式で出ることを体験しましょう。

**問4.** 次は、検量線の図と説明文を書くよう頼んだときに AI が出した .qmd の一部です。レンダリングすると警告が出て、図に番号が付きませんでした。問題点を2つ指摘し、直してください。

````markdown
```{python}
#| label: calibration
#| fig-cap: "Fe(II)-フェナントロリン錯体の検量線（510 nm）"
import numpy as np
import matplotlib.pyplot as plt
conc = np.array([0.0, 0.5, 1.0, 2.0, 3.0, 4.0])               # Fe 濃度 [mg/L]
absorb = np.array([0.001, 0.099, 0.200, 0.398, 0.601, 0.797])
slope, intercept = np.polyfit(conc, absorb, 1)
plt.scatter(conc, absorb)
plt.plot(conc, slope * conc + intercept)
plt.xlabel("Fe (mg/L)"); plt.ylabel("Absorbance")
plt.show()
```

図1に示すように、検量線の傾きは 0.21 L/mg であった（@calibration）。
````

---

## 解答

??? success "問1〜問3 の解答・確認ポイント"
    - **問1**：`#| label: fig-result` と `#| fig-cap: "結果の図"` をチャンクに付け、本文に `@fig-result` と書きます。レンダリングすると「図 1」のように番号付きで参照されます。図を増やしても番号は自動で振り直されます。
    - **問2**：本文に `反応は $C = C_0 e^{-kt}$ に従う。` と書くと、数式がきれいに表示されます。
    - **問3**：`--to docx` で `.docx` が生成されます。図・表・数式が Word 文書に埋め込まれ、共同編集に使えます（数式やレイアウトは形式により多少変わります）。

??? success "問4 の解答"
    **誤り1（ラベル）**：`#| label: calibration` には `fig-` が付いていないので、図として番号が振られません。本文の `@calibration` は相互参照ではなく「文献の引用」と解釈され、`.bib` に無いので警告が出て、本文に未解決の表示が残ります。`#| label: fig-calibration` にし、本文は `@fig-calibration` で参照します。「図1」と手で書く必要もありません（`lang: ja` で「図 1」と表示）。

    **誤り2（手書きの数値）**：本文の「0.21 L/mg」は計算結果ではありません。チャンクの中で傾きを表示してみると、

    ```python
    print(f"{slope:.4f}")
    ```

    出力:
    ```text
    0.1995
    ```

    で、本文と一致しません。数値は変数からインライン式で埋め込みます。

    修正版:
    ````markdown
    ---
    title: "検量線"
    lang: ja
    ---

    ```{python}
    #| label: fig-calibration
    #| fig-cap: "Fe(II)-フェナントロリン錯体の検量線（510 nm）"
    import numpy as np
    import matplotlib.pyplot as plt
    conc = np.array([0.0, 0.5, 1.0, 2.0, 3.0, 4.0])               # Fe 濃度 [mg/L]
    absorb = np.array([0.001, 0.099, 0.200, 0.398, 0.601, 0.797])
    slope, intercept = np.polyfit(conc, absorb, 1)
    plt.scatter(conc, absorb)
    plt.plot(conc, slope * conc + intercept)
    plt.xlabel("Fe (mg/L)"); plt.ylabel("Absorbance")
    plt.show()
    ```

    @fig-calibration に示すように、検量線の傾きは `{python} f"{slope:.4f}"` L/mg であった。
    ````
    レンダリングすると「図 1 に示すように、検量線の傾きは 0.1995 L/mg であった。」となり、データを差し替えれば数値も図番号も自動で追随します（インライン式は Quarto 1.4 以降）。

---

## この回のまとめ

- コードチャンクに `#| label: fig-xxx` / `#| fig-cap:` で図表番号を自動管理。
- 本文の `@fig-xxx` で相互参照（番号は自動追随）。
- 数式は LaTeX 記法（`$...$` / `$$...$$`）、引用は `.bib` ＋ `[@キー]`。
- 同じ文書から HTML / PDF / Word に出力できる。

### 次回予告

[第102回：再現可能な研究](lesson-102.md) では、「誰がやっても同じ結果になる」研究のための、環境管理とバージョン管理を学びます。
