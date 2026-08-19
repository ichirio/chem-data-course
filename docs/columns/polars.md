# 発展コラム：Polars ── pandasと「こう書ける」を比べる

!!! abstract "このコラムのねらい"
    - **Polars（ポーラーズ）** が何者で、なぜ速いのかを知る
    - pandasで書いた処理を **Polarsならどう書くか** を対訳で見る
    - **どちらを使うべきか（使い分け）** の判断基準を持つ
    - このコラムは読み物です。第4部（pandas）を一通りやったあとに読むと、いちばん腑に落ちます。

!!! note "前提"
    このコースの主役は **pandas** のままです。周辺ツール（RDKit・seaborn・scikit-learn）がpandasを前提にしているからです。Polarsは「pandasが重い・遅いと感じたときの選択肢」として押さえておくと、実務で効いてきます。

---

## 1. Polarsって何？

**Polars** は、pandasと同じ「表（DataFrame）を扱うライブラリ」です。中身が **Rust** という高速な言語で書かれていて、次のような特徴があります。

- **速い**：CPUの複数コアを自動で使う（マルチスレッド）。pandasは基本1コアなので、数百万行のデータでは数倍〜数十倍差がつくことも。
- **省メモリ**：内部が **Apache Arrow** という効率的な形式。pandasの半分程度のメモリで済むこともある。
- **書き方が一貫している**：`pl.col("列名")` を起点にした **式（Expression）** で、選択・計算・集計を同じ文法で書ける。
- **型と欠損に厳格**：`null`（欠損）と `NaN`（非数）を区別し、型が曖昧なまま処理が進みにくい。

インストールは1行です（このコースの環境に追加する場合）。

```bash
# conda（Miniforge）を使っている場合
conda install -c conda-forge polars

# pip の場合
pip install polars
```

```python
import polars as pl
print(pl.__version__)   # 例: 1.25.2
```

---

## 2. pandas →  Polars 対訳

第36回でも使った **高分子（ポリマー）** のデータで、同じ処理を pandas と Polars で並べます。タブを切り替えて見比べてください。

まずデータを用意します。

=== "pandas"

    ```python
    import pandas as pd

    polymers = pd.DataFrame({
        "polymer":     ["PE","PP","PS","PVC","PET","PMMA","Nylon6","Epoxy","Phenolic"],
        "category":    ["thermoplastic"]*7 + ["thermoset"]*2,
        "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],
        "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
        "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],
    })
    ```

=== "Polars"

    ```python
    import polars as pl

    polymers = pl.DataFrame({
        "polymer":     ["PE","PP","PS","PVC","PET","PMMA","Nylon6","Epoxy","Phenolic"],
        "category":    ["thermoplastic"]*7 + ["thermoset"]*2,
        "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],
        "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
        "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],
    })
    ```

`pd.DataFrame({...})` が `pl.DataFrame({...})` になっただけ。**辞書からの作り方は同じ**です。

Polarsで `print(polymers.head(3))` すると、pandasと違って **形（shape）と各列の型** も一緒に表示されます。

```text
shape: (3, 5)
┌─────────┬───────────────┬─────────┬─────────────┬──────┐
│ polymer ┆ category      ┆ density ┆ tensile_MPa ┆ Tg_C │
│ ---     ┆ ---           ┆ ---     ┆ ---         ┆ ---  │
│ str     ┆ str           ┆ f64     ┆ i64         ┆ i64  │
╞═════════╪═══════════════╪═════════╪═════════════╪══════╡
│ PE      ┆ thermoplastic ┆ 0.94    ┆ 25          ┆ -110 │
│ PP      ┆ thermoplastic ┆ 0.905   ┆ 35          ┆ -10  │
│ PS      ┆ thermoplastic ┆ 1.05    ┆ 45          ┆ 100  │
└─────────┴───────────────┴─────────┴─────────────┴──────┘
```

`str`（文字列）・`f64`（小数）・`i64`（整数）と型が明示されるので、「この列は数値のはずが文字列になっていた」といった事故に気づきやすいのが利点です。

---

### 2-1. CSVを読み込む

