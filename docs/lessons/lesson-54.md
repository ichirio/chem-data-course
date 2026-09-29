# 第54回　論文品質の図をつくる（解像度・フォント）

!!! abstract "この回のゴール"
    - レポート・論文に載せられる図の要件を知る
    - **解像度（dpi）**・フォントサイズ・余白を整える
    - 不要な枠を消してすっきり見せる
    - 図をファイルに正しく書き出す
    - 所要時間の目安: 60分
    - 使うデータ：**検量線**（濃度と吸光度）

同じデータでも、体裁ひとつで「実験メモ」にも「論文の図」にも見えます。今回は仕上げの技術です。

`lesson54.py` を作りましょう。

---

## 1. 論文品質の図の要件

| 要素 | 目安 |
|---|---|
| 解像度 | **300 dpi 以上**（印刷でぼやけない） |
| フォント | 軸ラベル・タイトルが十分大きい（11〜13pt程度） |
| 軸ラベル | 量と**単位**を必ず明記（例：Concentration (mM)） |
| 体裁 | 不要な枠線を消す、凡例は枠なし、色は控えめ |
| 形式 | 印刷は PNG(高dpi) か PDF/SVG（ベクター） |

---

## 2. 仕上げたコード

これまでの検量線を、論文品質に整えます。`fig, ax = plt.subplots()` という書き方（オブジェクト指向スタイル）を使うと、細かい調整がしやすくなります。

```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="ticks")          # すっきりしたテーマ

conc = np.array([0, 2, 4, 6, 8, 10])
absorbance = np.array([0.02, 0.21, 0.40, 0.59, 0.80, 0.99])
slope, intercept = np.polyfit(conc, absorbance, 1)

fig, ax = plt.subplots(figsize=(5, 4))
ax.scatter(conc, absorbance, color="black", zorder=3, label="data")
ax.plot(conc, slope * conc + intercept, color="crimson",
        label=f"y = {slope:.3f}x + {intercept:.3f}")

ax.set_xlabel("Concentration (mM)", fontsize=12)
ax.set_ylabel("Absorbance", fontsize=12)
ax.set_title("Calibration Curve", fontsize=13)
ax.legend(frameon=False)              # 凡例の枠を消す
sns.despine()                         # 上と右の枠線を消す

fig.tight_layout()
fig.savefig("figure_publication.png", dpi=300)   # 高解像度で保存
fig.savefig("figure_publication.pdf")            # ベクター形式でも
plt.show()
```

![論文品質の検量線](../images/lesson54_publication.png)

上と右の枠が消え、凡例の枠もなく、必要な情報（軸ラベル＋単位、近似式）が過不足なく載っています。figsize=(5, 4) インチを 300 dpi で保存したので、画像は 1500×1200 ピクセルあり、印刷しても十分なめらかです（PNG は画素の集まりなので、極端に拡大すればいずれはギザギザになります。拡大に強いのは PDF/SVG です）。

!!! note "`fig, ax` スタイルについて"
    `fig, ax = plt.subplots()` で、図全体(`fig`)と1枚の軸(`ax`)を取り出します。以降 `ax.set_xlabel(...)` のように `ax` に対して指定します。複数の図を細かく制御でき、実務ではこちらが主流です（これまでの `plt.xlabel()` 方式と結果は同じ）。

---

## 3. 保存形式の選び方

- **PNG（dpi=300）** … 手軽で万能。スライドやWordに貼るならこれ。
- **PDF / SVG** … ベクター形式。**いくら拡大してもぼやけない**。論文の投稿で好まれます。
- **dpi を上げる** … 保存時 `dpi=300` を忘れない。画面表示は粗くても、保存が高解像度なら印刷はきれいです。
- **figsize は掲載サイズに合わせる** … 論文の1段組の幅は約 8.5 cm（3.3 インチ）程度です。幅 5 インチの図を 3.3 インチに縮めて載せると、12 pt の文字は約 8 pt に小さくなります。最初から掲載サイズで作ると、フォントサイズがそのまま紙面の大きさになります。

