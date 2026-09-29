# 第52回　相関のヒートマップ

!!! abstract "この回のゴール"
    - 複数の量の関係を表す**相関係数**を求める
    - `corr()` で相関行列を作る
    - **ヒートマップ**で相関を色の濃淡として一望する
    - 所要時間の目安: 60分
    - 使うデータ：**高分子（ポリマー）**の物性

「密度が高いポリマーは引張強さも高い？」——複数の物性どうしの関係を、まとめて見たいときに使うのが**相関行列**と**ヒートマップ**です。

`lesson52.py` を作りましょう。

```python
import pandas as pd

polymers = pd.DataFrame({
    "polymer":     ["PE", "PP", "PS", "PVC", "PET", "PMMA", "Nylon6", "Epoxy", "Phenolic"],
    "density":     [0.94, 0.905, 1.05, 1.38, 1.38, 1.18, 1.14, 1.20, 1.30],
    "tensile_MPa": [25, 35, 45, 52, 55, 70, 80, 60, 50],
    "Tg_C":        [-110, -10, 100, 80, 76, 105, 50, 120, 180],
})
```

---

## 1. 相関行列を作る

**相関係数**は、2つの量が「一緒に増えるか」を −1〜+1 で表す数です。

- **+1 に近い** … 一方が増えると他方も増える（正の相関）
- **0 に近い** … 関係が薄い
- **−1 に近い** … 一方が増えると他方は減る（負の相関）

数値列だけの相関を一気に求めるのが `corr()` です。

```python
corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr().round(2)
print(corr)
```

出力:

```text
             density  tensile_MPa  Tg_C
density         1.00         0.49  0.68
tensile_MPa     0.49         1.00  0.56
Tg_C            0.68         0.56  1.00
```

対角線はすべて 1.00（自分自身との相関）。密度と Tg は 0.68 とやや強めの正の相関、と読めます。

!!! warning "データが少ないと、相関係数は1点で大きく変わる"
    このデータは9件しかありません。例えば、Tg が極端に低い PE（−110 °C）を除いて計算し直すと、tensile_MPa と Tg_C の相関は 0.56 から **0.16** に、density と Tg_C は 0.68 から 0.55 に下がります。件数が少ないときは、1点を除いて再計算して結果が安定しているかを確かめ、相関係数の数字を強く信じすぎないようにします（検定は第7部で扱います）。

---

## 2. ヒートマップで一望する

数字の行列も良いですが、**色**にすると関係が直感的に見えます。seaborn の `heatmap` を使います。

```python
import seaborn as sns
import matplotlib.pyplot as plt

corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr()

plt.figure(figsize=(5, 4))
sns.heatmap(corr, annot=True, cmap="coolwarm", vmin=-1, vmax=1, center=0)
plt.title("Correlation of Polymer Properties")
plt.tight_layout()
plt.savefig("heatmap.png", dpi=100)
plt.show()
```

![相関のヒートマップ](../images/lesson52_heatmap.png)

赤いほど正の相関、青いほど負の相関。`annot=True` で数値も重ねています。ひと目で「どの物性どうしが一緒に動く傾向があるか」が分かります。

!!! note "heatmap の引数"
    - `annot=True` … 各マスに数値を表示
    - `cmap="coolwarm"` … 色の配色（赤=高・青=低）
    - `vmin=-1, vmax=1, center=0` … 色スケールを −1〜+1・中心0に固定（相関の標準）

!!! warning "相関 ≠ 因果"
    相関があっても「原因と結果」とは限りません（たまたま一緒に動くだけのことも）。相関は「関係の手がかり」であり、因果の証明ではない——これは科学で最重要の注意点です。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「DataFrame `polymers` の数値列 `density`, `tensile_MPa`, `Tg_C` だけを選んで、Pearson の相関行列を計算し、小数第2位に丸めて print してください。
    続けて seaborn の `heatmap` で、`annot=True`, `cmap="coolwarm"`, `vmin=-1`, `vmax=1`, `center=0` として描いてください。
    データは9件だけなので、Tg が極端な PE を除いた場合の相関行列も print して比べてください。」

!!! warning "AIの出力で確かめること"
    - 文字列の列（`polymer` など）を含んだまま `df.corr()` していないか。pandas 2 以降では `ValueError: could not convert string to float: 'PE'` になります。列を選ぶか `numeric_only=True` を付ける。
    - `vmin=-1, vmax=1, center=0` があるか。ないと色の範囲がデータの最小〜最大に合わせられ、正の相関 0.49 が**青（負の相関の色）**に塗られてしまいます。
    - 相関行列が対称で対角がすべて 1 か。図の数値と print の値が一致するか。
    - 件数の少なさ：1点を除いて再計算し、相関係数が大きく変わらないか（PE を除くと tensile–Tg は 0.56 → 0.16）。
    - AI の説明文が「密度が高いと Tg が高くなる」のように**因果**として書いていないか。