=== "pandas"

    ```python
    polymers = pd.read_csv("data/polymers.csv")
    ```

=== "Polars"

    ```python
    polymers = pl.read_csv("data/polymers.csv")
    ```

名前も引数もほぼ同じ。**ここは覚え直し不要**です。

---

### 2-2. 列の選択とフィルタ（条件抽出）

「引張強さが50 MPa以上のポリマーの名前と強さ」を取り出します。

=== "pandas"

    ```python
    result = polymers.loc[polymers["tensile_MPa"] >= 50, ["polymer", "tensile_MPa"]]
    print(result)
    ```

=== "Polars"

    ```python
    result = (
        polymers
        .filter(pl.col("tensile_MPa") >= 50)
        .select(["polymer", "tensile_MPa"])
    )
    print(result)
    ```

pandasの `df.loc[条件, 列リスト]` が、Polarsでは **`.filter(...)`（行を絞る）** と **`.select(...)`（列を選ぶ）** の2ステップに分かれます。役割が分かれているぶん、長い処理でも読みやすくなります。

Polarsの出力:

```text
shape: (6, 2)
┌──────────┬─────────────┐
│ polymer  ┆ tensile_MPa │
│ ---      ┆ ---         │
│ str      ┆ i64         │
╞══════════╪═════════════╡
│ PVC      ┆ 52          │
│ PET      ┆ 55          │
│ PMMA     ┆ 70          │
│ Nylon6   ┆ 80          │
│ Epoxy    ┆ 60          │
│ Phenolic ┆ 50          │
└──────────┴─────────────┘
```

---

### 2-3. 新しい列を計算して追加

「比強度 ＝ 引張強さ ÷ 密度」の列を作ります。

=== "pandas"

    ```python
    polymers["strength_density"] = (polymers["tensile_MPa"] / polymers["density"]).round(1)
    print(polymers[["polymer", "strength_density"]].head(3))
    ```

=== "Polars"

    ```python
    result = polymers.with_columns(
        (pl.col("tensile_MPa") / pl.col("density")).round(1).alias("strength_density")
    )
    print(result.select(["polymer", "strength_density"]).head(3))
    ```

pandasは `df["新列"] = ...` と **元のdfを書き換えます**。Polarsの `.with_columns()` は **元を変えず、追加後の新しい表を返す**のが原則です（意図しない上書き事故が起きにくい）。列名は `.alias("名前")` で付けます。

Polarsの出力:

```text
shape: (3, 2)
┌─────────┬──────────────────┐
│ polymer ┆ strength_density │
│ ---     ┆ ---              │
│ str     ┆ f64              │
╞═════════╪══════════════════╡
│ PE      ┆ 26.6             │
│ PP      ┆ 38.7             │
│ PS      ┆ 42.9             │
└─────────┴──────────────────┘
```

---

### 2-4. 並べ替え（sort）

「引張強さの大きい順に上位3件」。

=== "pandas"

    ```python
    top3 = polymers.sort_values("tensile_MPa", ascending=False).head(3)
    ```

=== "Polars"

    ```python
    top3 = polymers.sort("tensile_MPa", descending=True).head(3)
    ```

`sort_values(..., ascending=False)` → `sort(..., descending=True)`。名前がわずかに違うだけです。

---

### 2-5. グループ集計（groupby）

第36回の山場、「カテゴリごとに件数・平均」を出す集計テーブルです。

=== "pandas"

    ```python
    summary = polymers.groupby("category").agg(
        n            =("polymer",     "count"),
        tensile_mean =("tensile_MPa", "mean"),
        density_mean =("density",     "mean"),
    ).round(2)
    print(summary)
    ```

=== "Polars"

    ```python
    summary = polymers.group_by("category").agg(
        n            = pl.len(),
        tensile_mean = pl.col("tensile_MPa").mean().round(2),
        density_mean = pl.col("density").mean().round(2),
    ).sort("category")
    print(summary)
    ```

考え方は同じ。Polarsでは集計内容を **`pl.col("列").mean()` のような式** で書きます。件数は `pl.len()`。`group_by` の結果は順不同なので、表を安定させたいときは `.sort("category")` を付けます。