!!! tip "日本語を使いたいとき（発展）"
    matplotlib の既定フォント（DejaVu Sans）には日本語の文字がないため、そのままでは □□□ に化け、`Glyph ... missing from font(s) DejaVu Sans.` という警告が出ます。OS に入っている日本語フォントを、スクリプトの最初で指定します。

    ```python
    import matplotlib.pyplot as plt

    plt.rcParams["font.family"] = "sans-serif"
    plt.rcParams["font.sans-serif"] = [
        "Yu Gothic", "Meiryo",              # Windows
        "Hiragino Sans",                    # macOS
        "Noto Sans CJK JP", "IPAexGothic",  # Ubuntu
        "DejaVu Sans",
    ]
    plt.rcParams["pdf.fonttype"] = 42       # PDF にフォントを埋め込む（投稿先での文字化け防止）
    ```

    - リストの先頭から順に、見つかったフォントが使われます。Windows 10/11 には游ゴシック（Yu Gothic）とメイリオが標準で入っています。
    - Ubuntu では `sudo apt install fonts-noto-cjk` で Noto Sans CJK JP を入れます。入れた後も □□□ のままなら、matplotlib のフォントキャッシュのフォルダ（`python -c "import matplotlib; print(matplotlib.get_cachedir())"` で表示される場所）を削除してから再実行します。
    - `japanize-matplotlib` というパッケージも紹介されることがありますが、更新が止まっており、Python 3.12 以降の venv では `import japanize_matplotlib` が `ModuleNotFoundError: No module named 'distutils'` で失敗します。上の方法を使いましょう。

    本コースは英語ラベルで進めますが、卒論などで必要になったら試してください。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「検量線（濃度 [mM] と吸光度 [無次元] の NumPy 配列）と近似直線の図を、論文用に仕上げてください。
    `fig, ax = plt.subplots(figsize=(3.3, 2.6))`（1段組の幅）で描き、軸ラベルは英語で単位つき、フォントは 8〜10 pt、上と右の枠を消し、凡例は枠なしにしてください。
    `figure1.png`（dpi=300）と `figure1.pdf` の両方で保存し、保存した PNG のピクセル数を print してください。」

!!! warning "AIの出力で確かめること"
    - `savefig` に `dpi=300` があるか。PIL の `Image.open("figure1.png").size` で画素数を確かめる。`figsize=(5, 4)` なら 300 dpi で 1500×1200 px、500×400 px なら dpi 指定が抜けています（既定は 100 dpi）。
    - 軸ラベルに単位があるか（`Concentration (mM)`）。吸光度は無次元なので単位なしが正しく、`Absorbance (mM)` のような誤った単位が付いていないか。
    - figsize が掲載サイズに合っているか。大きく作って縮小掲載すると、文字が指定より小さくなります。
    - `sns.set_theme()` はフォントサイズの既定も変えます。`fontsize=` の個別指定と組み合わせたとき、軸ラベル・目盛り・凡例の文字の大きさがちぐはぐになっていないか。
    - 日本語を使った図を PDF 保存する場合、`plt.rcParams["pdf.fonttype"] = 42` が設定されているか。

---

## 演習問題

**問1.** 本文のコードを実行し、`figure_publication.png`（300 dpi）と `figure_publication.pdf` の両方が保存されることを確認してください。PNG を開いて拡大し、ぼやけないことを見ましょう。

**問2.** `sns.despine()` の行を消して実行し、上・右の枠線があるとどう印象が変わるか比べてください。

**問3.** 第47回で作った「触媒別収率の棒グラフ」を、`fontsize` を大きく・`sns.despine()` で枠を消し・`dpi=300` で保存して、論文品質に仕上げてください。

**問4.** 次は AI が「論文用に仕上げた」と言って出してきた検量線の図のコードです。論文品質の要件（本文の表）に照らして、問題点を2つ指摘し、直してください。
```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from PIL import Image

sns.set_theme(style="ticks")
conc = np.array([0, 2, 4, 6, 8, 10])
absorbance = np.array([0.02, 0.21, 0.40, 0.59, 0.80, 0.99])

fig, ax = plt.subplots(figsize=(5, 4))
ax.scatter(conc, absorbance, color="black")
ax.set_xlabel("Concentration", fontsize=12)
ax.set_ylabel("Absorbance", fontsize=12)
sns.despine()
fig.tight_layout()
fig.savefig("figure1.png")

img = Image.open("figure1.png")
print(f"画像サイズ: {img.size[0]} x {img.size[1]} px, dpi = {round(img.info['dpi'][0])}")
```

