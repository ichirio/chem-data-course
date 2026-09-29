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
| **LogP** | 1-オクタノール/水の分配係数の対数。RDKit の `MolLogP` は Wildman–Crippen 法（原子ごとの寄与の和）による**推定値** | 大きいほど脂溶性（油になじむ） |
| **TPSA** | トポロジカル極性表面積 [Å²]。N・O 原子（と結合した H）の寄与の和（Ertl 法。RDKit の既定では S・P を含まない） | 大きいほど極性が高く、膜を透過しにくい（経口薬ではおよそ 140 Å² 以下が目安） |
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

!!! note "計算値と実測値は一致しない"
    `MolLogP` は構造から推定した値です。実測の logP はアスピリン 1.19、カフェイン −0.07、イブプロフェン 3.97 で、計算値（1.31、−1.03、3.07）と最大で約1ずれます。大小の傾向を見る目安として使い、実測値が必要な場面では文献値を使います。

---

## 3. リピンスキーのルール（薬らしさ）

**リピンスキーの「ルール・オブ・ファイブ」**は、経口薬になりやすい分子の目安です。次の4条件のうち**違反が2つ以上あると、経口吸収されにくい**とされます。

- 分子量 ≤ 500
- LogP ≤ 5
- 水素結合ドナー ≤ 5
- 水素結合アクセプター ≤ 10

!!! note "リピンスキー則の HBD / HBA は数え方が決まっている"
    元の論文では、ドナー＝NH と OH の水素の数、アクセプター＝N と O の原子数の合計、と定義されています。RDKit では `CalcNumLipinskiHBD` / `CalcNumLipinskiHBA` がこの数え方です。1節の `CalcNumHBD` / `CalcNumHBA` は別の定義なので、値が違うことがあります（アスピリンの HBA は 3 と 4）。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

def lipinski_violations(mol):
    """リピンスキー則の違反数を返す"""
    violations = 0
    if Descriptors.MolWt(mol) > 500: violations += 1
    if Descriptors.MolLogP(mol) > 5: violations += 1
    if rdMolDescriptors.CalcNumLipinskiHBD(mol) > 5: violations += 1   # NH・OH の H の数
    if rdMolDescriptors.CalcNumLipinskiHBA(mol) > 10: violations += 1  # N と O の原子数
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

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「RDKit で、薬の名前と SMILES の辞書から MW・LogP（`Descriptors.MolLogP`）・TPSA・HBD・HBA とリピンスキー違反数を計算する関数を書いてください。
    HBD/HBA は Lipinski の定義（`rdMolDescriptors.CalcNumLipinskiHBD` / `CalcNumLipinskiHBA`）で数え、
    条件は MW ≤ 500、LogP ≤ 5、HBD ≤ 5、HBA ≤ 10、違反が1つまでなら『経口薬らしい』と判定してください。
    LogP は計算値であることをコメントに明記してください。」

!!! warning "AIの出力で確かめること"
    - **しきい値の取り違え**：HBD ≤ 5、HBA ≤ 10 が逆になっていないか、`>` と `>=` が条件どおりか、コードの4行を1行ずつ読みます。
    - **判定の基準**：Ro5 は「違反が2つ以上で吸収されにくい」。`return v == 0` で合否を決めていないか。LogP 6.0 のタモキシフェン（違反1、実際に経口薬）が「不合格」になったら誤りです。
    - HBD/HBA がどの関数か（`CalcNumHBA` と `CalcNumLipinskiHBA` では値が違う）。アスピリンで HBA が 3 と出たら前者、4 なら後者です。
    - LogP を「実測値」と説明していないか。`MolLogP` は Crippen 法の推定値で、カフェインでは計算 −1.03、実測 −0.07 と1近くずれます。
    - TPSA に単位 Å² が付いているか。

---

## 演習問題

**問1.** パラセタモール `CC(=O)Nc1ccc(O)cc1` の LogP・TPSA・HBD・HBA を表示してください。

