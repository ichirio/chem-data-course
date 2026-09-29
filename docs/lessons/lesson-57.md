# 第57回　SMILESで分子を表現する

!!! abstract "この回のゴール"
    - **SMILES 記法**の基本ルールを読める
    - 多分野の分子を SMILES で表す
    - 標準形（canonical SMILES）を理解する
    - 所要時間の目安: 60分
    - 使うテーマ：**医薬・石油化学・高分子モノマー・生化学**

**SMILES**（Simplified Molecular Input Line Entry System）は、分子構造を1行の文字列で表す記法です。分子データのやり取りに欠かせません。

`lesson57.py` を作りましょう。`from rdkit import Chem` から。

---

## 1. SMILES の基本ルール

| 書き方 | 意味 | 例 |
|---|---|---|
| 元素記号 | 原子 | `C`（炭素）, `O`（酸素）, `N`（窒素） |
| 記号を並べる | 単結合でつながる | `CCO`（C-C-O＝エタノール） |
| `=` `#` | 二重・三重結合 | `C=C`（エチレン）, `C#N`（シアン化水素 HCN） |
| 小文字 | 芳香環の原子 | `c1ccccc1`（ベンゼン） |
| 数字 | 環の開閉 | `C1CCCCC1`（シクロヘキサン） |
| `( )` | 枝分かれ | `CC(C)C`（イソブタン） |

水素は普通、書きません（自動で補われます）。

!!! note "立体化学：`@` `@@` と `/` `\`"
    `@` / `@@` は不斉中心の立体配置、`/` `\` は二重結合の E/Z を表します。たとえば L-アラニンは `C[C@H](N)C(=O)O` です。
    これらを省いた SMILES は「立体が未指定」の構造になり、L体とD体、グルコースとガラクトース（どちらも `OCC1OC(O)C(O)C(O)C1O`）を区別できません。
    この講座でも分子量や LogP の計算では立体を省くことがありますが、「その名前の化合物そのもの」を表したいときは立体まで書きます。

---

## 2. 多分野の分子を SMILES で

いろいろな分野の分子を、SMILES から読み込んでみます。

```python
from rdkit import Chem

molecules = {
    "エタノール（溶媒）":       "CCO",
    "ベンゼン（石油化学）":     "c1ccccc1",
    "オクタン（燃料）":         "CCCCCCCC",
    "スチレン（高分子モノマー）": "C=Cc1ccccc1",
    "グリシン（アミノ酸）":     "NCC(=O)O",
    "アスピリン（医薬）":       "CC(=O)Oc1ccccc1C(=O)O",
}

for name, smi in molecules.items():
    mol = Chem.MolFromSmiles(smi)
    print(f"{name}: 重原子 {mol.GetNumAtoms()} 個")
```

出力:

```text
エタノール（溶媒）: 重原子 3 個
ベンゼン（石油化学）: 重原子 6 個
オクタン（燃料）: 重原子 8 個
スチレン（高分子モノマー）: 重原子 8 個
グリシン（アミノ酸）: 重原子 5 個
アスピリン（医薬）: 重原子 13 個
```

たった1行の文字列で、これだけ多様な分子を表せます。SMILES は分野を問わない共通言語です。

---

## 3. canonical SMILES（標準形）

同じ分子でも、SMILES の書き方は何通りもあります。RDKit に通すと、**一意な標準形**（canonical SMILES）に直せます（一意なのは同じツール・同じバージョンの中での話です）。これで「同じ分子か」を文字列比較で判定できます。

```python
from rdkit import Chem

# エタノールを2通りの書き方で
smi_a = "CCO"
smi_b = "OCC"      # 逆から書いても同じ分子

canon_a = Chem.MolToSmiles(Chem.MolFromSmiles(smi_a))
canon_b = Chem.MolToSmiles(Chem.MolFromSmiles(smi_b))

