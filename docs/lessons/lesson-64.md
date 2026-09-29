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

反応は `反応物 >> 生成物` の形で書きます。原子に `[C:1]` のような**原子マップ番号**を付け、反応物と生成物で原子がどう対応するかを示します。

エステル化（カルボン酸 + アルコール → エステル + 水）の例：

```text
[CX3:1](=[O:2])[OX2H1].[OX2H1:3][CX4:4]  >>  [CX3:1](=[O:2])[O:3][CX4:4]
```

- 左：カルボン酸 `[CX3](=O)[OX2H1]` と アルコール `[OX2H1][CX4]`（sp3 炭素に付いた OH。カルボン酸やフェノールの OH は除く）（`.` で区切る）
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
    "[CX3:1](=[O:2])[OX2H1].[OX2H1:3][CX4:4]>>[CX3:1](=[O:2])[O:3][CX4:4]"
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
    また、返ってくる生成物は化学的な整合性チェック（サニタイズ）前の状態です。記述子を計算する前に `Chem.SanitizeMol(p)` を通しておくと安全です。反応物の順番が反応SMARTSと違うと、エラーではなく空のタプル `()` が返ります。

---

## 3. 官能基変換の例：アルコールの酸化

反応SMARTS を変えれば、いろいろな変換を表せます。たとえば第一級アルコールを、酸化してアルデヒドにする（簡略化した例）。`[CH2:1]` は H が2つ付いた炭素なので、第一級アルコール（メタノールを除く）だけに一致します。酸化剤や過剰酸化（カルボン酸まで進む）は表していない、形式的な変換です。

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

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「RDKit の反応SMARTS で、カルボン酸と第一級・第二級アルコールからエステルを作る反応を書いてください。
    アルコールは sp3 炭素に付いた OH（`[CX4]`）に限り、カルボン酸どうし・フェノールでは反応しないようにしてください。
    (酸, アルコール) のペアのリストに適用し、生成物が無い場合は『生成物なし』と表示、ある場合は `Chem.SanitizeMol` してから canonical SMILES を表示してください。
    テストに酢酸＋酢酸（生成物なしが正解）を含めてください。」

!!! warning "AIの出力で確かめること"
    - **パターンが広すぎないか**：アルコール側が `[OX2H][#6]` だと、カルボン酸の OH にも一致して「酢酸＋酢酸 → 無水酢酸」をエステルとして出力します。「反応しないはずの組み合わせ」を必ず1つ試します。
    - `products[0][0]` をいきなり取っていないか。反応物の順番が逆だと `RunReactants` は空のタプルを返し、`IndexError` になります。`if not products:` で確認します。
    - 生成物の分子式が「酸＋アルコール − H2O」になっているか（酢酸 C2H4O2 ＋ エタノール C2H6O → 酢酸エチル C4H8O2）。
    - 同じ生成物が重複して返ることがある（対称な分子など）ので、canonical SMILES の集合で重複を除いているか。

---

## 演習問題

**問1.** 本文のエステル化反応で、**ギ酸 `C(=O)O`** と **メタノール `CO`** を反応させ、生成物（ギ酸メチル）を表示してください。

**問2.** 本文のエステル化反応で、**酢酸 `CC(=O)O`** と **プロパノール `CCCO`** を反応させ、生成物を表示してください。

**問3.** 本文のアルコール → アルデヒド反応を、**プロパノール `CCCO`** に適用して生成物を表示してください（プロピオンアルデヒドになるはずです）。

**問4.** 次は AI が書いた「複数の組み合わせでエステル化を試す」コードです。問題点を指摘し、直してください。

```python
from rdkit import Chem
from rdkit.Chem import AllChem

rxn = AllChem.ReactionFromSmarts(
    "[CX3:1](=[O:2])[OX2H].[OX2H:3][#6:4]>>[CX3:1](=[O:2])[O:3][#6:4]")

pairs = [("CC(=O)O", "CCO"),        # 酢酸 + エタノール
         ("CC(=O)O", "CC(=O)O"),    # 酢酸 + 酢酸（エステルはできないはず）
         ("CO", "CC(=O)O")]         # メタノール + 酢酸
for a, b in pairs:
    products = rxn.RunReactants((Chem.MolFromSmiles(a), Chem.MolFromSmiles(b)))
    print(a, "+", b, "→ エステル:", Chem.MolToSmiles(products[0][0]))
```

AI の出力:

```text
CC(=O)O + CCO → エステル: CCOC(C)=O
CC(=O)O + CC(=O)O → エステル: CC(=O)OC(C)=O
IndexError: tuple index out of range
```

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import AllChem
    rxn = AllChem.ReactionFromSmarts("[CX3:1](=[O:2])[OX2H1].[OX2H1:3][CX4:4]>>[CX3:1](=[O:2])[O:3][CX4:4]")
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
    rxn = AllChem.ReactionFromSmarts("[CX3:1](=[O:2])[OX2H1].[OX2H1:3][CX4:4]>>[CX3:1](=[O:2])[O:3][CX4:4]")
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

??? success "問4 の解答"
    誤りは2つです。

    1. **アルコールのパターンが広すぎる**：`[OX2H:3][#6:4]` は「炭素に付いた OH」なら何にでも一致するので、酢酸の OH もアルコール扱いされ、無水酢酸 `CC(=O)OC(C)=O` が「エステル」として出ました。sp3 炭素に付いた OH（`[OX2H1:3][CX4:4]`）に限定します。
    2. **生成物が無い場合を考えていない**：3組目は反応物の順番が反応SMARTS（酸, アルコール）と逆なので、`RunReactants` は空のタプルを返し、`products[0]` で `IndexError` になります。

    ```python
    from rdkit import Chem
    from rdkit.Chem import AllChem

    # アルコール側は sp3 炭素に付いた OH（[CX4]）に限定する
    rxn = AllChem.ReactionFromSmarts(
        "[CX3:1](=[O:2])[OX2H1].[OX2H1:3][CX4:4]>>[CX3:1](=[O:2])[O:3][CX4:4]")

    pairs = [("CC(=O)O", "CCO"),
             ("CC(=O)O", "CC(=O)O"),
             ("CO", "CC(=O)O")]
    for a, b in pairs:
        products = rxn.RunReactants((Chem.MolFromSmiles(a), Chem.MolFromSmiles(b)))
        if not products:
            print(a, "+", b, "→ 生成物なし（反応物の種類・順番を確認）")
            continue
        p = products[0][0]
        Chem.SanitizeMol(p)
        print(a, "+", b, "→ エステル:", Chem.MolToSmiles(p))
    ```

    出力:
    ```text
    CC(=O)O + CCO → エステル: CCOC(C)=O
    CC(=O)O + CC(=O)O → 生成物なし（反応物の種類・順番を確認）
    CO + CC(=O)O → 生成物なし（反応物の種類・順番を確認）
    ```
    3組目は `("CC(=O)O", "CO")` と順番を直せば `COC(C)=O`（酢酸メチル）が得られます。

---

## この回のまとめ

- 反応は `反応物 >> 生成物` の反応SMARTS で表す（原子マップ番号で反応前後の原子を対応づける）。
- `AllChem.ReactionFromSmarts(...)` で作り、`RunReactants((反応物,...))` で実行。
- 生成物は組のリストで返る。`Chem.MolToSmiles` で確認。
- 反応をプログラムで扱えると、生成物予測・合成設計を支援できる。

### 次回予告

[第65回：PubChemからデータを取得する](lesson-65.md) では、世界最大級の化合物データベース PubChem から、分子の情報をプログラムで取得します。
