# 第88回　Rでケモインフォマティクス入門（rcdk）

!!! abstract "この回のゴール"
    - R でも分子（構造式）を扱えることを知る
    - **rcdk** で SMILES から分子を作る
    - 分子式・分子量を計算する
    - 所要時間の目安: 60分
    - 使うテーマ：**R での分子ハンドリング**

第6部では Python の RDKit で分子を扱いました。実は **R でも同じことができます**。**rcdk**（R interface to the Chemistry Development Kit）を使えば、統計解析（第7部）と分子処理を、R だけで完結できます。

!!! info "準備：rcdk（Java が必要）"
    rcdk は Java 上で動く CDK ライブラリを R から使います。**Java の導入**が前提です。
    ```r
    install.packages("rcdk")     # rJava, Java が必要
    ```
    Java が未導入なら、先に Java（OpenJDK など）を入れてください。うまく動かないときは、Python の RDKit（第6部）を使う選択もあります。

`chem88.R` を作りましょう。

---

## 1. SMILES から分子を作る

`parse.smiles()` で、SMILES 文字列から分子オブジェクトを作ります（RDKit の `MolFromSmiles` に相当）。

```r
library(rcdk)

# SMILES から分子を作る（結果はリスト。[[1]] で取り出す）
mol <- parse.smiles("CC(=O)Oc1ccccc1C(=O)O")[[1]]   # アスピリン

print(mol)
```

`parse.smiles` は複数の SMILES をまとめて処理できるよう、**リスト**を返します。1つなら `[[1]]` で取り出します。

---

## 2. 分子式・分子量を計算する

```r
library(rcdk)
mol <- parse.smiles("CC(=O)Oc1ccccc1C(=O)O")[[1]]

# 分子式
formula <- get.mol2formula(mol)
cat("分子式:", formula@string, "\n")

# 分子量（平均）と厳密質量
cat("分子量:", round(get.natural.mass(mol), 2), "\n")
cat("厳密質量:", round(get.exact.mass(mol), 4), "\n")
```

出力:

```text
分子式: C9H8O4
分子量: 180.16
厳密質量: 180.0423
```

RDKit（第59回）と同じ値が、R でも得られました。

- `get.mol2formula(mol)@string` … 分子式（`@` はR のS4オブジェクトの要素アクセス）
- `get.natural.mass(mol)` … 平均分子量（180.16）
- `get.exact.mass(mol)` … 厳密質量（180.0423、質量分析用）

!!! note "`@` 記号について"
    rcdk の一部は R の **S4 オブジェクト**という形式で、要素に `@`（アットマーク）でアクセスします。`formula@string` は「formula の string 要素」。R 独特の書き方です。

---

## 3. 複数の分子をまとめて処理

`parse.smiles` は複数 SMILES をリストで受け取れます。ループで分子式を求めましょう。

```r
library(rcdk)

smiles <- c("CCO", "c1ccccc1", "CC(=O)Oc1ccccc1C(=O)O")
names  <- c("ethanol", "benzene", "aspirin")
mols <- parse.smiles(smiles)

for (i in seq_along(mols)) {
  f <- get.mol2formula(mols[[i]])@string
  mw <- round(get.natural.mass(mols[[i]]), 2)
  cat(names[i], ":", f, ", MW =", mw, "\n")
}
```

出力:

```text
ethanol : C2H6O , MW = 46.07 
benzene : C6H6 , MW = 78.11 
aspirin : C9H8O4 , MW = 180.16 
```

!!! success "R だけで統計＋分子処理"
    rcdk のおかげで、**分子の性質を計算 → data.frame にまとめ → 第7部の統計・ggplot2 で解析**、という流れを R だけで完結できます。R が得意な人は、Python に移らずとも化学データ分析ができます。

---

## 演習問題

**問1.** rcdk で、カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` の分子式・分子量・厳密質量を表示してください。

**問2.** メタノール `CO`、酢酸 `CC(=O)O`、グルコース `OCC1OC(O)C(O)C(O)C1O` の分子式と分子量を、ループでまとめて表示してください。

**問3.** `parse.smiles` に**無効な SMILES**（例：`"XYZ"`）を渡すとどうなるか確かめ、結果が `NULL` かどうかを `is.null()` で判定してください。

---

## 解答

??? success "問1 の解答"
    ```r
    library(rcdk)
    mol <- parse.smiles("CN1C=NC2=C1C(=O)N(C(=O)N2C)C")[[1]]
    cat("分子式:", get.mol2formula(mol)@string, "\n")
    cat("分子量:", round(get.natural.mass(mol), 2), "\n")
    cat("厳密質量:", round(get.exact.mass(mol), 4), "\n")
    ```

    出力:
    ```text
    分子式: C8H10N4O2
    分子量: 194.19
    厳密質量: 194.0804
    ```

??? success "問2 の解答"
    ```r
    library(rcdk)
    smiles <- c("CO", "CC(=O)O", "OCC1OC(O)C(O)C(O)C1O")
    names  <- c("methanol", "acetic acid", "glucose")
    mols <- parse.smiles(smiles)
    for (i in seq_along(mols)) {
      cat(names[i], ":", get.mol2formula(mols[[i]])@string,
          ", MW =", round(get.natural.mass(mols[[i]]), 2), "\n")
    }
    ```

    出力:
    ```text
    methanol : CH4O , MW = 32.04 
    acetic acid : C2H4O2 , MW = 60.05 
    glucose : C6H12O6 , MW = 180.16 
    ```

??? success "問3 の解答"
    ```r
    library(rcdk)
    mol <- parse.smiles("XYZ")[[1]]
    is.null(mol)      # [1] TRUE
    ```
    無効な SMILES では `NULL` が返ります。RDKit の `None` チェック（第56回）と同じ発想で、`is.null()` で確認できます。

---

## この回のまとめ

- **rcdk** で R でも分子を扱える（Java が必要）。
- `parse.smiles("SMILES")[[1]]` で分子を作る。
- `get.mol2formula(mol)@string`（分子式）、`get.natural.mass`（分子量）、`get.exact.mass`（厳密質量）。
- 複数 SMILES はリストで処理。無効な SMILES は `NULL`。

### 次回予告

[第89回：rcdkで分子記述子を計算する](lesson-89.md) では、LogP などの記述子を R で求め、統計解析につなげます。