両者の結果は一致します。

=== "pandas の出力"

    ```text
                   n  tensile_mean  density_mean
    category
    thermoplastic  7         51.71          1.14
    thermoset      2         55.00          1.25
    ```

=== "Polars の出力"

    ```text
    shape: (2, 4)
    ┌───────────────┬─────┬──────────────┬──────────────┐
    │ category      ┆ n   ┆ tensile_mean ┆ density_mean │
    │ ---           ┆ --- ┆ ---          ┆ ---          │
    │ str           ┆ u32 ┆ f64          ┆ f64          │
    ╞═══════════════╪═════╪══════════════╪══════════════╡
    │ thermoplastic ┆ 7   ┆ 51.71        ┆ 1.14         │
    │ thermoset     ┆ 2   ┆ 55.0         ┆ 1.25         │
    └───────────────┴─────┴──────────────┴──────────────┘
    ```

---

### 2-6. 欠損値の扱い

酵素アッセイのデータには欠損（空欄）があります。

```python
enzyme = pl.DataFrame({
    "sample":        ["S1","S2","S3","S4","S5","S6"],
    "substrate_mM":  [0.5, 1.0, 2.0, 4.0, 8.0, 16.0],
    "absorbance":    [0.12, 0.21, 0.35, None, 0.58, 0.62],
    "temperature_C": [25.0, 25.0, None, 25.0, 25.0, 25.0],
})
```

=== "pandas"

    ```python
    # 欠損のある行を落とす
    enzyme.dropna()

    # absorbance の欠損を平均で埋める
    enzyme["absorbance"] = enzyme["absorbance"].fillna(enzyme["absorbance"].mean().round(3))
    ```

=== "Polars"

    ```python
    # 欠損のある行を落とす
    enzyme.drop_nulls()

    # absorbance の欠損を平均で埋める
    enzyme.with_columns(
        pl.col("absorbance").fill_null(pl.col("absorbance").mean().round(3))
    )
    ```

`dropna → drop_nulls`、`fillna → fill_null`。名前の対応さえ掴めば移行はスムーズです。Polarsは欠損を **`null`** と呼び、`0/0` のような **`NaN`** とは別物として扱う点だけ意識しておきましょう。

平均で埋めた結果（S4が平均 0.376 になる）:

```text
┌────────┬────────────┐
│ sample ┆ absorbance │
╞════════╪════════════╡
│ S1     ┆ 0.12       │
│ S2     ┆ 0.21       │
│ S3     ┆ 0.35       │
│ S4     ┆ 0.376      │
│ S5     ┆ 0.58       │
│ S6     ┆ 0.62       │
└────────┴────────────┘
```

---

## 3. Polarsならではの強み

対訳だけなら「名前が少し違うpandas」ですが、Polarsには **pandasにない武器** が2つあります。

### 3-1. 遅延評価（lazy）：クエリを最適化してから走らせる

`pl.read_csv` の代わりに **`pl.scan_csv`** や **`LazyFrame`** を使うと、処理を **すぐには実行せず**「設計図」として溜め込みます。最後に `.collect()` を呼んだ瞬間、Polarsが全体を見渡して **無駄を省いた最適な順序** で一気に実行します。

```python
result = (
    pl.scan_csv("data/polymers.csv")          # まだ読まない（設計図）
      .filter(pl.col("tensile_MPa") >= 50)     # 条件も設計図に足すだけ
      .select(["polymer", "tensile_MPa"])
      .collect()                               # ここで初めて実行
)
```

何が嬉しいか：Polarsは「**最終的に必要なのは2列＋この条件だけ**」と分かるので、**CSVから必要な列・行だけを読む**（=読み込み量そのものを減らす）といった最適化を自動でやります。数GBのログ的データで効果が絶大です。pandasは「まず全部読んでから絞る」ので、この差が速度とメモリに直結します。

### 3-2. 式（Expression）で複数列を一気に処理

`pl.col(...)` の式は使い回せます。たとえば「数値列すべてを標準化」のような処理が短く書けます。

