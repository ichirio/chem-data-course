# 第90回　まとめ演習：Rで化合物データの統計解析

!!! abstract "この回のゴール"
    - rcdk（分子）と tidyverse（統計）を R だけで統合する
    - フィンガープリントで分子類似度を計算する
    - 記述子の表を作り、統計解析・作図する
    - 所要時間の目安: 60分（第7部の総仕上げ）
    - 使うテーマ：**R で完結する化合物データ分析**

第7部の集大成として、**分子の処理から統計解析まで、すべて R で**行います。第6部（Python）と同じことが R でもできる、という締めくくりです。

`chem90.R` を作りましょう。

---

## 1. フィンガープリントで分子類似度

rcdk でも、分子の類似度（谷本係数）を計算できます（第62回の R 版）。

```r
library(rcdk)

smiles <- c("CC(=O)Oc1ccccc1C(=O)O",   # aspirin
            "OC(=O)c1ccccc1O",           # salicylic acid
            "CC(C)Cc1ccc(cc1)C(C)C(=O)O") # ibuprofen
mols <- parse.smiles(smiles)
fps <- lapply(mols, get.fingerprint, type = "circular")

# 谷本係数（fingerprint パッケージの distance を明示）
cat("aspirin vs salicylic:", round(fingerprint::distance(fps[[1]], fps[[2]], method = "tanimoto"), 3), "\n")
cat("aspirin vs ibuprofen:", round(fingerprint::distance(fps[[1]], fps[[3]], method = "tanimoto"), 3), "\n")
```

出力:

```text
aspirin vs salicylic: 0.41
aspirin vs ibuprofen: 0.164
```

アスピリンとサリチル酸（構造が近い）は類似度が高く、イブプロフェン（別物）は低い。RDKit（第62回）と同じ傾向です。

!!! note "`fingerprint::distance`"
    類似度計算は `fingerprint` パッケージの `distance()` を使います。他のパッケージにも同名関数があるため、`fingerprint::` を付けて明示すると安全です（名前の衝突回避）。

---

## 2. 記述子の表を作り、統計解析する

複数分子の記述子を tibble にまとめ、第7部の統計・作図につなげます。

```r
library(rcdk)
library(tidyverse)

xlogp <- function(mol) as.numeric(eval.desc(mol, "org.openscience.cdk.qsar.descriptors.molecular.XLogPDescriptor")[1, 1])

drugs <- c("CC(=O)Oc1ccccc1C(=O)O", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C",
           "CC(C)Cc1ccc(cc1)C(C)C(=O)O", "CC(=O)Nc1ccc(O)cc1")
names <- c("aspirin", "caffeine", "ibuprofen", "paracetamol")
mols <- parse.smiles(drugs)

df <- tibble(
  name  = names,
  MW    = round(sapply(mols, get.natural.mass), 2),
  XLogP = round(sapply(mols, xlogp), 2)
)
print(df)

# MW と XLogP の相関（第83回）
cat("MW-XLogP 相関:", round(cor(df$MW, df$XLogP), 3), "\n")
```

出力:

```text
# A tibble: 4 × 3
  name           MW XLogP
  <chr>       <dbl> <dbl>
1 aspirin      180.  1.42
2 caffeine     194. -0.5 
3 ibuprofen    206.  3.64
4 paracetamol  151.  0.87
```

分子の記述子表が R だけででき、`cor()` で相関、`ggplot()` で作図——第6部（分子）と第7部（統計）が、R の中で1つになりました。

---

## 3. 作図してまとめる

```r
library(tidyverse)
ggplot(df, aes(x = MW, y = XLogP, label = name)) +
  geom_point(size = 4, color = "steelblue") +
  geom_text(vjust = -1) +
  labs(x = "Molecular Weight", y = "XLogP", title = "Drug descriptors (rcdk)") +
  theme_minimal()
```

分子量と脂溶性の関係を散布図にできます。ここに第7部の回帰直線（`geom_smooth`）を重ねれば、立派な解析レポートです。

!!! success "第7部の集大成"
    **rcdk で分子を処理 → tibble にまとめ → tidyverse で統計・作図 → Quarto で帳票**。統計処理も構造式ハンドリングも、**R だけで一気通貫**できました。Python（第6部）と R（第7部）、どちらのルートでも化学データ分析ができる——これがこのコースの到達点の一つです。

---

## 演習問題

**問1.** rcdk で、カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` とテオフィリン `Cn1c(=O)c2[nH]cnc2n(C)c1=O`（似た構造）の類似度を計算してください。高い類似度になるはずです。

**問2.** 本文の4つの薬の tibble を作り、MW の**平均**と XLogP の**平均**を `summarise` で求めてください。

**問3.** 本文の散布図（MW vs XLogP）を作り、各点に分子名のラベル（`geom_text`）を付けてください。

---

## 解答

??? success "問1 の解答"
    ```r
    library(rcdk)
    mols <- parse.smiles(c("CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "Cn1c(=O)c2[nH]cnc2n(C)c1=O"))
    fps <- lapply(mols, get.fingerprint, type = "circular")
    round(fingerprint::distance(fps[[1]], fps[[2]], method = "tanimoto"), 3)
    ```
    カフェインとテオフィリン（メチル基1つ違い）は構造がよく似ているため、高い類似度になります。

??? success "問2 の解答"
    ```r
    library(tidyverse)
    df %>% summarise(mean_MW = round(mean(MW), 1), mean_XLogP = round(mean(XLogP), 2))
    ```
    4つの薬の MW と XLogP の平均が求まります。

??? success "問3 の解答"
    ```r
    library(tidyverse)
    ggplot(df, aes(x = MW, y = XLogP, label = name)) +
      geom_point(size = 4, color = "steelblue") +
      geom_text(vjust = -1) +
      theme_minimal()
    ```

---

## 第7部　修了

おめでとうございます！ R で、**データ整形（tidyverse）・記述統計・仮説検定（t検定・ANOVA・カイ二乗）・回帰・帳票（Quarto）**、そして **分子のハンドリング（rcdk）**まで身につきました。統計処理も構造式も、R だけで完結できます。

!!! tip "ここまでの到達点"
    第1〜7部で、Python と R の両方で、化学データを **処理・可視化・統計解析・分子処理・レポート化**できるようになりました。実験・研究のデータ分析に必要な力は、ほぼ揃っています。残るは機械学習（第8部）と研究への総合応用（第9部）です。

### 次回予告

このあとは、**第8部：機械学習**を追加していきます。物性予測など、データから パターンを学ばせる手法を扱います。ここまで本当によく頑張りました！