print("CCO →", canon_a)
print("OCC →", canon_b)
print("同じ分子？", canon_a == canon_b)
```

出力:

```text
CCO → CCO
OCC → CCO
同じ分子？ True
```

`OCC` も `CCO` も、標準形にすると同じ `CCO`。**書き方が違っても同じ分子**だと判定できました。

!!! tip "canonical SMILES の使いどころ"
    データベースで「この分子は既に登録済みか？」を調べるとき、canonical SMILES どうしを比較すれば、書き方の違いに惑わされず判定できます。重複除去にも使います。ほかのツールやデータベースの構造と照合するときは、ツールに依存しにくい **InChIKey**（`Chem.MolToInchiKey(mol)`）も併用します。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「化合物名と SMILES の辞書 `{"エタノール": "CCO", ...}` を RDKit で読み込み、
    名前・入力 SMILES・canonical SMILES（`Chem.MolToSmiles`）・分子式（`rdMolDescriptors.CalcMolFormula`）の表を表示してください。
    canonical SMILES が同じものは同一化合物として数え、重複の件数も表示してください。
    SMILES は私が名前と照合するので、AI が SMILES を新しく作る場合は出典（PubChem の CID など）を書いてください。」

!!! warning "AIの出力で確かめること"
    - **AI が書いた SMILES が本当にその名前の化合物か**：分子式を `CalcMolFormula` で表示し、名前から分かる分子式と比べます（酢酸エチルなら C4H8O2。`CC(=O)OC` は C3H6O2 の酢酸メチル）。
    - 重複判定を**生の SMILES 文字列**で行っていないか。`"CCO"` と `"OCC"` は文字列としては別物です。必ず `Chem.MolToSmiles(Chem.MolFromSmiles(s))` で標準形にしてから比べます。
    - 立体を持つ化合物（アミノ酸・糖・多くの医薬品）の SMILES に `@` / `@@` があるか。無ければ「立体未指定」で、L体/D体などが区別されません。
    - 塩（`.` で区切られた `[Na+]` や `Cl` など）や電荷（`[O-]`）を含む SMILES が混ざっていないか。同じ薬でも塩の形だと canonical SMILES が一致しません。

---

## 演習問題

**問1.** 次の分子を SMILES から読み込み、それぞれの重原子数を表示してください：トルエン `Cc1ccccc1`（石油化学）、酢酸 `CC(=O)O`、エチレン `C=C`（高分子モノマー）。

**問2.** プロパノールを2通りの SMILES `CCCO` と `OCCC` で読み込み、`Chem.MolToSmiles` で標準形にして、同じ分子と判定されることを確認してください。

**問3.** カフェイン `CN1C=NC2=C1C(=O)N(C(=O)N2C)C` を読み込み、canonical SMILES を表示してください（入力とは違う形になるかもしれません）。

**問4.** 次は AI が書いた「化合物リストの重複を数える」コードです。問題点を指摘し、直してください。

```python
from rdkit import Chem

data = {
    "エタノール":          "CCO",
    "エタノール（別表記）": "OCC",
    "酢酸":                "CC(=O)O",
    "酢酸エチル":          "CC(=O)OC",
}
unique = set(data.values())
print("異なる化合物:", len(unique), "種類")
```

AI の出力:

```text
異なる化合物: 4 種類
```

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    for name, smi in [("トルエン", "Cc1ccccc1"), ("酢酸", "CC(=O)O"), ("エチレン", "C=C")]:
        mol = Chem.MolFromSmiles(smi)
        print(name, mol.GetNumAtoms())
    ```

    出力:
    ```text
    トルエン 7
    酢酸 4
    エチレン 2
    ```

??? success "問2 の解答"
    ```python
    from rdkit import Chem
    a = Chem.MolToSmiles(Chem.MolFromSmiles("CCCO"))
    b = Chem.MolToSmiles(Chem.MolFromSmiles("OCCC"))
    print(a, b, a == b)
    ```

    出力:
    ```text
    CCCO CCCO True
    ```

??? success "問3 の解答"
    ```python
    from rdkit import Chem
    smi = "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"
    print(Chem.MolToSmiles(Chem.MolFromSmiles(smi)))
    ```

    出力:
    ```text
    Cn1c(=O)c2c(ncn2C)n(C)c1=O
    ```
    入力とは違う文字列ですが、同じカフェイン分子の標準形です。

??? success "問4 の解答"
    誤りは2つです。

    1. **文字列のまま比べている**：`"CCO"` と `"OCC"` は同じエタノールですが、文字列としては別物なので 4 種類と数えられます。canonical SMILES に直してから比べます。
    2. **SMILES が名前と合っていない**：`CC(=O)OC` は酢酸**メチル**（C3H6O2）です。酢酸エチル（C4H8O2）は `CCOC(C)=O`。分子式を表示すれば気づけます。

    ```python
    from rdkit import Chem
    from rdkit.Chem import rdMolDescriptors

    data = {
        "エタノール":          "CCO",
        "エタノール（別表記）": "OCC",
        "酢酸":                "CC(=O)O",
        "酢酸エチル":          "CCOC(C)=O",
    }
    canon = {}
    for name, smi in data.items():
        mol = Chem.MolFromSmiles(smi)
        canon[name] = Chem.MolToSmiles(mol)
        print(name, canon[name], rdMolDescriptors.CalcMolFormula(mol))
    print("異なる化合物:", len(set(canon.values())), "種類")
    ```

    出力:
    ```text
    エタノール CCO C2H6O
    エタノール（別表記） CCO C2H6O
    酢酸 CC(=O)O C2H4O2
    酢酸エチル CCOC(C)=O C4H8O2
    異なる化合物: 3 種類
    ```

---

## この回のまとめ

- SMILES は分子を1行の文字列で表す記法（元素・結合・環・枝分かれ）。
- 芳香環は小文字（`c1ccccc1`）、二重結合は `=`、枝分かれは `( )`。
- 分野を問わず分子を表せる共通言語。
- `Chem.MolToSmiles()` で **canonical SMILES**（一意な標準形）に。同一判定・重複除去に使う。

### 次回予告

[第58回：分子を描画する](lesson-58.md) では、SMILES から**分子の構造式を画像**として描きます。
