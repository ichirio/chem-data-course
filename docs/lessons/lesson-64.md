# 第64回　化学反応をSMARTSで表す

!!! abstract "この回のゴール"
    - 反応を **反応SMARTS** で表す
    - 反応物から生成物を計算する
    - エステル化を例に、反応を"実行"する
    - 所要時間の目安: 60分
    - 使うテーマ：**エステル化・加水分解などの官能基変換**

分子だけでなく、**反応**もパターンとして書けます。「反応物 → 生成物」を RDKit に計算させられます。

`lesson64.py` を作りましょう。

---

## 1. 反応SMARTS の書き方

反応は `反応物 >> 生成物` の形で書きます。原子に `[C:1]` のような**番号（マッピング）**を付け、反応物と生成物で原子がどう対応するかを示します。

エステル化（カルボン酸 + アルコール → エステル + 水）の例：

```text
[CX3:1](=[O:2])[OX2H].[OX2H:3][#6:4]  >>  [CX3:1](=[O:2])[O:3][#6:4]
```

- 左：カルボン酸 `[CX3](=O)[OH]` と アルコール `[OH][C]`（`.` で区切る）
- `>>`：反応の矢印
- 右：エステル（酸のOHが外れ、アルコールのOとつながる）

---

## 2. 反応を実行する

`AllChem.ReactionFromSmarts()` で反応を作り、`RunReactants()` で反応物を反応させます。

```python
from rdkit import Chem
from rdkit.Chem import AllChem

# エステル化の反応
rxn = AllChem.ReactionFromSmarts(
    "[CX3:1](=[O:2])[OX2H].[OX2H:3][#6:4]>>[CX3:1](=[O:2])[O:3][#6:4]"
)

acid = Chem.MolFromSmiles("CC(=O)O")     # 酢酸
alcohol = Chem.MolFromSmiles("CCO")      # エタノール

products = rxn.RunReactants((acid, alcohol))

# 生成物を表示（重複を除く）
seen = set()
for prod_set in products:
    smi = Chem.MolToSmiles(prod_set[0])
    if smi not in seen:
        seen.add(smi)
        print("生成物:", smi)
```

出力:

```text
生成物: CCOC(C)=O
```

酢酸 + エタノール → **酢酸エチル（CCOC(C)=O）**。反応の生成物が正しく計算できました。

!!! note "`RunReactants` はタプルで渡す"
    反応物は `(acid, alcohol)` のように**タプル**で渡します（反応SMARTS で書いた順番に対応）。結果は「生成物の組」のリストで返るので、`prod_set[0]` で最初の生成物を取り出します。

---

## 3. 官能基変換の例：ニトロ化やハロゲン化も

反応SMARTS を変えれば、いろいろな変換を表せます。たとえば第一級アルコールを、酸化してアルデヒドにする（簡略化した例）：

```python
from rdkit import Chem
from rdkit.Chem import AllChem

# アルコール → アルデヒド（簡略化）
rxn = AllChem.ReactionFromSmarts("[CH2:1][OX2H]>>[CH1:1]=O")

ethanol = Chem.MolFromSmiles("CCO")
products = rxn.RunReactants((ethanol,))

for prod_set in products:
    print("生成物:", Chem.MolToSmiles(prod_set[0]))
```

出力:

```text
生成物: CC=O
```

エタノール → アセトアルデヒド（CC=O）。反応をプログラムで扱えると、「この反応をライブラリ全体に適用したらどんな生成物ができるか」を一括で調べられます。

!!! success "反応を扱う意義"
    合成ルートの探索、反応生成物の予測、反応データベースの構築——反応をパターンとして扱えると、有機合成・プロセス化学・材料合成の設計を、計算で支援できます。

!!! warning "反応SMARTS は難しい"
    反応SMARTS を正確に書くのは、実は上級者向けです。この回は「反応もプログラムで扱える」という感覚をつかむのが目的。実際に使うときは、既存の反応ライブラリやAIの助けを借りるのが現実的です。

---

## 演習問題

**問1.** 本文のエステル化反応で、**ギ酸 `C(=O)O`** と **メタノール `CO`** を反応させ、生成物（ギ酸メチル）を表示してください。

**問2.** 本文のエステル化反応で、**酢酸 `CC(=O)O`** と **プロパノール `CCCO`** を反応させ、生成物を表示してください。

**問3.** 本文のアルコール → アルデヒド反応を、**プロパノール `CCCO`** に適用して生成物を表示してください（プロピオンアルデヒドになるはずです）。

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import AllChem
    rxn = AllChem.ReactionFromSmarts("[CX3:1](=[O:2])[OX2H].[OX2H:3][#6:4]>>[CX3:1](=[O:2])[O:3][#6:4]")
    acid = Chem.MolFromSmiles("C(=O)O")   # ギ酸
    alcohol = Chem.MolFromSmiles("CO")    # メタノール
    for prod_set in rxn.RunReactants((acid, alcohol)):
        print("生成物:", Chem.MolToSmiles(prod_set[0]))
        break
    ```

    出力:
    ```text
    生成物: COC=O
    ```
    ギ酸メチル（COC=O）ができました。

??? success "問2 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import AllChem
    rxn = AllChem.ReactionFromSmarts("[CX3:1](=[O:2])[OX2H].[OX2H:3][#6:4]>>[CX3:1](=[O:2])[O:3][#6:4]")
    acid = Chem.MolFromSmiles("CC(=O)O")   # 酢酸
    alcohol = Chem.MolFromSmiles("CCCO")   # プロパノール
    for prod_set in rxn.RunReactants((acid, alcohol)):
        print("生成物:", Chem.MolToSmiles(prod_set[0]))
        break
    ```

    出力:
    ```text
    生成物: CCCOC(C)=O
    ```
    酢酸プロピル（CCCOC(C)=O）です。

??? success "問3 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import AllChem
    rxn = AllChem.ReactionFromSmarts("[CH2:1][OX2H]>>[CH1:1]=O")
    propanol = Chem.MolFromSmiles("CCCO")
    for prod_set in rxn.RunReactants((propanol,)):
        print("生成物:", Chem.MolToSmiles(prod_set[0]))
        break
    ```

    出力:
    ```text
    生成物: CCC=O
    ```
    プロピオンアルデヒド（CCC=O）ができました。

---

## この回のまとめ

- 反応は `反応物 >> 生成物` の反応SMARTS で表す（原子番号でマッピング）。
- `AllChem.ReactionFromSmarts(...)` で作り、`RunReactants((反応物,...))` で実行。
- 生成物は組のリストで返る。`Chem.MolToSmiles` で確認。
- 反応をプログラムで扱えると、生成物予測・合成設計を支援できる。

### 次回予告

[第65回：PubChemからデータを取得する](lesson-65.md) では、世界最大級の化合物データベース PubChem から、分子の情報をプログラムで取得します。
