# 第65回　PubChemからデータを取得する（PubChemPy）

!!! abstract "この回のゴール"
    - 化合物データベース **PubChem** を知る
    - **PubChemPy** で、名前から化合物情報を取得する
    - 分子式・分子量・IUPAC名・SMILES を得る
    - 所要時間の目安: 60分
    - 使うテーマ：**公開データベースの活用**

**PubChem** は、1億件以上の化合物を収録する世界最大級の無料データベース（米国NIH運営）です。プログラムから直接、分子の情報を取得できます。

!!! warning "インターネット接続が必要"
    この回は PubChem のサーバーに問い合わせるため、**ネット接続が必須**です。また、データベースは更新されるので、取得値が将来わずかに変わることがあります。SMILES の表記なども提供元の形式に依存します。

!!! info "準備：PubChemPy"
    ```bash
    pip install pubchempy       # conda環境でも venv でも
    ```

`lesson65.py` を作りましょう。

---

## 1. 名前から化合物を取得する

`pcp.get_compounds(名前, "name")` で、化合物名から情報を取ってきます。

```python
import pubchempy as pcp

# 名前で検索（結果はリスト。最初の1件を使う）
compound = pcp.get_compounds("aspirin", "name")[0]

print("CID:", compound.cid)                         # PubChem のID
print("分子式:", compound.molecular_formula)
print("分子量:", compound.molecular_weight)
print("IUPAC名:", compound.iupac_name)
```

出力:

```text
CID: 2244
分子式: C9H8O4
分子量: 180.16
IUPAC名: 2-acetyloxybenzoic acid
```

アスピリンの正式な情報が取得できました。`cid`（2244）は PubChem 内の固有IDです。

---

## 2. SMILES を取得して RDKit につなぐ

PubChem から SMILES を取れば、これまでの RDKit の処理につなげられます。

```python
import pubchempy as pcp
from rdkit import Chem
from rdkit.Chem import Descriptors

compound = pcp.get_compounds("caffeine", "name")[0]
smiles = compound.connectivity_smiles          # 構造の SMILES

print("カフェインの SMILES:", smiles)

# RDKit で使う
mol = Chem.MolFromSmiles(smiles)
print("RDKitで計算した分子量:", round(Descriptors.MolWt(mol), 2))
```

出力:

```text
カフェインの SMILES: CN1C=NC2=C1C(=O)N(C(=O)N2C)C
RDKitで計算した分子量: 194.19
```

!!! success "データ取得 → 自前の解析、の流れ"
    **PubChem から構造を取得 → RDKit で記述子を計算 → pandas で表に → 可視化**。
    公開データベースを起点に、これまで学んだすべてをつなげられます。実際の研究でも、既知化合物の情報はデータベースから取り、自分の解析に組み込みます。

---

## 3. 複数の化合物をまとめて取得

名前のリストをループして、まとめて情報を集めます。

```python
import pubchempy as pcp
import pandas as pd

names = ["aspirin", "caffeine", "ibuprofen"]
rows = []
for name in names:
    c = pcp.get_compounds(name, "name")[0]
    rows.append({"name": name, "CID": c.cid,
                 "formula": c.molecular_formula,
                 "MW": c.molecular_weight})

df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

出力（取得時点の例）:

```text
     name  CID  formula     MW
  aspirin 2244   C9H8O4 180.16
 caffeine 2519 C8H10N4O2 194.19
ibuprofen 3672 C13H18O2 206.28
```

!!! warning "アクセスは控えめに"
    データベースへの大量・高速なアクセスは、サーバーに負担をかけ、制限されることがあります。**必要な分だけ・間隔をあけて**取得し、一度取ったデータは CSV に保存して使い回しましょう（第32回）。

---

## 演習問題

**問1.** `pcp.get_compounds` で、パラセタモール（`"acetaminophen"` という名前で検索）の CID・分子式・分子量・IUPAC名を表示してください。

**問2.** グルコース（`"glucose"`）の SMILES を PubChem から取得し、その SMILES を RDKit に渡して分子量を計算してください。

**問3.** 3つの溶媒 `["ethanol", "acetone", "toluene"]` の分子式と分子量を PubChem から取得し、DataFrame にまとめてください。

---

## 解答

??? success "問1 の解答"
    ```python
    import pubchempy as pcp
    c = pcp.get_compounds("acetaminophen", "name")[0]
    print("CID:", c.cid)
    print("分子式:", c.molecular_formula)
    print("分子量:", c.molecular_weight)
    print("IUPAC名:", c.iupac_name)
    ```

    出力（取得時点の例）:
    ```text
    CID: 1983
    分子式: C8H9NO2
    分子量: 151.16
    IUPAC名: N-(4-hydroxyphenyl)acetamide
    ```

??? success "問2 の解答"
    ```python
    import pubchempy as pcp
    from rdkit import Chem
    from rdkit.Chem import Descriptors
    c = pcp.get_compounds("glucose", "name")[0]
    mol = Chem.MolFromSmiles(c.connectivity_smiles)
    print("分子量:", round(Descriptors.MolWt(mol), 2))
    ```

    出力（例）:
    ```text
    分子量: 180.16
    ```

??? success "問3 の解答"
    ```python
    import pubchempy as pcp
    import pandas as pd
    rows = []
    for name in ["ethanol", "acetone", "toluene"]:
        c = pcp.get_compounds(name, "name")[0]
        rows.append({"name": name, "formula": c.molecular_formula, "MW": c.molecular_weight})
    print(pd.DataFrame(rows).to_string(index=False))
    ```

    出力（取得時点の例）:
    ```text
     name formula     MW
  ethanol   C2H6O  46.07
  acetone   C3H6O  58.08
  toluene    C7H8  92.14
    ```

---

## この回のまとめ

- PubChem は世界最大級の無料化合物データベース。**ネット接続が必要**。
- `pcp.get_compounds(名前, "name")[0]` で化合物を取得。
- `.cid` `.molecular_formula` `.molecular_weight` `.iupac_name` `.connectivity_smiles` などが得られる。
- 取得した SMILES を RDKit につなげば、自前の解析に組み込める。
- アクセスは控えめに。取得データは保存して使い回す。

### 次回予告

[第66回：分光データ（IR / NMRピーク）を整理する](lesson-66.md) では、スペクトルのピークデータを pandas で整理・解析します。