**問2.** カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` とグルコース `OC[C@H]1OC(O)[C@H](O)[C@@H](O)[C@@H]1O` の LogP を比べてください。どちらが水になじみやすい（LogP が小さい）ですか？

**問3.** 本文の `lipinski_violations` 関数を使って、パラセタモールとカフェインのリピンスキー違反数を表示してください。

**問4.** 次は AI が書いた「リピンスキー則で経口薬らしいかを判定する」コードです。タモキシフェン（乳がんの経口薬）とアシクロビル（抗ウイルスの経口薬）がどちらも「不合格」になりました。問題点を指摘し、直してください。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

def is_druglike(mol):
    """リピンスキー則で経口薬らしいかを判定する"""
    v = 0
    if Descriptors.MolWt(mol) > 500: v += 1
    if Descriptors.MolLogP(mol) > 5: v += 1
    if rdMolDescriptors.CalcNumHBD(mol) > 10: v += 1
    if rdMolDescriptors.CalcNumHBA(mol) > 5: v += 1
    return v == 0

for name, smi in [("tamoxifen", r"CC/C(=C(\c1ccccc1)c1ccc(OCCN(C)C)cc1)c1ccccc1"),
                  ("aciclovir", "Nc1nc2n(COCCO)cnc2c(=O)[nH]1")]:
    print(name, is_druglike(Chem.MolFromSmiles(smi)))
```

AI の出力:

```text
tamoxifen False
aciclovir False
```

（タモキシフェンの SMILES には `\` が含まれるので、Python では `r"..."`（raw 文字列）で書いています。）

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
                      ("glucose", "OC[C@H]1OC(O)[C@H](O)[C@@H](O)[C@@H]1O")]:
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

??? success "問4 の解答"
    誤りは2つです。

    1. **HBD と HBA のしきい値が逆**：正しくは HBD ≤ 5、HBA ≤ 10。アシクロビルは HBA が 6 なので、誤った「HBA > 5」で違反にされていました。HBD/HBA は Lipinski の定義（`CalcNumLipinskiHBD` / `CalcNumLipinskiHBA`）で数えます。
    2. **合否の基準が厳しすぎる**：Ro5 は「違反が2つ以上で吸収されにくい」なので、違反1つまでは合格です。タモキシフェンは LogP が 6.0 で違反1つですが、実際に経口薬です。

    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors

    def lipinski_violations(mol):
        """リピンスキー則の違反数を返す"""
        v = 0
        if Descriptors.MolWt(mol) > 500: v += 1
        if Descriptors.MolLogP(mol) > 5: v += 1
        if rdMolDescriptors.CalcNumLipinskiHBD(mol) > 5: v += 1
        if rdMolDescriptors.CalcNumLipinskiHBA(mol) > 10: v += 1
        return v

    for name, smi in [("tamoxifen", r"CC/C(=C(\c1ccccc1)c1ccc(OCCN(C)C)cc1)c1ccccc1"),
                      ("aciclovir", "Nc1nc2n(COCCO)cnc2c(=O)[nH]1")]:
        mol = Chem.MolFromSmiles(smi)
        v = lipinski_violations(mol)
        print(f"{name}: LogP={Descriptors.MolLogP(mol):.2f}, 違反 {v} 個 → 経口薬らしい: {v <= 1}")
    ```

    出力:
    ```text
    tamoxifen: LogP=6.00, 違反 1 個 → 経口薬らしい: True
    aciclovir: LogP=-1.33, 違反 0 個 → 経口薬らしい: True
    ```

---

## この回のまとめ

- 分子記述子＝構造から計算する性質の数値（LogP, TPSA, HBD, HBA…）。
- `Descriptors.MolLogP` / `Descriptors.TPSA` / `rdMolDescriptors.CalcNumHBD` など。
- リピンスキー則で「経口薬らしさ」を判定（違反2つ以上で吸収されにくい）。HBD/HBA は `CalcNumLipinskiHBD` / `CalcNumLipinskiHBA` で数える。
- 記述子は創薬以外（環境・材料）でも、性質推定・機械学習の特徴量に使う。

### 次回予告

[第61回：部分構造検索とサブストラクチャ](lesson-61.md) では、「この分子にカルボキシ基はあるか？」といった部分構造の検索を学びます。
