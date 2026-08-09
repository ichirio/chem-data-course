# 第97回　特徴量エンジニアリング（分子記述子を特徴に）

!!! abstract "この回のゴール"
    - RDKit の分子記述子を、機械学習の**特徴量**にする
    - SMILES から特徴量の表（DataFrame）を作る
    - 第6部（分子）と第8部（機械学習）を合流させる
    - 所要時間の目安: 60分
    - 使うテーマ：**分子から機械学習用の特徴を作る**

機械学習の性能は、**どんな特徴量を与えるか**で大きく変わります。分子の場合、RDKit の記述子（第60回）が強力な特徴量になります。

`ml97.py` を作りましょう。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors
import pandas as pd
```

---

## 1. 分子から記述子の表を作る

SMILES のリストから、複数の記述子を計算して DataFrame にします（第63回のパターン）。この表が機械学習の入力になります。

```python
mols = {
    "methanol": "CO", "ethanol": "CCO", "benzene": "c1ccccc1", "toluene": "Cc1ccccc1",
    "octane": "CCCCCCCC", "aspirin": "CC(=O)Oc1ccccc1C(=O)O",
    "caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "glucose": "OCC1OC(O)C(O)C(O)C1O",
    "hexane": "CCCCCC", "naphthalene": "c1ccc2ccccc2c1",
    "ibuprofen": "CC(C)Cc1ccc(cc1)C(C)C(=O)O", "acetic_acid": "CC(=O)O",
}

rows = []
for name, smi in mols.items():
    m = Chem.MolFromSmiles(smi)
    rows.append({
        "name": name,
        "MW":   round(Descriptors.MolWt(m), 1),
        "LogP": round(Descriptors.MolLogP(m), 2),
        "TPSA": round(Descriptors.TPSA(m), 1),
        "HBD":  rdMolDescriptors.CalcNumHBD(m),
        "HBA":  rdMolDescriptors.CalcNumHBA(m),
        "RotB": rdMolDescriptors.CalcNumRotatableBonds(m),
    })

df = pd.DataFrame(rows)
print(df.head(6).to_string(index=False))
```

出力:

```text
    name    MW  LogP  TPSA  HBD  HBA  RotB
methanol  32.0 -0.39  20.2    1    1     0
 ethanol  46.1 -0.00  20.2    1    1     0
 benzene  78.1  1.69   0.0    0    0     0
 toluene  92.1  2.00   0.0    0    0     0
  octane 114.2  3.37   0.0    0    0     5
 aspirin 180.2  1.31  63.6    1    3     2
```

6種類の記述子が特徴量になりました。この `df[["MW","LogP","TPSA","HBD","HBA","RotB"]]` を、機械学習モデルの `X` として使えます。

---

## 2. 良い特徴量とは

!!! note "特徴量選びのポイント"
    - **目的に関係する記述子を選ぶ**：溶解性を予測するなら、極性に関わる TPSA・LogP・HBD/HBA が効きそう。
    - **多すぎる特徴量は過学習の元**（第95回）：関係の薄いものは入れない。
    - **スケールをそろえる**：MW（数百）と HBD（0〜数個）ではスケールが違いすぎる。次回・第98回の前に**標準化**（第43回）が有効。

RDKit には200種類以上の記述子があります。`Descriptors.descList` で一覧を取得できますが、まずは意味の分かる基本的なものから始めるのがおすすめです。

---

## 3. 標準化して機械学習の準備

スケールをそろえる標準化（第43回）を、scikit-learn の `StandardScaler` で行います。

```python
from sklearn.preprocessing import StandardScaler

features = ["MW", "LogP", "TPSA", "HBD", "HBA", "RotB"]
X = df[features]

X_scaled = StandardScaler().fit_transform(X)
print("標準化後の平均（各列ほぼ0）:", X_scaled.mean(axis=0).round(2))
```

出力:

```text
標準化後の平均（各列ほぼ0）: [ 0.  0. -0.  0.  0.  0.]
```

各特徴量の平均が0・標準偏差1にそろい、機械学習（特に PCA やクラスタリング）の準備が整いました。

!!! success "分子 → 特徴量 → 機械学習"
    **SMILES → RDKit で記述子計算 → DataFrame → 標準化 → 機械学習**。
    第6部で学んだ分子処理が、機械学習の入力作りに直結しました。QSAR（構造-活性相関）は、まさにこの流れで分子から活性を予測します（第100回）。

---

## 演習問題

**問1.** 本文の `mols` から記述子の DataFrame を作り、全12分子を表示してください。

**問2.** 作った DataFrame で、`MW` と `LogP` の相関を `df["MW"].corr(df["LogP"])` で調べてください（第52回）。特徴量どうしの関係を見るのは、機械学習の前の大事な確認です。

**問3.** `StandardScaler` で `["MW", "LogP", "TPSA"]` の3列を標準化し、標準化後の各列の標準偏差が（ほぼ）1になることを確認してください。

---

## 解答

??? success "問1 の解答"
    ```python
    # 本文のループで df を作ってから
    print(df.to_string(index=False))
    ```
    全12分子の記述子表が表示されます（methanol〜acetic_acid）。

??? success "問2 の解答"
    ```python
    print(round(df["MW"].corr(df["LogP"]), 3))
    ```
    MW と LogP の相関係数が表示されます。強く相関する特徴量どうしは、片方だけで足りることもあります（多重共線性、第84回）。

??? success "問3 の解答"
    ```python
    from sklearn.preprocessing import StandardScaler
    Xs = StandardScaler().fit_transform(df[["MW", "LogP", "TPSA"]])
    print(Xs.std(axis=0).round(3))     # [1. 1. 1.]
    ```

    出力:
    ```text
    [1. 1. 1.]
    ```

---

## この回のまとめ

- RDKit の分子記述子を、機械学習の**特徴量**にする（SMILES → 記述子 → DataFrame）。
- 目的に関係する記述子を選ぶ。多すぎは過学習の元。
- `StandardScaler` でスケールをそろえる（PCA・クラスタリングの前に有効）。
- 「分子 → 特徴量 → 機械学習」が QSAR の基本の流れ。

### 次回予告

[第98回：次元削減と可視化（PCA）](lesson-98.md) では、たくさんの記述子を2次元に圧縮して、化合物の分布を可視化します。