```python
# 数値列すべてを (x - mean) / std で標準化
polymers.with_columns(
    (pl.col(pl.Float64, pl.Int64) - pl.col(pl.Float64, pl.Int64).mean())
    / pl.col(pl.Float64, pl.Int64).std()
)
```

「列を型でまとめて選ぶ」「同じ式を全列に適用」がpandasより素直に書けます。

---

## 4. pandas と Polars の相互変換

現場での結論は **「共存」** です。重い前処理だけPolarsで速く回し、可視化（seaborn）やRDKit連携はpandasに戻す、という使い方ができます。変換は一瞬です。

```python
# pandas → Polars
pf = pl.from_pandas(pdf)

# Polars → pandas（matplotlib / seaborn / scikit-learn へ渡すとき）
pdf = pf.to_pandas()
```

!!! tip "実務の型"
    「**巨大データの読み込み・結合・集計は Polars、図と機械学習は pandas**」。この橋渡しを覚えておけば、両方の良いとこ取りができます。

---

## 5. 使い分けの指針

| こんなとき | おすすめ | 理由 |
|---|---|---|
| 授業・レポートの数百〜数万行 | **pandas** | 速度差が体感できず、教材・情報量・周辺ツールが豊富 |
| RDKit / seaborn / scikit-learn と直結 | **pandas** | これらがpandasを前提にしている |
| クロマト・分光の大量ロウデータ（数百万行〜） | **Polars** | マルチスレッドで劇的に速い |
| ハイスループットスクリーニングの一括処理 | **Polars** | 省メモリ・lazyで巨大ファイルを捌ける |
| メモリが足りず pandas が落ちる | **Polars** | Arrowベースで消費が少ない |
| 型や欠損のバグを早く潰したい | **Polars** | 型が厳格・`null`/`NaN`を区別 |

ひとことで言うと、**「まずpandas。遅い・重いと感じたらPolars。仕上げはpandasに戻す」**。

---

## 6. Polarsの優れた点（まとめ）

| 観点 | Polars | pandas |
|---|---|---|
| 速度 | Rust製・マルチスレッドで数倍〜数十倍 | 基本1コア |
| メモリ | Apache Arrowで省メモリ | コピーが多く重くなりがち |
| 遅延評価 | `scan_csv`/`LazyFrame`でクエリ最適化 | 基本は逐次実行 |
| 記法の一貫性 | `pl.col(...)` の式で統一 | メソッドが混在しがち |
| 型・欠損 | 厳格、`null`と`NaN`を区別 | `NaN`混在で曖昧 |
| エコシステム | 発展中 | 圧倒的に成熟（本コースの土台） |

---

## 7. 移行時の注意（つまずきポイント）

!!! warning "pandasの癖のままだとハマる点"
    - **インデックスが無い**：Polarsに `df.index` はありません。行番号でなく **条件（filter）で選ぶ**のが基本。
    - **`.with_columns` は元を変えない**：pandasの `df["x"]=...` 感覚で「代入したのに変わらない」となりがち。**戻り値を受け取る**こと。
    - **`groupby` は順不同**：表を安定させたいなら `.sort()` を付ける。
    - **`null` と `NaN` は別物**：欠損は `null`。`fill_null` と `fill_nan` を取り違えない。
    - **メソッド名の違い**：`dropna→drop_nulls`、`fillna→fill_null`、`rename` の引数の渡し方など、細部が異なる。

---

## 8. まとめ

- Polarsは **「速くて省メモリなDataFrame」**。辞書やCSVからの作り方はpandasとほぼ同じ。
- 選択は `.filter()`＋`.select()`、列追加は `.with_columns(...).alias(...)`、集計は `pl.col("列").mean()` の式。
- 独自の強みは **lazy評価（クエリ最適化）** と **式による一括処理**。
- 現場は **共存**：重い処理はPolars、図と機械学習はpandas。`to_pandas()` / `from_pandas()` で橋渡し。
- 判断は **「まずpandas、遅い・重いと感じたらPolars」**。

!!! success "次の一歩"
    手元にある大きめのCSV（クロマトの生データなど）を、`pd.read_csv` と `pl.read_csv` で読み込んで所要時間を比べてみましょう。差が出ないなら、pandasのままで十分ということです。
