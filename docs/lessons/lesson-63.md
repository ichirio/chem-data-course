# 第63回　化合物ライブラリを一括処理する

!!! abstract "この回のゴール"
    - たくさんの分子（化合物ライブラリ）をまとめて処理する
    - RDKit の計算結果を **pandas の DataFrame** にまとめる
    - 性質でフィルタ・並べ替えする
    - 所要時間の目安: 60分
    - 使うテーマ：**炭化水素ライブラリ**の物性一覧

第6部（分子）と第4部（pandas）が合流します。「たくさんの分子の性質を表にまとめて分析する」——実務でいちばん使う処理です。

`lesson63.py` を作りましょう。

---

## 1. 分子のリストを性質の表にする

SMILES のリストをループし、各分子の性質を計算して、DataFrame にまとめます。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors
import pandas as pd

# 炭化水素のライブラリ（石油化学）
library = [
    ("benzene",     "c1ccccc1"),
    ("toluene",     "Cc1ccccc1"),
    ("o-xylene",    "Cc1ccccc1C"),
    ("octane",      "CCCCCCCC"),
    ("cyclohexane", "C1CCCCC1"),
]

rows = []
for name, smi in library:
    mol = Chem.MolFromSmiles(smi)
    rows.append({
        "name":    name,
        "formula": rdMolDescriptors.CalcMolFormula(mol),
        "MW":      round(Descriptors.MolWt(mol), 2),
        "LogP":    round(Descriptors.MolLogP(mol), 2),
    })

df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

出力:

```text
       name formula     MW  LogP
    benzene    C6H6  78.11  1.69
    toluene    C7H8  92.14  2.00
   o-xylene   C8H10 106.17  2.30
     octane   C8H18 114.23  3.37
cyclohexane   C6H12  84.16  2.34
```

分子の集まりが、きれいな性質テーブルになりました。ここまで来れば、第4部で学んだ pandas の全機能が使えます。

!!! tip "パターンを覚える"
    **「空リスト rows を用意 → 各分子を計算して辞書で append → DataFrame にする」**。これはRDKitとpandasをつなぐ定番パターンです。分子が10個でも1万個でも、同じコードで表になります。

---

## 2. 表を分析する（第4部の合流）

DataFrame になれば、フィルタ・並べ替え・集計が自由自在です。

```python
# LogP が 2.0 より大きい分子だけ（第34回のフィルタ）
print(df[df["LogP"] > 2.0])

# 分子量の大きい順に並べ替え（第36回）
print(df.sort_values("MW", ascending=False))
```

出力:

```text
          name formula      MW  LogP
2     o-xylene   C8H10  106.17  2.30
3       octane   C8H18  114.23  3.37
4  cyclohexane   C6H12   84.16  2.34
          name formula      MW  LogP
3       octane   C8H18  114.23  3.37
2     o-xylene   C8H10  106.17  2.30
1      toluene    C7H8   92.14  2.00
4  cyclohexane   C6H12   84.16  2.34
0      benzene    C6H6   78.11  1.69
```

「脂溶性の高い分子はどれ？」「一番重い分子は？」に、すぐ答えられます。

なお、トルエンの LogP は実際には 1.995 で、表示用に `round(..., 2)` した 2.00 が表に入っています。しきい値ちょうど付近の値は、丸める前の値で判定するほうが安全です。

---

## 3. CSV に保存して再利用

作った表は CSV に保存でき（第32回）、次の解析や可視化（第5部）に使えます。

```python
df.to_csv("hydrocarbon_library.csv", index=False)
print("保存しました")
```

!!! success "これが実務のデータフロー"
    **SMILES のリスト → RDKit で性質計算 → DataFrame → フィルタ・集計・可視化・保存。**
    大量の化合物を扱うスクリーニングも、材料候補の絞り込みも、この流れです。分子（第6部）が、データ分析（第4部・第5部）に完全につながりました。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「(名前, SMILES) のタプルのリストから、RDKit で分子式・MolWt・MolLogP を計算し、pandas の DataFrame（列：name, formula, MW, LogP）にしてください。
    `MolFromSmiles` が `None` になった行は黙って捨てずに、名前のリストとして最後に表示してください。
    入力件数・表の行数・失敗件数を print し、表は MW の大きい順に並べてから `library.csv` に保存してください。」

!!! warning "AIの出力で確かめること"
    - **行数の突き合わせ**：`len(library)` と `len(df)` を print して一致するか。`try: ... except: pass` で失敗行を黙って捨てるコードは、表が「きれいに」見えるので気づきにくい誤りです。
    - **名前と分子式の照合**：formula 列を見て、名前と合っているか（シクロヘキセンなら C6H10。`C1CCCCC1` はシクロヘキサン C6H12）。
    - **並べ替えの結果**：`sort_values("MW", ascending=False)` の結果が本当に単調減少か、MW 列を上から目で追います（`df["MW"].is_monotonic_decreasing` でも確認できます）。
    - 丸めた値（`round(..., 2)`）でしきい値判定していないか。

---

## 演習問題

**問1.** 医薬分子のライブラリ `[("aspirin", "CC(=O)Oc1ccccc1C(=O)O"), ("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"), ("ibuprofen", "CC(C)Cc1ccc(cc1)C(C)C(=O)O")]` から、name・formula・MW・LogP の DataFrame を作って表示してください。

