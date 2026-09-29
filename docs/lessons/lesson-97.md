# 第97回　特徴量エンジニアリング（分子記述子を特徴に）

!!! abstract "この回のゴール"
    - RDKit の分子記述子を、機械学習の**特徴量**にする
    - SMILES から特徴量の表（DataFrame）を作る
    - 第6部（分子）と第8部（機械学習）を合流させる
    - 所要時間の目安: 60分
    - 使うテーマ：**分子から機械学習用の特徴を作る**

機械学習の性能は、**どんな特徴量を与えるか**で大きく変わります。分子の場合、RDKit の記述子（第60回）が強力な特徴量になります。

`ml97.py` を作りましょう。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors, rdMolDescriptors
import pandas as pd
```

---

## 1. 分子から記述子の表を作る

SMILES のリストから、複数の記述子を計算して DataFrame にします（第63回のパターン）。この表が機械学習の入力になります。

```python
mols = {
    "methanol": "CO", "ethanol": "CCO", "benzene": "c1ccccc1", "toluene": "Cc1ccccc1",
    "octane": "CCCCCCCC", "aspirin": "CC(=O)Oc1ccccc1C(=O)O",
    "caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C", "glucose": "OCC1OC(O)C(O)C(O)C1O",
    "hexane": "CCCCCC", "naphthalene": "c1ccc2ccccc2c1",
    "ibuprofen": "CC(C)Cc1ccc(cc1)C(C)C(=O)O", "acetic_acid": "CC(=O)O",
}

rows = []
for name, smi in mols.items():
    m = Chem.MolFromSmiles(smi)
    rows.append({
        "name": name,
        "MW":   round(Descriptors.MolWt(m), 1),
        "LogP": round(Descriptors.MolLogP(m), 2),
        "TPSA": round(Descriptors.TPSA(m), 1),
        "HBD":  rdMolDescriptors.CalcNumHBD(m),
        "HBA":  rdMolDescriptors.CalcNumHBA(m),
        "RotB": rdMolDescriptors.CalcNumRotatableBonds(m),
    })

df = pd.DataFrame(rows)
print(df.head(6).to_string(index=False))
```

出力:

```text
    name    MW  LogP  TPSA  HBD  HBA  RotB
methanol  32.0 -0.39  20.2    1    1     0
 ethanol  46.1 -0.00  20.2    1    1     0
 benzene  78.1  1.69   0.0    0    0     0
 toluene  92.1  2.00   0.0    0    0     0
  octane 114.2  3.37   0.0    0    0     5
 aspirin 180.2  1.31  63.6    1    3     2
```

6種類の記述子が特徴量になりました。この `df[["MW","LogP","TPSA","HBD","HBA","RotB"]]` を、機械学習モデルの `X` として使えます。

!!! note "記述子は「計算値」"
    `MolLogP` は原子ごとの寄与を足し合わせる推算値（Wildman–Crippen 法）で、実測値とは一致しません（例：メタノール 計算 −0.39／実測 −0.77、オクタン 計算 3.37／実測 5.18）。`HBA` の数え方もソフトによって定義が異なります。論文やレポートでは、どのソフトのどの関数で計算したかを書きます。

---

## 2. 良い特徴量とは

!!! note "特徴量選びのポイント"
    - **目的に関係する記述子を選ぶ**：溶解性を予測するなら、極性に関わる TPSA・LogP・HBD/HBA が効きそう。
    - **多すぎる特徴量は過学習の元**（第95回）：関係の薄いものは入れない。
    - **スケールをそろえる**：MW（数十〜数百）と HBD（0〜数個）ではスケールが違いすぎる。距離や分散を使う手法（PCA・KMeans、第98・99回）や正則化つきのモデルの前には**標準化**（第43回）が有効（決定木系のモデルでは不要）。

RDKit には200種類以上の記述子があります。`Descriptors.descList` で一覧を取得できますが、まずは意味の分かる基本的なものから始めるのがおすすめです。

---

## 3. 標準化して機械学習の準備

スケールをそろえる標準化（第43回）を、scikit-learn の `StandardScaler` で行います。

```python
from sklearn.preprocessing import StandardScaler

features = ["MW", "LogP", "TPSA", "HBD", "HBA", "RotB"]
X = df[features]

