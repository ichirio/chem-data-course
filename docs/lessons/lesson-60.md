# 第60回　分子記述子（LogP・極性表面積など）

!!! abstract "この回のゴール"
    - **分子記述子**（構造から計算する性質の数値）を知る
    - LogP・TPSA・水素結合ドナー/アクセプターを求める
    - 「リピンスキーのルール」で薬らしさを判定する
    - 所要時間の目安: 60分
    - 使うテーマ：**医薬分子の性質評価**

**分子記述子**は、分子の構造から計算される「性質を表す数値」です。溶けやすさ・膜透過性・反応性などの手がかりになり、創薬や材料設計で広く使われます。

`lesson60.py` を作りましょう。

---

## 1. 代表的な記述子

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

mol = Chem.MolFromSmiles("CC(=O)Oc1ccccc1C(=O)O")   # アスピリン

print("LogP:", round(Descriptors.MolLogP(mol), 2))
print("TPSA:", round(Descriptors.TPSA(mol), 2))
print("水素結合ドナー(HBD):", rdMolDescriptors.CalcNumHBD(mol))
print("水素結合アクセプター(HBA):", rdMolDescriptors.CalcNumHBA(mol))
print("回転可能結合:", rdMolDescriptors.CalcNumRotatableBonds(mol))
```

出力:

```text
LogP: 1.31
TPSA: 63.6
水素結合ドナー(HBD): 1
水素結合アクセプター(HBA): 3
回転可能結合: 2
```

| 記述子 | 意味 | 何が分かる |
|---|---|---|
| **LogP** | 油/水への分配のしやすさ | 大きいほど脂溶性（油になじむ） |
| **TPSA** | 極性表面積 | 大きいほど極性・膜透過しにくい |
| **HBD / HBA** | 水素結合の授受の数 | 溶解性・相互作用の目安 |
| **回転可能結合** | 分子の柔らかさ | 多いほど柔軟 |

---

## 2. 分子を比べる

3つの医薬分子で記述子を比べてみます。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

molecules = {
    "aspirin":   "CC(=O)Oc1ccccc1C(=O)O",
    "caffeine":  "CN1C=NC2=C1C(=O)N(C(=O)N2C)C",
    "ibuprofen": "CC(C)Cc1ccc(cc1)C(C)C(=O)O",
}

for name, smi in molecules.items():
    mol = Chem.MolFromSmiles(smi)
    logp = round(Descriptors.MolLogP(mol), 2)
    tpsa = round(Descriptors.TPSA(mol), 2)
    print(f"{name}: LogP={logp}, TPSA={tpsa}")
```

出力:

```text
aspirin: LogP=1.31, TPSA=63.6
caffeine: LogP=-1.03, TPSA=61.82
ibuprofen: LogP=3.07, TPSA=37.3
```

イブプロフェンは LogP が高く（脂溶性が高い）、カフェインは LogP が負（水になじむ）。数値で性質の違いが読み取れます。

---

## 3. リピンスキーのルール（薬らしさ）

**リピンスキーの「ルール・オブ・ファイブ」**は、経口薬になりやすい分子の目安です。次の4条件のうち**違反が2つ以上あると、経口吸収されにくい**とされます。

- 分子量 ≤ 500
- LogP ≤ 5
- 水素結合ドナー ≤ 5
- 水素結合アクセプター ≤ 10

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

def lipinski_violations(mol):
    """リピンスキー則の違反数を返す"""
    violations = 0
    if Descriptors.MolWt(mol) > 500: violations += 1
    if Descriptors.MolLogP(mol) > 5: violations += 1
    if rdMolDescriptors.CalcNumHBD(mol) > 5: violations += 1
    if rdMolDescriptors.CalcNumHBA(mol) > 10: violations += 1
    return violations

for name, smi in [("aspirin", "CC(=O)Oc1ccccc1C(=O)O"),
                  ("ibuprofen", "CC(C)Cc1ccc(cc1)C(C)C(=O)O")]:
    mol = Chem.MolFromSmiles(smi)
    print(f"{name}: 違反 {lipinski_violations(mol)} 個")
```

出力:

```text
aspirin: 違反 0 個
ibuprofen: 違反 0 個
```

どちらも違反0＝経口薬らしい、と判定できました。

!!! success "記述子は全分野で使える"
    LogP は環境化学（生物濃縮）、TPSA は材料の表面設計、といったように、記述子は創薬以外でも「構造から性質を推定する」道具として使われます。第8部の機械学習では、これらの記述子を**特徴量**にして物性を予測します。

---

## 演習問題

**問1.** パラセタモール `CC(=O)Nc1ccc(O)cc1` の LogP・TPSA・HBD・HBA を表示してください。

**問2.** カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` とグルコース `OCC1OC(O)C(O)C(O)C1O` の LogP を比べてください。どちらが水になじみやすい（LogP が小さい）ですか？

**問3.** 本文の `lipinski_violations` 関数を使って、パラセタモールとカフェインのリピンスキー違反数を表示してください。

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors
    mol = Chem.MolFromSmiles("CC(=O)Nc1ccc(O)cc1")
    print("LogP:", round(Descriptors.MolLogP(mol), 2))
    print("TPSA:", round(Descriptors.TPSA(mol), 2))
    print("HBD:", rdMolDescriptors.CalcNumHBD(mol))
    print("HBA:", rdMolDescriptors.CalcNumHBA(mol))
    ```

    出力:
    ```text
    LogP: 1.35
    TPSA: 49.33
    HBD: 2
    HBA: 2
    ```

??? success "問2 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors
    for name, smi in [("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"),
                      ("glucose", "OCC1OC(O)C(O)C(O)C1O")]:
        print(name, round(Descriptors.MolLogP(Chem.MolFromSmiles(smi)), 2))
    ```

    出力:
    ```text
    caffeine -1.03
    glucose -3.22
    ```
    グルコースの方が LogP が小さく、より水になじみやすい（親水性が高い）。糖なので納得です。

??? success "問3 の解答"
    ```python
    # 本文の lipinski_violations を定義したうえで
    for name, smi in [("paracetamol", "CC(=O)Nc1ccc(O)cc1"),
                      ("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C")]:
        mol = Chem.MolFromSmiles(smi)
        print(name, lipinski_violations(mol))
    ```

    出力:
    ```text
    paracetamol 0
    caffeine 0
    ```

---

## この回のまとめ

- 分子記述子＝構造から計算する性質の数値（LogP, TPSA, HBD, HBA…）。
- `Descriptors.MolLogP` / `Descriptors.TPSA` / `rdMolDescriptors.CalcNumHBD` など。
- リピンスキー則で「経口薬らしさ」を判定（違反2つ以上で吸収されにくい）。
- 記述子は創薬以外（環境・材料）でも、性質推定・機械学習の特徴量に使う。

### 次回予告

[第61回：部分構造検索とサブストラクチャ](lesson-61.md) では、「この分子にカルボキシ基はあるか？」といった部分構造の検索を学びます。
