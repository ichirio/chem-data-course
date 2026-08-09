# 第89回　rcdkで分子記述子を計算する

!!! abstract "この回のゴール"
    - rcdk で分子記述子（LogP・TPSA など）を計算する
    - 複数分子の記述子を data.frame にまとめる
    - 第7部の統計・作図につなげる
    - 所要時間の目安: 60分
    - 使うテーマ：**分子記述子の統計解析への橋渡し**

第60回では RDKit で記述子を計算しました。ここでは rcdk で同じことを行い、**R の統計解析（第7部）へ直接つなげます**。

`chem89.R` を作りましょう。

---

## 1. 記述子を計算する

rcdk では `eval.desc()` に「記述子の名前」を渡して計算します。

```r
library(rcdk)
mol <- parse.smiles("CC(=O)Oc1ccccc1C(=O)O")[[1]]   # アスピリン

# XLogP（脂溶性）
xlogp <- eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")
cat("XLogP:", round(as.numeric(xlogp[1, 1]), 2), "\n")

# TPSA（極性表面積）
tpsa <- eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.TPSADescriptor")
cat("TPSA:", round(as.numeric(tpsa[1, 1]), 2), "\n")
```

出力:

```text
XLogP: 1.42
TPSA: 63.6
```

記述子名が長いのが難点ですが、`eval.desc` の結果は data.frame なので `[1, 1]` で値を取り出します。

!!! note "利用できる記述子を調べる"
    `get.desc.names("topological")` などで、使える記述子の一覧を取得できます。CDK には数百の記述子があり、`get.desc.categories()` でカテゴリを見られます。

---

## 2. ヘルパー関数でまとめる

記述子名が長いので、短い関数にまとめると便利です（第6回の関数）。

```r
library(rcdk)

xlogp <- function(mol) {
  as.numeric(eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")[1, 1])
}

for (nm in list(c("aspirin", "CC(=O)Oc1ccccc1C(=O)O"),
                c("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"),
                c("ethanol", "CCO"))) {
  mol <- parse.smiles(nm[2])[[1]]
  cat(nm[1], ": XLogP =", round(xlogp(mol), 2), "\n")
}
```

出力:

```text
aspirin : XLogP = 1.42 
caffeine : XLogP = -0.5 
ethanol : XLogP = -0.08 
```

アスピリンは脂溶性寄り、カフェイン・エタノールは水寄り。RDKit（第60回）とは計算法が違うため値は少しずれますが、**傾向は同じ**です。

---

## 3. 記述子を data.frame にまとめて統計へ

計算した記述子を data.frame にすれば、第7部の統計・ggplot2 がそのまま使えます。

```r
library(rcdk)
library(tidyverse)

xlogp <- function(mol) as.numeric(eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")[1, 1])

smiles <- c("CCO", "CC(=O)Oc1ccccc1C(=O)O", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C")
names  <- c("ethanol", "aspirin", "caffeine")
mols <- parse.smiles(smiles)

df <- tibble(
  name = names,
  MW = sapply(mols, get.natural.mass),
  XLogP = sapply(mols, xlogp)
)
print(df)
```

`sapply(mols, 関数)` は「リストの各要素に関数を適用」する R の書き方（Python の map に相当）。これで**分子記述子の表**ができ、`summary(df)` や `ggplot(df, ...)` で解析・作図できます。

!!! success "分子 → 統計、R で一気通貫"
    「rcdk で記述子を計算 → tibble にまとめ → 第7部の統計・回帰・作図」。第6部（Python/RDKit）でやったことが、R だけでも実現できます。あなたが R 中心なら、この流れが自然です。

---

## 演習問題

**問1.** rcdk で、イブプロフェン `CC(C)Cc1ccc(cc1)C(C)C(=O)O` の XLogP と TPSA を計算してください。

**問2.** ベンゼン `c1ccccc1`、トルエン `Cc1ccccc1`、オクタン `CCCCCCCC` の XLogP を、ヘルパー関数とループでまとめて表示してください（炭化水素なので LogP は正の大きめの値になるはず）。

**問3.** 問2の3分子について、name・MW・XLogP を持つ tibble を作り、`arrange` で XLogP の大きい順に並べてください。

---

## 解答

??? success "問1 の解答"
    ```r
    library(rcdk)
    mol <- parse.smiles("CC(C)Cc1ccc(cc1)C(C)C(=O)O")[[1]]
    xl <- eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")
    tp <- eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.TPSADescriptor")
    cat("XLogP:", round(as.numeric(xl[1,1]), 2), " TPSA:", round(as.numeric(tp[1,1]), 2), "\n")
    ```

    出力:
    ```text
    XLogP: 3.64  TPSA: 37.3 
    ```

??? success "問2 の解答"
    ```r
    library(rcdk)
    xlogp <- function(mol) as.numeric(eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")[1,1])
    for (nm in list(c("benzene","c1ccccc1"), c("toluene","Cc1ccccc1"), c("octane","CCCCCCCC"))) {
      mol <- parse.smiles(nm[2])[[1]]
      cat(nm[1], ":", round(xlogp(mol), 2), "\n")
    }
    ```

    出力:
    ```text
    benzene : 2.02 
    toluene : 2.46 
    octane : 4.89 
    ```

??? success "問3 の解答"
    ```r
    library(rcdk); library(tidyverse)
    xlogp <- function(mol) as.numeric(eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")[1,1])
    smiles <- c("c1ccccc1","Cc1ccccc1","CCCCCCCC"); nm <- c("benzene","toluene","octane")
    mols <- parse.smiles(smiles)
    tibble(name=nm, MW=sapply(mols, get.natural.mass), XLogP=sapply(mols, xlogp)) %>%
      arrange(desc(XLogP))
    ```
    octane（最も脂溶性が高い）が先頭に来ます。

---

## この回のまとめ

- rcdk の `eval.desc(mol, "記述子名")` で記述子を計算（結果は data.frame）。
- 記述子名は長いので**ヘルパー関数**にまとめると便利。
- `sapply(mols, 関数)` で複数分子に一括適用（map 相当）。
- 記述子を tibble にまとめれば、第7部の統計・作図が使える。

### 次回予告

[第90回：まとめ演習（rcdkで化合物データの統計解析）](lesson-90.md) では、第7部の総仕上げとして、分子の類似度計算と統計解析を R で行います。