X_scaled = StandardScaler().fit_transform(X)
print("標準化後の平均（各列ほぼ0）:", X_scaled.mean(axis=0).round(2))
```

出力:

```text
標準化後の平均（各列ほぼ0）: [ 0.  0. -0.  0.  0.  0.]
```

各特徴量の平均が0・標準偏差1にそろい、機械学習（特に PCA やクラスタリング）の準備が整いました。

!!! warning "予測モデルでは、標準化は「訓練データだけ」で fit する"
    ここでは全12分子をまとめて標準化しましたが、これは PCA やクラスタリング（第98・99回）のような、正解を使わない**可視化・グループ分け**だから問題ありません。予測モデルを評価するときに、全データで `fit_transform` してから分割すると、テストの分子の平均・標準偏差が学習にもれ込みます（**データ漏洩**、第96回）。`make_pipeline(StandardScaler(), モデル)` にまとめれば、分割や交差検証のたびに訓練データだけで標準化されます。

!!! success "分子 → 特徴量 → 機械学習"
    **SMILES → RDKit で記述子計算 → DataFrame → 標準化 → 機械学習**。
    第6部で学んだ分子処理が、機械学習の入力作りに直結しました。QSAR（構造-活性相関）は、まさにこの流れで分子から活性を予測します（第100回）。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「分子名をキー、SMILES を値とする辞書 `mols` があります。RDKit で各分子の MolWt・MolLogP・TPSA・HBD・HBA・回転可能結合数を計算し、
    1行1分子の pandas DataFrame にしてください。`Chem.MolFromSmiles` が None を返した分子はスキップせず、名前をリストに記録して最後に表示してください。
    入力の分子数と DataFrame の行数も表示してください。」

!!! warning "AIの出力で確かめること"
    - **行数が合っているか**：`len(mols)` と `len(df)` を比べる。`try: ... except: pass` で失敗した分子を黙って捨てるコードは、AI がよく書く危険なパターン。
    - **構造が本当にその分子か**：よく知っている分子の MW を検算する（アスピリン 180.2、イブプロフェン 206.3、カフェイン 194.2）。値が違えば SMILES が別の化合物（例：メチル基の欠落）。
    - **記述子の定義**：`MolLogP` は計算値（実測 LogP ではない）、`HBA` は定義によって数が変わる。レポートにどの関数を使ったか書けるか。
    - **標準化の場所**：`StandardScaler().fit_transform(X)` を全データで行ってから予測モデルを評価していないか（第96回のパイプラインへ）。
    - **標準化後の確認**：`X_scaled.mean(axis=0)` がほぼ 0、`X_scaled.std(axis=0)` が 1。pandas の `.std()` は ddof=1 なので 1 より少し大きく出るのは正常。

---

## 演習問題

**問1.** 本文の `mols` から記述子の DataFrame を作り、全12分子を表示してください。

**問2.** 作った DataFrame で、`MW` と `LogP` の相関を `df["MW"].corr(df["LogP"])` で調べてください（第52回）。特徴量どうしの関係を見るのは、機械学習の前の大事な確認です。

**問3.** `StandardScaler` で `["MW", "LogP", "TPSA"]` の3列を標準化し、標準化後の各列の標準偏差が（ほぼ）1になることを確認してください。

**問4.** 次は、医薬品5種の記述子表を作るために AI が書いたコードと出力です（RDKit のエラーメッセージ行は省略）。問題点を2つ指摘し、直してください。

```python
from rdkit import Chem
from rdkit.Chem import Descriptors
import pandas as pd

mols = {"aspirin": "CC(=O)Oc1ccccc1C(=O)O",
        "ibuprofen": "CC(C)Cc1ccc(cc1)C(=O)O",
        "paracetamol": "CC(=O)Nc1ccc(O)cc1",
        "caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C",
        "naproxen": "COc1ccc2cc(ccc2c1)C(C)C(=O)O)"}

rows = []
for name, smi in mols.items():
    try:
        m = Chem.MolFromSmiles(smi)
        rows.append({"name": name, "MW": round(Descriptors.MolWt(m), 1),
                     "LogP": round(Descriptors.MolLogP(m), 2)})
    except Exception:
        pass
df = pd.DataFrame(rows)
print(df.to_string(index=False))
```

出力:

```text
       name    MW  LogP
    aspirin 180.2  1.31
  ibuprofen 178.2  2.58
paracetamol 151.2  1.35
   caffeine 194.2 -1.03