---

## 演習問題

**問1.** `polymers` の数値3列（density, tensile_MPa, Tg_C）の相関行列を `corr()` で作り、小数第2位に丸めて表示してください。density と Tg_C の相関はいくつですか？

**問2.** その相関行列を `sns.heatmap`（`annot=True`, `cmap="coolwarm"`）でヒートマップにしてください。

**問3.** 配色を変えてみましょう。`cmap="viridis"` でヒートマップを描き、`coolwarm` との印象の違いを見てください（相関の可視化には赤青の `coolwarm` 系が向く理由も考えましょう）。

**問4.** 次は AI が書いた「ポリマー物性の相関ヒートマップ」のコードです（`polymers` は本文のもの）。実行するとエラーになります。また、エラーを直しても図の色づけに問題が残ります。2つの問題を指摘し、直してください。
```python
import seaborn as sns, matplotlib.pyplot as plt

corr = polymers.corr().round(2)
sns.heatmap(corr, annot=True, cmap="coolwarm")
plt.show()
```

出力（最後の行）:
```text
ValueError: could not convert string to float: 'PE'
```

---

## 解答

??? success "問1 の解答"
    ```python
    corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr().round(2)
    print(corr)
    ```

    出力:
    ```text
                 density  tensile_MPa  Tg_C
    density         1.00         0.49  0.68
    tensile_MPa     0.49         1.00  0.56
    Tg_C            0.68         0.56  1.00
    ```
    density と Tg_C の相関は **0.68**（やや強い正の相関）。

??? success "問2 の解答"
    ```python
    import seaborn as sns, matplotlib.pyplot as plt
    corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr()
    plt.figure(figsize=(5, 4))
    sns.heatmap(corr, annot=True, cmap="coolwarm", vmin=-1, vmax=1, center=0)
    plt.title("Correlation")
    plt.tight_layout()
    plt.show()
    ```

??? success "問3 の解答"
    ```python
    import seaborn as sns, matplotlib.pyplot as plt
    corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr()
    plt.figure(figsize=(5, 4))
    sns.heatmap(corr, annot=True, cmap="viridis")
    plt.title("Correlation (viridis)")
    plt.tight_layout()
    plt.show()
    ```
    `viridis` は連続量向き。相関は「正・負・ゼロ」に意味があるので、**中心0で赤青に分かれる `coolwarm`** のほうが、正負が直感的に読めます。

??? success "問4 の解答"
    **誤り1：文字列の列 `polymer` まで相関の計算に含めている。** pandas 2 以降の `corr()` は、数値でない列があるとエラーになります。数値の列だけを選ぶ（または `corr(numeric_only=True)`）ようにします。

    **誤り2：色の範囲を −1〜+1・中心0に固定していない。** `vmin`・`vmax`・`center` がないと、色の範囲がデータの最小値（0.49）〜最大値（1.00）に合わせられ、正の相関である 0.49 が青（負の相関に見える色）で塗られてしまいます。

    ```python
    import seaborn as sns, matplotlib.pyplot as plt

    corr = polymers[["density", "tensile_MPa", "Tg_C"]].corr().round(2)   # 数値列だけ
    print(corr)
    sns.heatmap(corr, annot=True, cmap="coolwarm", vmin=-1, vmax=1, center=0)
    plt.title("Correlation of Polymer Properties")
    plt.tight_layout()
    plt.show()
    ```

    出力:
    ```text
                 density  tensile_MPa  Tg_C
    density         1.00         0.49  0.68
    tensile_MPa     0.49         1.00  0.56
    Tg_C            0.68         0.56  1.00
    ```
    すべて正の相関なので、正しい図ではすべてのマスが赤系（白〜赤）になります。青いマスがあったら、色の範囲の設定を疑いましょう。

---

## この回のまとめ

- 相関係数は 2量の連動を −1〜+1 で表す。`df.corr()` で相関行列。
- `sns.heatmap(corr, annot=True, cmap="coolwarm", center=0)` で色の濃淡に。
- 相関の可視化は中心0の赤青系（coolwarm）が読みやすい。
- **相関は因果ではない**。関係の手がかりとして使う。

### 次回予告

[第53回：箱ひげ図・バイオリンプロット](lesson-53.md) では、カテゴリごとの「分布の違い」を比べる図を学びます。
