# 第102回　再現可能な研究：環境管理とバージョン管理

!!! abstract "この回のゴール"
    - 「再現可能な研究」がなぜ大切かを理解する
    - 環境（ライブラリ・バージョン）を記録・共有する
    - バージョン管理（Git）と乱数シードで再現性を担保する
    - 所要時間の目安: 60分

**再現性**——「誰が・いつ・どこで実行しても同じ結果になる」ことは、科学の根幹です。データ分析でも、この再現性を守る技術があります。この回は、これまで各所で触れた再現性を1つにまとめます。

---

## 1. 再現性を脅かすもの

- **環境の違い**：ライブラリのバージョンが違うと、結果が変わることがある。
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
    pip freeze > requirements.txt          # 使ったパッケージ一覧
    conda env export > environment.yml     # conda環境まるごと
    ```
    別の人は `pip install -r requirements.txt` で同じ環境を再現できます。

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

## 演習問題

**問1.** 自分の学習環境で `pip freeze > requirements.txt`（または `conda env export`）を実行し、環境ファイルを作ってください。中身にライブラリとバージョンが並ぶことを確認しましょう。

**問2.** 乱数を使うコード（例：`np.random.default_rng()` でのサンプリング）で、seed を固定した場合と固定しない場合で、2回実行して結果が変わるか比べてください。

**問3.** 再現可能なプロジェクトの構成（data / scripts / results / README など）を、自分の練習用フォルダで作ってみてください。README には「何のプロジェクトか・実行手順」を書きましょう。

---

## 解答

??? success "問1 の解答・確認ポイント"
    ```bash
    pip freeze > requirements.txt
    ```
    `numpy==2.x`、`pandas==2.x` のように「名前==バージョン」が並べば成功。このファイルがあれば、別PC・数年後でも同じ環境を再現できます。

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
    seed を固定した上2つは同じ、固定しない下2つは（ほぼ確実に）異なります。再現性には seed 固定が不可欠、と分かります。

??? success "問3 の解答・確認ポイント"
    `data/` `scripts/` `results/` フォルダと `README.md` を作ります。README に「目的・データの説明・実行手順（例：`python scripts/analysis.py`）」を書いておけば、他の人（未来の自分）が迷わず再現できます。

---

## この回のまとめ

- 再現性を脅かすのは、環境差・乱数・手作業・記録不足。
- 環境を記録：`requirements.txt` / `environment.yml`（Python）、`renv`（R）。
- Git でコード・データ・環境の履歴を残す。
- 乱数は seed 固定（`default_rng(42)` / `random_state=42` / `set.seed(42)`）。
- 「生データは触らない・結果は再生成できる」プロジェクト構成に。

### 次回予告

[第103回：AIを研究の相棒にする](lesson-103.md) では、AI（Claude / ChatGPT）を研究に活かす方法と、その正しい付き合い方を学びます。
