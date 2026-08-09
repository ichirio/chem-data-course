# 第56回　RDKit入門：分子を扱うライブラリ

!!! abstract "この回のゴール"
    - **ケモインフォマティクス**と **RDKit** が何かを知る
    - SMILES 文字列から分子オブジェクトを作る
    - 分子の基本情報（原子数など）を取り出す
    - 所要時間の目安: 60分
    - 使うテーマ：**多分野の分子**（医薬・石油化学・生化学…）

!!! info "第6部スタート：化学に特化したプログラミング"
    ここからは、化学そのもの——**分子の構造**をコンピュータで扱います。第4部・第5部が「表やグラフ」だったのに対し、第6部は「分子」が主役。**RDKit** は、そのための世界標準の無料ライブラリです。

---

## 1. ケモインフォマティクスは、あらゆる化学分野で使える

分子を「データ」として扱う技術（ケモインフォマティクス）は、特定の分野だけのものではありません。

| 分野 | 使いどころの例 |
|---|---|
| **創薬・医薬** | 化合物ライブラリの性質予測、類似化合物探索 |
| **生化学** | アミノ酸・代謝物・天然物の構造解析 |
| **高分子** | モノマーの設計、繰り返し単位の性質 |
| **石油化学** | 炭化水素の分類、物性の推定 |
| **材料・触媒** | 分子設計、構造-物性相関 |
| **農薬・環境** | 化合物の毒性・分解性の予測 |

**「分子の構造から性質を読み解く」**という共通の道具が RDKit です。この第6部の技術は、あなたがどの分野に進んでも役立ちます。

---

## 2. RDKit を使う準備

第1回で RDKit を入れていない場合は、次で導入します。

```bash
conda install -c conda-forge rdkit -y   # conda環境の人（推奨）
pip install rdkit                        # venvの人
```

`lesson56.py` を作り、動作確認しましょう。

```python
from rdkit import Chem

# SMILES という文字列から分子を作る（エタノール）
mol = Chem.MolFromSmiles("CCO")

print(mol)                          # 分子オブジェクト
print("原子数:", mol.GetNumAtoms())  # 水素を除いた原子数
```

出力:

```text
<rdkit.Chem.rdchem.Mol object at 0x...>
原子数: 3
```

`CCO` はエタノール $C_2H_6O$ の SMILES 表記。RDKit がそれを「分子」として理解し、原子数（C, C, O の3つ。水素は省略表示）を返しました。

!!! note "SMILES とは"
    分子の構造を**文字列で表す記法**です（次回で詳しく）。`CCO`＝エタノール、`c1ccccc1`＝ベンゼン、のように、分子を1行のテキストで書けます。データベースやファイルでの分子のやり取りに使われます。

---

## 3. 分子の基本情報

分子オブジェクトから、いろいろな情報を取り出せます。

```python
from rdkit import Chem

mol = Chem.MolFromSmiles("CC(=O)Oc1ccccc1C(=O)O")   # アスピリン

print("原子数(重原子):", mol.GetNumAtoms())
print("結合数:", mol.GetNumBonds())

# 各原子の元素記号を見る
for atom in mol.GetAtoms():
    print(atom.GetSymbol(), end=" ")
print()
```

出力:

```text
原子数(重原子): 13
結合数: 13
C C O O C C C C C C C O O 
```

アスピリンの重原子（水素以外）が13個あり、その元素記号が順に取り出せました。分子を「原子と結合の集まり」としてプログラムで扱えるのが RDKit です。

!!! warning "SMILES が間違っていると None になる"
    `Chem.MolFromSmiles("間違ったSMILES")` は `None`（分子でない）を返します。読み込んだら `if mol is None:` でチェックする習慣をつけると安全です（第10回の例外処理の発想）。

---

## 演習問題

**問1.** RDKit を import し、メタン `C`、水 `O`、二酸化炭素 `O=C=O` の分子を作って、それぞれの重原子数（`GetNumAtoms()`）を表示してください。

**問2.** ベンゼン `c1ccccc1` の分子を作り、原子数と結合数を表示してください。6員環なので、どちらも6になるはずです。

**問3.** 存在しない出鱈目な SMILES（例：`"XYZ123"`）を `MolFromSmiles` に渡すとどうなるか確かめ、`if mol is None:` で「無効な SMILES です」と表示するコードを書いてください。

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    for name, smi in [("メタン", "C"), ("水", "O"), ("二酸化炭素", "O=C=O")]:
        mol = Chem.MolFromSmiles(smi)
        print(name, mol.GetNumAtoms())
    ```

    出力:
    ```text
    メタン 1
    水 1
    二酸化炭素 3
    ```
    （水素は省略されるので、メタンは C の1つ、水は O の1つと数えられます。）

??? success "問2 の解答"
    ```python
    from rdkit import Chem
    mol = Chem.MolFromSmiles("c1ccccc1")
    print("原子数:", mol.GetNumAtoms())
    print("結合数:", mol.GetNumBonds())
    ```

    出力:
    ```text
    原子数: 6
    結合数: 6
    ```

??? success "問3 の解答"
    ```python
    from rdkit import Chem
    mol = Chem.MolFromSmiles("XYZ123")
    if mol is None:
        print("無効な SMILES です")
    else:
        print("原子数:", mol.GetNumAtoms())
    ```

    出力:
    ```text
    無効な SMILES です
    ```
    （RDKit は警告メッセージも表示しますが、`None` チェックで安全に処理できます。）

---

## この回のまとめ

- ケモインフォマティクス＝分子をデータとして扱う技術。**全化学分野**で有用。
- RDKit は無料の標準ライブラリ。`from rdkit import Chem`。
- `Chem.MolFromSmiles("SMILES")` で文字列から分子を作る。
- `GetNumAtoms()` `GetNumBonds()` `GetAtoms()` で構造情報を取得。
- 無効な SMILES は `None`。読み込み後にチェックする。

### 次回予告

[第57回：SMILESで分子を表現する](lesson-57.md) では、分子を文字列で表す SMILES 記法を、多分野の分子例で学びます。
