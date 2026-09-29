# 第102回　再現可能な研究：環境管理とバージョン管理

!!! abstract "この回のゴール"
    - 「再現可能な研究」がなぜ大切かを理解する
    - 環境（ライブラリ・バージョン）を記録・共有する
    - バージョン管理（Git）と乱数シードで再現性を担保する
    - 所要時間の目安: 60分

**再現性**——「誰が・いつ・どこで実行しても同じ結果になる」ことは、科学の根幹です。データ分析でも、この再現性を守る技術があります。この回は、これまで各所で触れた再現性を1つにまとめます。

---

## 1. 再現性を脅かすもの

- **環境の違い**：ライブラリのバージョンが違うと、結果が変わることがある。たとえば scikit-learn の `KMeans` は、1.4 で `n_init` の既定値が 10 から `"auto"` に変わりました。`n_init` を書かずにクラスタリングしたコードは、バージョンによって違う結果になりえます（第99回で `n_init=10` を明示したのはこのためです）。
- **乱数**：seed を固定しないと、実行のたびに結果が変わる（第26回・第95回）。
- **手作業**：手でコピペ・手で数値入力すると、ミスと非再現の温床（第86回）。
- **記録不足**：「どのデータで・どのコードで・どの手順で」が残っていない。

!!! warning "「昨日は動いたのに」を防ぐ"
    数か月後の自分や、共同研究者が同じ結果を再現できないのは、研究では致命的です。再現性の技術は、**未来の自分と仲間を守る**ものです。

---

## 2. 環境を記録・共有する

使ったライブラリとバージョンを記録します（第19回）。

=== "Python"

    ```bash
    pip freeze > requirements.txt                          # venv の場合：使ったパッケージ一覧
    conda env export --from-history > environment.yml      # conda の場合：自分で入れたパッケージだけ
    python --version > python-version.txt                  # Python 本体のバージョンも残す
    ```
    別の人は `pip install -r requirements.txt`（conda なら `conda env create -f environment.yml`）で同じ環境を作れます。`conda env export` をオプションなしで使うと、OS 固有のビルド番号まで書かれて Windows と Ubuntu の間で再現できないことがあるため、`--from-history` を付けます。conda 環境で `pip freeze` をすると `numpy @ file:///...` のような行が混ざることがあり、そのときは `pip list --format=freeze` を使います。

=== "R"

    ```r
    # renv パッケージでプロジェクトの環境を固定
    install.packages("renv")
    renv::init()        # プロジェクトの環境を記録し始める
    renv::snapshot()    # 現在のパッケージ状態を保存
    ```
    `renv` は R 版の環境管理で、`renv.lock` にバージョンを記録します。

これらのファイル（`requirements.txt` / `environment.yml` / `renv.lock`）を、コードと一緒に保管・共有します。

---

## 3. バージョン管理（Git）で履歴を残す

Git（第7回）で、**コードとデータの変更履歴**を残します。

```bash
git add analysis.py data.csv requirements.txt
git commit -m "反応収率の解析：初版"
```

- 「いつ・何を・なぜ変えたか」がすべて残る。
- 過去の任意の時点に戻れる。
- 「論文の図を作ったときのコード」を、コミットで特定できる。

!!! tip "環境ファイルも Git で管理"
    `requirements.txt` や `environment.yml` も Git に含めます。すると「このコミットのコードは、この環境で動く」がセットで残り、完全に再現できます。

---

## 4. 乱数シードを固定する

乱数を使う処理（データ分割・シャッフル・シミュレーション）では、**seed を必ず固定**します。

```python
import numpy as np
rng = np.random.default_rng(42)              # NumPy（第26回）

from sklearn.model_selection import train_test_split
train_test_split(X, y, random_state=42)      # scikit-learn（第95回）
```

```r
set.seed(42)      # R（乱数を使う前に）
```

seed を固定すれば、乱数を使っても毎回同じ結果になり、再現できます。

!!! warning "seed の落とし穴"
    - `np.random.seed(42)` は古い方式の乱数だけに効き、`np.random.default_rng()`（引数なし）には**効きません**。Generator には `default_rng(42)` のように直接 seed を渡します。
    - データ分割だけでなく、乱数を使う**モデル**（`RandomForestRegressor`・`KMeans` など）にも `random_state` を指定します。

---

## 5. 再現可能なプロジェクトの形

!!! success "おすすめのプロジェクト構成"
    ```text
    my-research/
      data/           … 生データ（変更しない）
      scripts/        … 解析コード
      results/        … 出力（図・表）
      report.qmd      … レポート
      requirements.txt … 環境
      README.md       … 概要・手順
      .git/           … バージョン管理
    ```
    **「生データは触らない・コードで加工・結果は再生成できる・全部記録する」**。これが再現可能な研究の基本形です。誰かがこのフォルダを受け取れば、同じ結果を再現できます。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    「収率データのブートストラップ解析（Python・NumPy）を、誰が実行しても同じ結果になるように書いてください。
    乱数は `np.random.default_rng(42)` の Generator だけを使い、scikit-learn の関数やモデルにはすべて `random_state=42` を渡してください。
    スクリプトの最初で Python・NumPy・pandas・scikit-learn のバージョンを表示し、`requirements.txt` と `README.md`（実行手順つき）の中身も提案してください。」

