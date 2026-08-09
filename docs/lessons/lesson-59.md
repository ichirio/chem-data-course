# 第59回　分子量・分子式・元素組成を計算する

!!! abstract "この回のゴール"
    - 分子から**分子式**を自動で求める
    - **分子量**と**厳密質量（モノアイソトピック質量）**を計算する
    - 元素ごとの原子数を数える
    - 所要時間の目安: 60分
    - 使うテーマ：**多分野の分子**の基本物性

第2回・第3回では、原子量表と組成から分子量を手計算しました。RDKit なら、SMILES から**一発**で求まります。

`lesson59.py` を作りましょう。

---

## 1. 分子式と分子量

`Descriptors` と `rdMolDescriptors` を使います。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

mol = Chem.MolFromSmiles("CC(=O)Oc1ccccc1C(=O)O")   # アスピリン

print("分子式:", rdMolDescriptors.CalcMolFormula(mol))
print("分子量:", round(Descriptors.MolWt(mol), 2))
print("厳密質量:", round(Descriptors.ExactMolWt(mol), 4))
```

出力:

```text
分子式: C9H8O4
分子量: 180.16
厳密質量: 180.0423
```

- **分子式** `C9H8O4` … 水素も含めて自動でカウント（SMILES では省略した H も数える）。
- **分子量** 180.16 … 平均原子量による分子量（普通の計算）。
- **厳密質量** 180.0423 … 最も多い同位体だけで計算した質量（質量分析で使う）。

!!! note "分子量と厳密質量の違い"
    - **分子量（MolWt）**… 各元素の**平均原子量**を使う。化学量論の計算に。
    - **厳密質量（ExactMolWt）**… 各元素の**最も多い同位体**の質量を使う。**質量分析（MS）**でピークを同定するときに必須。

    分野によって使う値が違うので、両方あることを覚えておきましょう。

---

## 2. いろいろな分子で

多分野の分子の分子式・分子量をまとめて求めます。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors

molecules = {
    "caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C",
    "benzene":  "c1ccccc1",
    "ethanol":  "CCO",
}

for name, smi in molecules.items():
    mol = Chem.MolFromSmiles(smi)
    formula = rdMolDescriptors.CalcMolFormula(mol)
    mw = round(Descriptors.MolWt(mol), 2)
    print(f"{name}: {formula}, MW = {mw}")
```

出力:

```text
caffeine: C8H10N4O2, MW = 194.19
benzene: C6H6, MW = 78.11
ethanol: C2H6O, MW = 46.07
```

第2回で電卓のように手計算したのが、SMILES から自動で出せるようになりました。

---

## 3. 元素ごとの原子数を数える

水素を明示的に付けてから、元素ごとに数えます。

```python
from rdkit import Chem
from collections import Counter

mol = Chem.MolFromSmiles("CCO")       # エタノール
mol_h = Chem.AddHs(mol)                # 水素を明示的に追加

symbols = [atom.GetSymbol() for atom in mol_h.GetAtoms()]
print(Counter(symbols))                # 元素ごとの個数
```

出力:

```text
Counter({'H': 6, 'C': 2, 'O': 1})
```

エタノール $C_2H_6O$ の組成（H:6, C:2, O:1）が数えられました。`Chem.AddHs` で省略されていた水素を復活させ、`Counter`（第13回の辞書の仲間）で数えています。

---

## 演習問題

**問1.** グルコース `OCC1OC(O)C(O)C(O)C1O` の分子式・分子量・厳密質量を表示してください（分子式は $C_6H_{12}O_6$ になるはずです）。

**問2.** イブプロフェン `CC(C)Cc1ccc(cc1)C(C)C(=O)O`、パラセタモール `CC(=O)Nc1ccc(O)cc1` の分子式と分子量をまとめて表示してください。

**問3.** カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` の元素組成を、`AddHs` と `Counter` で数えてください（窒素 N が4個含まれているはずです）。

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors
    mol = Chem.MolFromSmiles("OCC1OC(O)C(O)C(O)C1O")
    print("分子式:", rdMolDescriptors.CalcMolFormula(mol))
    print("分子量:", round(Descriptors.MolWt(mol), 2))
    print("厳密質量:", round(Descriptors.ExactMolWt(mol), 4))
    ```

    出力:
    ```text
    分子式: C6H12O6
    分子量: 180.16
    厳密質量: 180.0634
    ```
    （分子量はアスピリンとほぼ同じ180ですが、厳密質量は違います。組成が違うためです。）

??? success "問2 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors
    for name, smi in [("ibuprofen", "CC(C)Cc1ccc(cc1)C(C)C(=O)O"),
                      ("paracetamol", "CC(=O)Nc1ccc(O)cc1")]:
        mol = Chem.MolFromSmiles(smi)
        print(name, rdMolDescriptors.CalcMolFormula(mol), round(Descriptors.MolWt(mol), 2))
    ```

    出力:
    ```text
    ibuprofen C13H18O2 206.28
    paracetamol C8H9NO2 151.16
    ```

??? success "問3 の解答"
    ```python
    from rdkit import Chem
    from collections import Counter
    mol = Chem.AddHs(Chem.MolFromSmiles("CN1C=NC2=C1C(=O)N(C(=O)N2C)C"))
    print(Counter(atom.GetSymbol() for atom in mol.GetAtoms()))
    ```

    出力:
    ```text
    Counter({'H': 10, 'C': 8, 'N': 4, 'O': 2})
    ```
    カフェイン $C_8H_{10}N_4O_2$。窒素が4個含まれています。

---

## この回のまとめ

- `rdMolDescriptors.CalcMolFormula(mol)` で分子式（H も自動カウント）。
- `Descriptors.MolWt(mol)`（分子量）と `Descriptors.ExactMolWt(mol)`（厳密質量）。
- 厳密質量は質量分析（MS）で使う。
- `Chem.AddHs` ＋ `Counter` で元素ごとの原子数を数える。

### 次回予告

[第60回：分子記述子（LogP・極性表面積など）](lesson-60.md) では、分子の性質を数値で表す「記述子」を学びます。創薬でおなじみの指標です。
