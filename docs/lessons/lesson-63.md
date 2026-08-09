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
4  cyclohexane   C6H12   84.16  2.34
1      toluene    C7H8   92.14  2.00
0      benzene    C6H6   78.11  1.69
```

「脂溶性の高い分子はどれ？」「一番重い分子は？」に、すぐ答えられます。

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

## 演習問題

**問1.** 医薬分子のライブラリ `[("aspirin", "CC(=O)Oc1ccccc1C(=O)O"), ("caffeine", "CN1C=NC2=C1C(=O)N(C(=O)N2C)C"), ("ibuprofen", "CC(C)Cc1ccc(cc1)C(C)C(=O)O")]` から、name・formula・MW・LogP の DataFrame を作って表示してください。

**問2.** 問1の DataFrame から、`LogP` が 0 より大きい分子だけを取り出してください（カフェインは LogP が負なので除かれるはずです）。

**問3.** 問1の DataFrame を分子量（MW）の大きい順に並べ替え、`drug_library.csv` に保存してください。

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

---

## この回のまとめ

- 定番パターン：**空リスト → 分子ごとに辞書で append → `pd.DataFrame`**。
- DataFrame になれば、第4部のフィルタ・並べ替え・集計・保存が全部使える。
- SMILES → 性質計算 → 表 → 分析、が化合物ライブラリ処理の基本フロー。
- 分子（第6部）とデータ分析（第4・5部）が合流する。

### 次回予告

[第64回：化学反応をSMARTSで表す](lesson-64.md) では、反応をパターンとして表し、生成物を予測します。
