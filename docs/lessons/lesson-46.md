# 第46回　matplotlib入門：はじめてのグラフ

!!! abstract "この回のゴール"
    - グラフ描画ライブラリ **matplotlib** を使う
    - 折れ線グラフを描き、軸ラベル・タイトルをつける
    - グラフをファイルに保存する
    - 所要時間の目安: 60分
    - 使うデータ：**溶解度**（温度と溶解度の関係）

!!! info "第5部スタート：可視化"
    第4部で「集計・テーブル」ができました。第5部では、それを**グラフ**にします。数字の羅列より、1枚の図のほうが桁違いに伝わります。

!!! note "グラフのラベルは英語で書きます"
    matplotlib は初期設定だと日本語が文字化け（□□□）します。本コースでは**軸やタイトルは英語**で書きます（英語表記は論文でも標準）。日本語を使いたい場合は `japanize-matplotlib` などの追加設定が必要です（発展）。

`lesson46.py` を作りましょう。

---

## 1. はじめての折れ線グラフ

`import matplotlib.pyplot as plt` と略すのが慣例です。`plt.plot(x, y)` で線を引き、`plt.show()` で表示します。

```python
import matplotlib.pyplot as plt

# 硝酸カリウム KNO3 の溶解度（温度ごと）
temp = [0, 20, 40, 60, 80, 100]            # 温度 [℃]
solubility = [13, 32, 64, 110, 169, 246]   # 溶解度 [g / 100 g 水]

plt.plot(temp, solubility)
plt.show()
```

これだけで折れ線グラフが表示されます。でも、これでは「何のグラフか」が分かりません。装飾を足しましょう。

---

## 2. ラベル・タイトル・目印をつける

```python
import matplotlib.pyplot as plt

temp = [0, 20, 40, 60, 80, 100]
solubility = [13, 32, 64, 110, 169, 246]

plt.figure(figsize=(6, 4))                       # 図の大きさ（横6・縦4インチ）
plt.plot(temp, solubility, marker="o", color="teal")  # 点(marker)つき・色指定
plt.xlabel("Temperature (C)")                    # x軸ラベル
plt.ylabel("Solubility (g / 100 g water)")       # y軸ラベル
plt.title("Solubility of KNO3 vs Temperature")   # タイトル
plt.grid(True, alpha=0.3)                         # 薄い目盛り線
plt.tight_layout()                                # レイアウト自動調整
plt.savefig("solubility.png", dpi=100)            # 画像として保存
plt.show()                                        # 画面に表示
```

生成される図:

![溶解度と温度の折れ線グラフ](../images/lesson46_solubility.png)

温度が上がると溶解度が急に増える様子が、ひと目で分かります。表の数字だけでは気づきにくい「曲がり方」も、グラフなら直感的です。

!!! note "主なパーツ"
    - `plt.figure(figsize=(横, 縦))` … 図のサイズ
    - `marker="o"` … 各データ点に丸印。`"s"`（四角）`"^"`（三角）なども
    - `color="teal"` … 線の色（`"red"` `"navy"` や `"#4c72b0"` などでも）
    - `plt.savefig("名前.png", dpi=100)` … 画像保存（dpi は解像度）

!!! warning "savefig は show より前に"
    `plt.savefig()` は `plt.show()` の**前**に書きます。show の後だと図がクリアされ、空の画像が保存されることがあります。

---

## 演習問題

**問1.** 本文のコードを実行し、`solubility.png` が保存され、折れ線グラフが表示されることを確認してください。

**問2.** 別の物質のデータでグラフを描いてみましょう。塩化ナトリウム NaCl の溶解度はほぼ一定です。`temp = [0, 20, 40, 60, 80, 100]`、`solubility = [35.7, 36.0, 36.6, 37.3, 38.4, 39.8]` で折れ線グラフを描き、軸ラベルとタイトルをつけてください。KNO3 と比べて、線の形はどう違いますか？

**問3.** 問2のグラフの線の色を変え（例：`color="crimson"`）、マーカーを四角（`marker="s"`）にして、`nacl.png` という名前で保存してください。

---

## 解答

??? success "問1 の解答・確認ポイント"
    本文のコードをそのまま実行します。ウィンドウにグラフが出て、スクリプトと同じフォルダに `solubility.png` ができていれば成功です。表示されない場合は、`plt.show()` を書いたか確認しましょう。

??? success "問2 の解答"
    ```python
    import matplotlib.pyplot as plt

    temp = [0, 20, 40, 60, 80, 100]
    solubility = [35.7, 36.0, 36.6, 37.3, 38.4, 39.8]

    plt.figure(figsize=(6, 4))
    plt.plot(temp, solubility, marker="o", color="teal")
    plt.xlabel("Temperature (C)")
    plt.ylabel("Solubility (g / 100 g water)")
    plt.title("Solubility of NaCl vs Temperature")
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.show()
    ```
    NaCl はほぼ横ばい（水平に近い直線）。KNO3 の急な右肩上がりとは対照的で、「温度による溶解度の変化」が物質で大きく違うことが、グラフだと一目瞭然です。

??? success "問3 の解答"
    ```python
    plt.figure(figsize=(6, 4))
    plt.plot(temp, solubility, marker="s", color="crimson")
    plt.xlabel("Temperature (C)")
    plt.ylabel("Solubility (g / 100 g water)")
    plt.title("Solubility of NaCl")
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig("nacl.png", dpi=100)
    plt.show()
    ```

---

## この回のまとめ

- `import matplotlib.pyplot as plt`。`plt.plot(x, y)` で折れ線、`plt.show()` で表示。
- `xlabel` / `ylabel` / `title` / `grid` で読めるグラフに。
- `marker` `color` `figsize` で見た目を調整。
- `plt.savefig("名前.png", dpi=100)` で保存（show の前に）。
- ラベルは英語で（日本語は文字化けするため）。

### 次回予告

[第47回：折れ線・散布図・棒グラフ](lesson-47.md) では、目的別に使い分ける3つの基本グラフを学びます。