**問2.** 問1の DataFrame から、`LogP` が 0 より大きい分子だけを取り出してください（カフェインは LogP が負なので除かれるはずです）。

**問3.** 問1の DataFrame を分子量（MW）の大きい順に並べ替え、`drug_library.csv` に保存してください。

**問4.** 次は AI が書いた「炭化水素ライブラリの性質表」を作るコードです。4件入れたのに3行しか出てきません。問題点を指摘し、直してください。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors
import pandas as pd

library = [
    ("benzene",     "c1ccccc1"),
    ("toluene",     "Cc1ccccc1"),
    ("cyclohexene", "C1CCCCC1"),
    ("ethylbenzene", "CCc1ccccc"),
]
rows = []
for name, smi in library:
    try:
        mol = Chem.MolFromSmiles(smi)
        rows.append({"name": name, "formula": rdMolDescriptors.CalcMolFormula(mol),
                     "MW": round(Descriptors.MolWt(mol), 2)})
    except Exception:
        pass
df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

AI の出力:

```text
       name formula    MW
    benzene    C6H6 78.11
    toluene    C7H8 92.14
cyclohexene   C6H12 84.16
```

---

## 解答

??? success "問1 の解答"
    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors
    import pandas as pd

    library = [
        ("aspirin", "CC(=O)Oc1ccccc1C(=O)O"),
        ("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"),
        ("ibuprofen", "CC(C)Cc1ccc(cc1)C(C)C(=O)O"),
    ]
    rows = []
    for name, smi in library:
        mol = Chem.MolFromSmiles(smi)
        rows.append({"name": name,
                     "formula": rdMolDescriptors.CalcMolFormula(mol),
                     "MW": round(Descriptors.MolWt(mol), 2),
                     "LogP": round(Descriptors.MolLogP(mol), 2)})
    df = pd.DataFrame(rows)
    print(df.to_string(index=False))
    ```

    出力:
    ```text
         name   formula     MW  LogP
      aspirin    C9H8O4 180.16  1.31
     caffeine C8H10N4O2 194.19 -1.03
    ibuprofen  C13H18O2 206.28  3.07
    ```

??? success "問2 の解答"
    ```python
    print(df[df["LogP"] > 0])
    ```

    出力:
    ```text
            name   formula      MW  LogP
    0    aspirin    C9H8O4  180.16  1.31
    2  ibuprofen  C13H18O2  206.28  3.07
    ```
    カフェイン（LogP = −1.03）が除かれました。

??? success "問3 の解答"
    ```python
    df_sorted = df.sort_values("MW", ascending=False)
    df_sorted.to_csv("drug_library.csv", index=False)
    print(df_sorted.to_string(index=False))
    ```

    出力:
    ```text
         name   formula     MW  LogP
    ibuprofen  C13H18O2 206.28  3.07
     caffeine C8H10N4O2 194.19 -1.03
      aspirin    C9H8O4 180.16  1.31
    ```

??? success "問4 の解答"
    誤りは2つです。

    1. **失敗を黙って捨てている**：`ethylbenzene` の `CCc1ccccc` は環が閉じていない（`1` が1回しかない）無効な SMILES で、`MolFromSmiles` が `None` を返し、`CalcMolFormula(None)` の例外が `except: pass` で握りつぶされています。`None` を明示的にチェックし、失敗した名前を記録します。正しい SMILES は `CCc1ccccc1`。
    2. **名前と構造が合っていない**：`C1CCCCC1` はシクロヘキサン（C6H12）。シクロヘキセンは二重結合を1つ持つ `C1=CCCCC1`（C6H10）です。formula 列を見れば気づけます。

    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors, rdMolDescriptors
    import pandas as pd

    library = [
        ("benzene",      "c1ccccc1"),
        ("toluene",      "Cc1ccccc1"),
        ("cyclohexene",  "C1=CCCCC1"),
        ("ethylbenzene", "CCc1ccccc1"),
    ]
    rows, failed = [], []
    for name, smi in library:
        mol = Chem.MolFromSmiles(smi)
        if mol is None:
            failed.append(name)
            continue
        rows.append({"name": name, "formula": rdMolDescriptors.CalcMolFormula(mol),
                     "MW": round(Descriptors.MolWt(mol), 2)})
    df = pd.DataFrame(rows)
    print(df.to_string(index=False))
    print("入力:", len(library), "件 / 表:", len(df), "件 / 失敗:", failed)
    ```

    出力:
    ```text
            name formula     MW
         benzene    C6H6  78.11
         toluene    C7H8  92.14
     cyclohexene   C6H10  82.15
    ethylbenzene   C8H10 106.17
    入力: 4 件 / 表: 4 件 / 失敗: []
    ```

---

## この回のまとめ

- 定番パターン：**空リスト → 分子ごとに辞書で append → `pd.DataFrame`**。
- DataFrame になれば、第4部のフィルタ・並べ替え・集計・保存が全部使える。
- SMILES → 性質計算 → 表 → 分析、が化合物ライブラリ処理の基本フロー。
- 分子（第6部）とデータ分析（第4・5部）が合流する。

### 次回予告

[第64回：化学反応をSMARTSで表す](lesson-64.md) では、反応をパターンとして表し、生成物を予測します。