```

---

## 解答

??? success "問1 の解答"
    ```python
    # 本文のループで df を作ってから
    print(df.to_string(index=False))
    ```
    出力:
    ```text
           name    MW  LogP  TPSA  HBD  HBA  RotB
       methanol  32.0 -0.39  20.2    1    1     0
        ethanol  46.1 -0.00  20.2    1    1     0
        benzene  78.1  1.69   0.0    0    0     0
        toluene  92.1  2.00   0.0    0    0     0
         octane 114.2  3.37   0.0    0    0     5
        aspirin 180.2  1.31  63.6    1    3     2
       caffeine 194.2 -1.03  61.8    0    3     0
        glucose 180.2 -3.22 110.4    5    6     1
         hexane  86.2  2.59   0.0    0    0     3
    naphthalene 128.2  2.84   0.0    0    0     0
      ibuprofen 206.3  3.07  37.3    1    1     4
    acetic_acid  60.1  0.09  37.3    1    1     0
    ```
    `len(df)` が 12 であること（読み込めなかった分子がないこと）も確認しましょう。

??? success "問2 の解答"
    ```python
    print(round(df["MW"].corr(df["LogP"]), 3))
    ```
    出力:
    ```text
    -0.049
    ```
    この12分子では MW と LogP はほぼ無相関（−0.049）で、両方を特徴量に入れる意味があります。強く相関する特徴量どうしは、片方だけで足りることもあります（多重共線性、第84回）。

??? success "問3 の解答"
    ```python
    from sklearn.preprocessing import StandardScaler
    Xs = StandardScaler().fit_transform(df[["MW", "LogP", "TPSA"]])
    print(Xs.std(axis=0).round(3))     # [1. 1. 1.]
    ```

    出力:
    ```text
    [1. 1. 1.]
    ```
    `StandardScaler` は母標準偏差（`ddof=0`）で割ります。pandas の `df.std()`（`ddof=1`）で確かめると 1.044 になり、「1 にならない」と慌てる原因になります。

??? success "問4 の解答"
    **誤り1（分子が黙って消えている）**：5分子を入れたのに表は4行で、naproxen がありません。naproxen の SMILES は末尾に余分な `)` があり、`MolFromSmiles` が `None` を返し、`MolWt(None)` のエラーを `except: pass` が握りつぶしています。`None` は明示的に調べ、読めなかった分子を記録します。

    **誤り2（構造の誤り）**：ibuprofen の MW が 178.2 です。イブプロフェン（C13H18O2）は 206.3 のはず。SMILES `CC(C)Cc1ccc(cc1)C(=O)O` は α-メチル基が抜けた 4-イソブチル安息香酸です。正しくは `CC(C)Cc1ccc(cc1)C(C)C(=O)O`。

    確かめ方：入力数と行数（5 → 4）を比べる。よく知っている分子の分子量を検算する。

    ```python
    from rdkit import Chem
    from rdkit.Chem import Descriptors
    import pandas as pd

    mols = {"aspirin": "CC(=O)Oc1ccccc1C(=O)O",
            "ibuprofen": "CC(C)Cc1ccc(cc1)C(C)C(=O)O",
            "paracetamol": "CC(=O)Nc1ccc(O)cc1",
            "caffeine": "CN1C=NC2=C1C(=O)N(C(=O)N2C)C",
            "naproxen": "COc1ccc2cc(ccc2c1)C(C)C(=O)O"}

    rows, failed = [], []
    for name, smi in mols.items():
        m = Chem.MolFromSmiles(smi)
        if m is None:                 # 読めなかった分子は記録して、あとで必ず確認する
            failed.append(name)
            continue
        rows.append({"name": name, "MW": round(Descriptors.MolWt(m), 1),
                     "LogP": round(Descriptors.MolLogP(m), 2)})
    df = pd.DataFrame(rows)
    print(df.to_string(index=False))
    print("読めなかった分子:", failed)
    print("分子数:", len(mols), "→", len(df))
    ```

    出力:
    ```text
           name    MW  LogP
        aspirin 180.2  1.31
      ibuprofen 206.3  3.07
    paracetamol 151.2  1.35
       caffeine 194.2 -1.03
       naproxen 230.3  3.04
    読めなかった分子: []
    分子数: 5 → 5
    ```

---

## この回のまとめ

- RDKit の分子記述子を、機械学習の**特徴量**にする（SMILES → 記述子 → DataFrame）。
- 目的に関係する記述子を選ぶ。多すぎは過学習の元。
- `StandardScaler` でスケールをそろえる（PCA・クラスタリングの前に有効）。
- 「分子 → 特徴量 → 機械学習」が QSAR の基本の流れ。

### 次回予告

[第98回：次元削減と可視化（PCA）](lesson-98.md) では、たくさんの記述子を2次元に圧縮して、化合物の分布を可視化します。