!!! warning "AIの出力で確かめること"
    - **seed が本当に効いているか**：同じスクリプトを2回実行して、出力が完全に一致するか。`np.random.seed(42)` の後に `default_rng()`（引数なし）を使っていたら seed は効いていない。
    - **乱数を使う全部の場所**：`train_test_split`・`KFold(shuffle=True)`・`KMeans`・`RandomForest` などに `random_state` があるか。1か所でも抜けると再現しない。
    - **既定値に頼っていないか**：`KMeans(n_clusters=3)` のように、バージョンで既定値が変わった引数（`n_init`）を省略していないか。
    - **環境ファイルの中身**：`requirements.txt` に `@ file:///` のようなローカルパスが入っていないか。conda なら `--from-history` を使っているか。Python 本体のバージョンが記録されているか。
    - **生データを上書きしていないか**：`data/` の CSV を `to_csv` で上書きするコードは危険。出力は `results/` へ。

---

## 演習問題

**問1.** 自分の学習環境で `pip freeze > requirements.txt`（または `conda env export`）を実行し、環境ファイルを作ってください。中身にライブラリとバージョンが並ぶことを確認しましょう。

**問2.** 乱数を使うコード（例：`np.random.default_rng()` でのサンプリング）で、seed を固定した場合と固定しない場合で、2回実行して結果が変わるか比べてください。

**問3.** 再現可能なプロジェクトの構成（data / scripts / results / README など）を、自分の練習用フォルダで作ってみてください。README には「何のプロジェクトか・実行手順」を書きましょう。

**問4.** 次は、収率の平均の 95 % 信頼区間をブートストラップ法で求めるために AI が書いたコードです。「seed を固定したので再現できます」とのことでしたが、3回実行すると 79.06〜81.47、79.01〜81.46、79.03〜81.49 と毎回変わりました。問題点を指摘し、直してください。

```python
import numpy as np

np.random.seed(42)                       # 再現性のため seed を固定
rng = np.random.default_rng()

yields = np.array([78.2, 81.5, 79.9, 83.1, 80.4, 77.6, 82.0, 79.3])   # 収率 [%]（架空）
boot = [rng.choice(yields, size=len(yields), replace=True).mean() for _ in range(2000)]
low, high = np.percentile(boot, [2.5, 97.5])
print(f"平均収率の95%信頼区間: {low:.2f}〜{high:.2f} %")
```

---

## 解答

??? success "問1 の解答・確認ポイント"
    ```bash
    pip freeze > requirements.txt
    ```
    `numpy==2.4.4`、`pandas==3.0.2` のように「名前==バージョン」が並べば成功。このファイルがあれば、別PCでも同じバージョンをそろえやすくなります。ただし Python 本体のバージョンや OS は記録されないので、`python --version` も一緒に残しましょう（数年たつと古いバージョンが新しい Python に入らないこともあります）。

??? success "問2 の解答"
    ```python
    import numpy as np
    # seed 固定：2回とも同じ
    print(np.random.default_rng(0).integers(0, 100, 3))
    print(np.random.default_rng(0).integers(0, 100, 3))
    # seed なし：毎回変わる
    print(np.random.default_rng().integers(0, 100, 3))
    print(np.random.default_rng().integers(0, 100, 3))
    ```
    出力（上2行。下2行は実行のたびに違う3つの整数になります）:
    ```text
    [85 63 51]
    [85 63 51]
    ```
    seed を固定した上2つは同じ、固定しない下2つは（ほぼ確実に）異なります。再現性には seed 固定が不可欠、と分かります。

??? success "問3 の解答・確認ポイント"
    `data/` `scripts/` `results/` フォルダと `README.md` を作ります。README に「目的・データの説明・実行手順（例：`python scripts/analysis.py`）」を書いておけば、他の人（未来の自分）が迷わず再現できます。

??? success "問4 の解答"
    **誤り**：`np.random.seed(42)` は古い方式の乱数（`np.random.rand` など）の seed で、`np.random.default_rng()` で作る新しい Generator には影響しません。引数なしの `default_rng()` は毎回ちがう seed で始まるので、結果が実行ごとに変わります。

    確かめ方：同じスクリプトを2回実行し、出力が一致するかを見ます（「seed を書いた」ことではなく「結果が一致した」ことで確認する）。

    ```python
    import numpy as np

    rng = np.random.default_rng(42)          # Generator には seed を直接渡す

    yields = np.array([78.2, 81.5, 79.9, 83.1, 80.4, 77.6, 82.0, 79.3])   # 収率 [%]（架空）
    boot = [rng.choice(yields, size=len(yields), replace=True).mean() for _ in range(2000)]
    low, high = np.percentile(boot, [2.5, 97.5])
    print(f"平均収率の95%信頼区間: {low:.2f}〜{high:.2f} %")
    ```

    出力（何度実行しても同じ）:
    ```text
    平均収率の95%信頼区間: 79.02〜81.48 %
    ```

---

## この回のまとめ

- 再現性を脅かすのは、環境差・乱数・手作業・記録不足。
- 環境を記録：`requirements.txt` / `environment.yml`（Python）、`renv`（R）。
- Git でコード・データ・環境の履歴を残す。
- 乱数は seed 固定（`default_rng(42)` / `random_state=42` / `set.seed(42)`）。
- 「生データは触らない・結果は再生成できる」プロジェクト構成に。

### 次回予告

[第103回：AIを研究の相棒にする](lesson-103.md) では、AI（Claude / ChatGPT）を研究に活かす方法と、その正しい付き合い方を学びます。