出力:
```text
画像サイズ: 500 x 400 px, dpi = 100
```

---

## 解答

??? success "問1 の解答・確認ポイント"
    本文のコードをそのまま実行。同じフォルダに `.png` と `.pdf` ができます。PNG の画像サイズ（画像ビューアのプロパティなど）が 1500×1200 ピクセルになっていれば 300 dpi で保存できています。PDF のほうは、いくら拡大しても線や文字がなめらかなことを確かめましょう。

??? success "問2 の解答・確認ポイント"
    `sns.despine()` を消すと、グラフの四辺すべてに枠線が付きます。上と右の線は情報を持たないため、消したほうが**すっきりしてデータに目が向く**——というのが論文図の定石です。

??? success "問3 の解答"
    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns
    sns.set_theme(style="ticks")

    catalysts = ["Pd", "Pt", "Ni"]
    yield_pct = [82, 76, 62]

    fig, ax = plt.subplots(figsize=(5, 4))
    ax.bar(catalysts, yield_pct, color="#4c72b0")
    ax.set_xlabel("Catalyst", fontsize=12)
    ax.set_ylabel("Yield (%)", fontsize=12)
    ax.set_title("Yield by Catalyst", fontsize=13)
    ax.set_ylim(0, 100)
    sns.despine()
    fig.tight_layout()
    fig.savefig("yield_pub.png", dpi=300)
    plt.show()
    ```

??? success "問4 の解答"
    **誤り1：解像度が指定されていない。** `savefig` に `dpi` を書かないと既定の 100 dpi で保存され、5×4 インチの図は 500×400 ピクセルしかありません。出力の画像サイズと dpi から分かります。

    **誤り2：x 軸ラベルに単位がない。** 「Concentration」だけでは mM なのか mol/L なのか分かりません。吸光度は無次元なので、y 軸は単位なしで正しいです。

    ```python
    import numpy as np
    import matplotlib.pyplot as plt
    import seaborn as sns
    from PIL import Image

    sns.set_theme(style="ticks")
    conc = np.array([0, 2, 4, 6, 8, 10])
    absorbance = np.array([0.02, 0.21, 0.40, 0.59, 0.80, 0.99])

    fig, ax = plt.subplots(figsize=(5, 4))
    ax.scatter(conc, absorbance, color="black")
    ax.set_xlabel("Concentration (mM)", fontsize=12)     # 単位を明記
    ax.set_ylabel("Absorbance", fontsize=12)
    sns.despine()
    fig.tight_layout()
    fig.savefig("figure1.png", dpi=300)                   # 解像度を指定
    fig.savefig("figure1.pdf")                            # ベクター形式も

    img = Image.open("figure1.png")
    print(f"画像サイズ: {img.size[0]} x {img.size[1]} px, dpi = {round(img.info['dpi'][0])}")
    ```

    出力:
    ```text
    画像サイズ: 1500 x 1200 px, dpi = 300
    ```
    「300 dpi で保存したつもり」を、画素数という数字で確かめるのがポイントです（`PIL` は matplotlib と一緒にインストールされる画像ライブラリ Pillow です）。

---

## この回のまとめ

- 論文図は **300 dpi 以上**・十分なフォント・**単位つき軸ラベル**・すっきりした体裁。
- `fig, ax = plt.subplots()` のオブジェクト指向スタイルが細かい制御に便利。
- `sns.despine()` で上・右の枠を消す、`legend(frameon=False)` で凡例の枠を消す。
- 保存は PNG(dpi=300) か PDF/SVG。日本語は追加フォント設定で対応。

### 次回予告

[第55回：まとめ演習（反応速度データの可視化）](lesson-55.md) では、第5部の総仕上げとして、化学反応の時間変化データをグラフで解析します。
