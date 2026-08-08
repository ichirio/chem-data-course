# 第19回　仮想環境とパッケージ管理（conda / pip）

!!! abstract "この回のゴール"
    - なぜプロジェクトごとに環境を分けるのかを理解する
    - conda 環境／venv の作成・切り替えを整理する
    - `requirements.txt` / `environment.yml` で環境を再現・共有する
    - 所要時間の目安: 60分

第1回で `chem` 環境を作りました。今回はその仕組みを一歩深く理解し、**環境を再現・共有できる**ようにします。研究の再現性に直結する大事な回です。

---

## 1. なぜ環境を分けるのか

1台のパソコンで複数のプロジェクトを進めると、必要なライブラリのバージョンが衝突することがあります。

!!! example "衝突の例"
    - プロジェクトA は「pandas 1.5」で動く古いコード
    - プロジェクトB は「pandas 2.2」の新機能を使う

    同じパソコンに1つしか入れられないと、どちらかが壊れます。**プロジェクトごとに独立した環境**を作れば、それぞれに好きなバージョンを入れられ、互いに干渉しません。

---

## 2. 2つの仕組み（復習と整理）

第1回・第11回で触れた2つを整理します。

=== "conda 環境（このコースの主軸）"

    ```bash
    conda create -n myproject python=3.12 -y   # 作る
    conda activate myproject                    # 入る（先頭が (myproject) に）
    conda install -c conda-forge pandas -y      # 入れる
    conda deactivate                            # 出る
    conda env list                              # 環境の一覧を見る
    ```

    化学ライブラリや R も扱えるのが強み（第1回参照）。

=== "venv（Python標準）"

    ```bash
    python -m venv myenv          # 作る（myenv フォルダができる）
    # 入る:
    #   Windows: myenv\Scripts\Activate.ps1
    #   Ubuntu : source myenv/bin/activate
    pip install pandas            # 入れる
    deactivate                    # 出る
    ```

    追加インストール不要で手軽。Python だけで完結する用途に。

!!! tip "毎回のはじめに（再確認）"
    作業前に必ず環境に「入る」（先頭が `(chem)` などになる）。これを忘れて `ModuleNotFoundError` になるのは"あるある"です。

---

## 3. 環境を再現・共有する（超重要）

「自分のパソコンでは動くのに、相手のパソコンでは動かない」を防ぐには、**使ったライブラリの一覧を書き出して共有**します。

### pip の場合：requirements.txt

```bash
pip freeze > requirements.txt        # 今の環境の中身を書き出す
```

`requirements.txt` の中身（例）:

```text
numpy==2.1.0
pandas==2.2.3
matplotlib==3.10.1
```

別のパソコンでは、この1ファイルから一括で同じ環境を作れます。

```bash
pip install -r requirements.txt      # まとめて入れる
```

### conda の場合：environment.yml

```bash
conda env export > environment.yml           # 書き出す（チャンネルやバージョンも含む）
conda env create -f environment.yml          # 別PCで同じ環境を作る
```

!!! success "再現性は研究の生命線"
    「いつ・どのバージョンで・何を使ったか」を記録し、共有できること——これは研究の**再現性**そのものです。`requirements.txt` や `environment.yml` を Git（第7回）で一緒に管理すれば、コードと環境の両方を残せます。実際、このコースのリポジトリにも `requirements.txt` が入っています。

---

## 4. conda と pip の使い分け（第1回の再確認）

| | conda | pip |
|---|---|---|
| 得意 | 化学・科学系（RDKit 等）、R も | Python製パッケージ全般（世界標準） |
| 方針 | 科学系は conda で | conda に無いものは pip で |

!!! warning "同じものを両方で入れない"
    1つの環境で、同じパッケージを conda と pip の両方から入れると競合の原因になります。「科学系は conda、無いものだけ pip」と役割を分けましょう（第1回のルール）。

---

## 演習問題

**問1.** `conda env list`（venv の人は環境フォルダの確認）で、今どんな環境があるかを表示してください。学習用の `chem` 環境が一覧にありますか？

**問2.** 学習用の環境に入った状態で `pip freeze > requirements.txt` を実行し、できたファイルを開いて中身（入っているライブラリとバージョン）を確認してください。

**問3.** 練習用に新しい環境 `test-env` を作り、そこに `pip install requests`（軽いライブラリ）だけを入れ、`pip list` で「学習用環境とは中身が違う（独立している）」ことを確認してください。確認後は不要なら削除して構いません（`conda remove -n test-env --all` など）。

---

## 解答

??? success "問1 の解答・確認ポイント"
    ```bash
    conda env list
    ```
    `base` と、第1回で作った `chem` が一覧に出れば成功です（`*` が今いる環境）。venv の人は、作った環境フォルダ（例 `chem-env`）が存在するかを確認します。

??? success "問2 の解答・確認ポイント"
    ```bash
    conda activate chem        # 環境に入る（venvなら各自のactivate）
    pip freeze > requirements.txt
    ```
    できた `requirements.txt` に `pandas==2.2.x` や `matplotlib==3.10.x` のように「名前==バージョン」が並んでいれば成功です。このファイルがあれば、別PCで同じ環境を再現できます。

??? success "問3 の解答・確認ポイント"
    ```bash
    conda create -n test-env python=3.12 -y
    conda activate test-env
    pip install requests
    pip list                   # requests はあるが pandas 等は無い＝独立している
    ```
    `chem` 環境には pandas があり、`test-env` には無い——この違いが「環境が独立している」証拠です。学び終えたら `conda deactivate` して `conda remove -n test-env --all` で削除できます。

---

## この回のまとめ

- プロジェクトごとに環境を分けると、バージョン衝突を防げる。
- conda 環境（主軸）／venv（標準）、どちらも「作る・入る・入れる・出る」。
- **再現・共有**：pip は `requirements.txt`、conda は `environment.yml`。
- 再現性は研究の生命線。環境ファイルも Git で管理する。

### 次回予告

[第20回：まとめ演習](lesson-20.md) では、第2部で学んだこと（ファイル・クラス・内包表記・関数）を全部使って、実験データを整理するツールを作ります。第2部の総仕上げです。
