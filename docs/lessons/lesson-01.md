# 第1回　開発環境の構築

!!! abstract "この回のゴール"
    - 自分のパソコン（Ubuntu または Windows 11）に、学習に必要な道具をそろえる
    - Python が動くことを確認する
    - コードエディタ（VS Code）と、AIの相棒、Git を使えるようにする
    - 所要時間の目安: 60〜90分（最初だけ少し長めです。1回で終わらなくても大丈夫）

## この回で入れるもの

| 道具 | 役割 | なぜこれ？ |
|---|---|---|
| **Miniforge** | Python本体と科学ライブラリの管理 | 化学で使う RDKit などを、無料でトラブルなく入れられる |
| **VS Code** | コードを書くエディタ | 無料・定番・AI拡張が最も充実 |
| **Git** | 変更履歴の管理・共有 | 研究の再現性とポートフォリオの土台 |
| **R**（任意・後半で本格利用） | 統計 | 統計の回で使います。今回は入れるだけ |

!!! note "なぜ Anaconda ではなく Miniforge？"
    Miniforge は Anaconda と同じ「conda」の仕組みを使いますが、**完全無料で利用条件の制約がなく**、化学ライブラリの入手元（conda-forge）が最初から設定されています。軽くて安心です。

---

## ステップ0　すでに入っているか確認する（済みならスキップ）

インストールを始める前に、**もう入っているものは飛ばして構いません**。ターミナル（Windows は「Miniforge Prompt」か「PowerShell」、Ubuntu は端末）で次を打って確認します。

```bash
python --version     # 例: Python 3.12.3  → 出れば Python は導入済み
conda --version      # 例: conda 24.x     → 出れば Miniforge/conda 導入済み
R --version          # 先頭に R version … → 出れば R は導入済み
git --version        # 例: git version 2.x → 出れば Git は導入済み
code --version       # 数字が出れば VS Code 導入済み
```

| 表示された | 出なかった（`command not found` 等） |
|---|---|
| そのツールのインストールは**スキップ**してOK | 該当ステップに従って入れる |

!!! warning "「Python は入っているが conda は入っていない」場合"
    パソコンにすでに Python があっても、**このコースでは `chem` という専用環境を1つ作って学ぶ**ことをおすすめします（他の作業と混ざらず、化学ライブラリの導入も安定するため）。
    その方法は2通りあります。どちらか片方でOKです。

    - **A. Miniforge/conda を使う（推奨）**… 化学ライブラリ（RDKit 等）が一番安定。ステップ1へ。
    - **B. すでにある Python の `venv` を使う（conda を増やしたくない人向け）**… ステップ2の「venv 版」を参照。

---

## ステップ1　Miniforge を入れる（＝ Python 本体が入る）

!!! info "conda が既にある人はスキップ"
    ステップ0で `conda --version` が表示された人は、このステップは不要です。ステップ2へ進んでください。

Miniforge を入れると、**Python 本体も一緒に入ります**（別途 Python を入れる必要はありません）。

=== "Windows 11"

    1. ブラウザで [Miniforge の配布ページ](https://github.com/conda-forge/miniforge/releases/latest) を開く。
    2. `Miniforge3-Windows-x86_64.exe` をダウンロードして実行する。
    3. インストーラでは基本そのまま「Next」で進めてOK（インストール先は初期値のままで問題ありません）。
    4. 完了したら、スタートメニューから **「Miniforge Prompt」** を開く。以降のコマンドはこの黒い画面（ターミナル）に入力します。

=== "Ubuntu"

    ターミナル（`Ctrl`+`Alt`+`T`）を開いて、次を順に実行します。

    ```bash
    # インストーラをダウンロード
    wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"
    # 実行（画面の指示に従い、最後の「初期化しますか？」は yes）
    bash Miniforge3-Linux-x86_64.sh
    ```

    終わったらターミナルを一度閉じて開き直します。行の先頭に `(base)` と表示されれば成功です。

### 動作確認

ターミナル（Miniforge Prompt / Ubuntu のターミナル）で次を実行します。

```bash
conda --version
python --version
```

`conda 24.x` や `Python 3.12.x` のようにバージョンが表示されれば成功です。

---

## ステップ2　学習用の環境をつくる

パソコンには色々な作業が混ざります。**プロジェクト専用の「環境」を1つ作って隔離**しておくと、後でトラブりません。`chem` という名前の環境を作ります。方法は2通り、**どちらか片方でOK**です。

=== "A. conda で作る（推奨）"

    化学ライブラリ（RDKit 等）が一番安定します。迷ったらこちら。

    ```bash
    # chem という名前の環境を作る（Python 3.12 を指定）
    conda create -n chem python=3.12 -y

    # 作った環境に「入る」（これ以降、行の先頭が (chem) になる）
    conda activate chem
    ```

=== "B. venv で作る（すでにPythonがあり、condaを増やしたくない人）"

    OSに入っている Python をそのまま使い、標準機能の `venv` で環境を分けます。

    ```bash
    # 好きな作業フォルダの中で実行（例：chem-course フォルダ）
    python -m venv chem-env          # chem-env という環境フォルダを作る

    # 環境に「入る」
    # Windows (PowerShell):
    chem-env\Scripts\Activate.ps1
    # Ubuntu / macOS:
    source chem-env/bin/activate
    ```

    入ると行の先頭に `(chem-env)` と表示されます。

!!! tip "毎回のはじめに（超重要）"
    作業を始めるたびに、環境に「入る」操作が必要です。
    先頭が `(chem)`（またはvenvなら `(chem-env)`）になっているか、毎回確認してください。
    なっていなければ conda は `conda activate chem`、venv は上の activate コマンドを実行します。

---

## ステップ2.5　パッケージの入れ方（conda と pip）

Python の便利な機能は「パッケージ（ライブラリ）」として配られています。それを入れる道具が **conda** と **pip** の2つです。ここは一生使う知識なので、基本だけ押さえましょう。

### 今の主流と、このコースが Miniforge(conda) を選ぶ理由

!!! question "pip と conda、どっちが正解？"
    結論から言うと「**分野で棲み分け**」です。どちらが勝ち、ではありません。

    - **一般のPython開発**（Webやアプリ、自動化）… **pip＋venv が世界の基盤**。近年は高速な **uv** が新しい定番になりつつあります。
    - **化学・科学計算** … **conda が依然として標準**。このコースはこちら側です。

    なぜ化学では conda かというと、両者の"世界観"が違うからです。

    | | pip の世界観 | conda の世界観 |
    |---|---|---|
    | 配るもの | **Pythonのパッケージ**だけ | Python も **R も、C/C++ライブラリも**、環境ごと |
    | 得意 | Python完結のもの | **複数言語・コンパイル済み科学ソフトが混ざる分野** |

**このコースが Miniforge(conda) を主軸にする理由は3つです。**

1. **化学に特化した道具が壊れずに入るから。**
   化学では RDKit・OpenBabel・量子化学ソフトなど「Pythonの外側（C/C++）」に依存する道具が多く、pip だと環境によってビルドに失敗しがちでした。conda-forge は**ビルド済み**で配るので「入れたら動く」。初心者が環境で挫折しないための、いちばん確実な道です。

2. **このコースは Python と R の両方を使うから。**
   これが決定打です。**Python も R 本体も、1つの環境でまとめて管理できるのは conda だけ**。統計の回（第71回〜）で R に切り替えても、同じ仕組みで扱えます。pip は R を管理できません。

3. **初心者が道具を1本に絞れるから。**
   覚えることを減らし、トラブルを減らす。学習に集中できます。

!!! tip "でも pip も必ず覚えます（土台スキル）"
    conda を主軸にしても、**pip は全プログラマー必須の土台**です。conda に無いパッケージは pip で入れますし、将来どんな現場でも使います。だから下で pip の使い方もしっかり扱います。
    さらに速さを求めるなら **uv**（Rust製の高速な pip 互換ツール）という現代的な選択肢もあります。まずは pip に慣れ、慣れたら uv に触れてみる——という順番がおすすめです（深掘りは第19回）。

### conda と pip の使い分け

| | conda | pip |
|---|---|---|
| 入手元 | conda-forge など | PyPI（Pythonの公式倉庫） |
| 得意 | 科学・化学系（**RDKit**, NumPy など、内部でC/C++を使う重いもの） | Python製の軽いパッケージ全般。**世界標準** |
| 使える環境 | conda環境のみ | conda環境でも venv でも使える |

!!! note "かんたんな方針"
    - **conda環境（方法A）を使う人**：化学・科学系は **conda** で、conda に無いものだけ **pip** で。
    - **venv（方法B）を使う人**：すべて **pip** で入れます（venvにcondaは使えません）。
    - どちらでも、**pip はほぼ必ず使う**ので、pip の使い方は覚えておきましょう。

### まず定番パッケージを入れる

=== "conda環境の人"

    ```bash
    # 科学・化学系は conda-forge から（安定）
    conda install -c conda-forge numpy pandas matplotlib seaborn scikit-learn jupyter -y
    ```

=== "venvの人"

    ```bash
    # すべて pip で
    pip install numpy pandas matplotlib seaborn scikit-learn jupyter
    ```

### pip の基本コマンド（丸ごと覚える価値あり）

環境に入った状態（先頭が `(chem)` 等）で使います。

```bash
pip install pandas              # 入れる
pip install "pandas==2.2.0"     # バージョンを指定して入れる
pip install --upgrade pandas    # 最新に更新する
pip uninstall pandas            # 消す
pip list                        # 今入っているもの一覧
pip show pandas                 # 詳細（バージョン・保存場所など）
```

!!! tip "requirements.txt で「まとめて再現」"
    使ったパッケージを一覧ファイルに書き出しておくと、別のPCでも一発で同じ環境を再現できます。研究の再現性にも直結する大事な習慣です。
    ```bash
    pip freeze > requirements.txt      # 今の環境を書き出す
    pip install -r requirements.txt    # 別のPCでまとめて入れる
    ```

!!! warning "conda環境で pip を使うときの注意"
    conda環境の中で pip も使えますが、**同じパッケージを conda と pip の両方で入れない**でください（競合の原因）。
    「基本は conda、conda に無いものだけ pip」と役割を分ければ安全です。

### 化学専用ライブラリ RDKit を入れる

第6部（ケモインフォマティクス）で使う **RDKit** も入れておきましょう。

=== "conda環境の人（推奨）"

    ```bash
    conda install -c conda-forge rdkit -y
    ```

=== "venvの人"

    ```bash
    pip install rdkit
    ```

### 動作確認

```bash
python -c "import numpy, pandas, matplotlib, rdkit; print('OK: 主要パッケージが使えます')"
```

`OK: 主要パッケージが使えます` と出れば成功です。

---

## ステップ2.6　R を入れる（第7部で使用・今は入れるだけ）

!!! info "R が既にある人はスキップ"
    ステップ0で `R --version` が表示された人は、このステップは不要です。

R は統計の回（第71回〜）で使います。今回は入れて動けばOKです。方法はどちらか1つ。

=== "A. conda で入れる（conda環境の人に手軽）"

    ```bash
    conda install -c conda-forge r-base -y
    ```

=== "B. 公式インストーラで入れる（venvの人・R単体で使いたい人）"

    - **Windows 11 / Ubuntu 共通**：[CRAN 公式サイト](https://cran.r-project.org/) からお使いのOS版をダウンロードしてインストール。
    - Ubuntu はコマンドでも入ります：`sudo apt update && sudo apt install r-base -y`

動作確認：ターミナルで `R --version` を打ち、`R version …` と出れば成功です。

---

## ステップ3　VS Code（エディタ）を入れる

=== "Windows 11"

    [VS Code 公式サイト](https://code.visualstudio.com/) から Windows 版をダウンロードしてインストールします。
    （インストール時「PATH への追加」にチェックが入っていればそのままでOK）

=== "Ubuntu"

    ```bash
    sudo snap install code --classic
    ```
    うまくいかない場合は [公式サイト](https://code.visualstudio.com/) から `.deb` を入れてください。

### 拡張機能を入れる

VS Code を開き、左端の四角いアイコン（拡張機能）から次を検索してインストールします。

- **Python**（Microsoft製）… Python を書く・実行する基本
- **Jupyter**（Microsoft製）… セル単位で対話的に実行する

!!! info "AIの相棒を入れる（任意）"
    使っているAIサービスに合わせて、VS Code に補助ツールを追加できます。

    - **ChatGPT を使う場合**: 拡張機能で **「Codex」**（OpenAI製）を入れ、ChatGPTアカウントでサインイン。
    - **Claude を使う場合**: **Claude Code** を使う（VS Code拡張、またはターミナルでCLI）。

    エディタは共通のまま、AIだけ各自のサービスを差し込む——という形になります。契約がなくても、無料の範囲やWebのAIに質問しながら進められます。

---

## ステップ4　Git を入れて名前を設定する

!!! info "Git が既にある人は導入だけスキップ"
    ステップ0で `git version …` が表示された人は、インストールは不要です。ただし**名前とメールの設定（下記）は一度だけ必要**なので、そこは行ってください。

=== "Windows 11"

    [Git for Windows](https://git-scm.com/download/win) をダウンロードしてインストール（設定はすべて初期値でOK）。

=== "Ubuntu"

    ```bash
    sudo apt update && sudo apt install git -y
    ```

インストール後、自分の名前とメールを一度だけ設定します（コミットの署名に使われます）。

```bash
git config --global user.name "あなたの名前"
git config --global user.email "you@example.com"
```

---

## ステップ5　はじめてのプログラムを動かす

いよいよ動作確認です。デスクトップなどに `chem-course` フォルダを作り、VS Code で開きます。
その中に `hello_chem.py` というファイルを新規作成し、次を**そのままコピペ**してください。

```python title="hello_chem.py"
# 水 H2O の分子量を計算してみる
atomic_mass = {"H": 1.008, "O": 15.999}   # 原子量（g/mol）
water = 2 * atomic_mass["H"] + 1 * atomic_mass["O"]

print("はじめまして、化学データ分析の世界へ！")
print(f"水 (H2O) の分子量は {water:.3f} g/mol です")

# いま使っている Python のバージョンも確認
import sys
print("Python:", sys.version.split()[0])
```

### 実行のしかた

- VS Code の右上にある **▶（実行）ボタン**を押す、または
- ターミナルで次を実行します（`(chem)` になっていることを確認）。

```bash
python hello_chem.py
```

次のような出力が出れば大成功です。

```text
はじめまして、化学データ分析の世界へ！
水 (H2O) の分子量は 18.015 g/mol です
Python: 3.12.x
```

!!! success "ここまでできたら"
    環境構築は完了です。おめでとうございます！ ここが一番の山場でした。次回からは実際にプログラムを書いていきます。

---

## 演習問題

**問1.** ライブラリがちゃんと入ったか確認しましょう。次のコードを `check.py` として実行し、エラーが出ないこと・バージョンが表示されることを確かめてください。

```python title="check.py"
import numpy, pandas, matplotlib
print("numpy:", numpy.__version__)
print("pandas:", pandas.__version__)
print("matplotlib:", matplotlib.__version__)
```

**問2.** `hello_chem.py` をまねて、**二酸化炭素 CO₂ の分子量**を計算して表示するプログラムを書いてください。炭素 C の原子量は 12.011、酸素 O は 15.999 とします。

**問3.**（余力があれば）R も動くか確認しましょう。ターミナルで `R` と打つと R が起動します。次を入力してみてください。

```r
1.008 * 2 + 15.999   # 水の分子量を R で計算
q()                  # R を終了（Save workspace? は n でOK）
```

**問4.** パッケージ管理に慣れましょう。環境に入った状態（先頭が `(chem)` 等）で、次を試してください。

1. `pip list` で今入っているパッケージ一覧を表示する。
2. `pip show pandas` で pandas の詳細（バージョンなど）を表示する。
3. `pip freeze > requirements.txt` で一覧をファイルに書き出し、中身を開いて確認する。

---

## 解答

??? success "問1 の解答・確認ポイント"
    3つとも `x.y.z` の形でバージョンが表示されれば成功です。もし `ModuleNotFoundError` が出たら、環境に入り忘れていないか（先頭が `(chem)` 等になっているか）、ステップ2.5 のパッケージ導入（conda install または pip install）が終わっているかを確認してください。

??? success "問2 の解答"
    ```python title="co2.py"
    # 二酸化炭素 CO2 の分子量
    atomic_mass = {"C": 12.011, "O": 15.999}
    co2 = 1 * atomic_mass["C"] + 2 * atomic_mass["O"]
    print(f"二酸化炭素 (CO2) の分子量は {co2:.3f} g/mol です")
    ```

    出力:
    ```text
    二酸化炭素 (CO2) の分子量は 44.009 g/mol です
    ```

??? success "問3 の解答・確認ポイント"
    `R` を起動して式を入力すると、`[1] 18.015` のように答えが表示されます。`[1]` は「結果の1個目」という R の印です。`q()` で終了できれば、R も無事に動いています。

??? success "問4 の解答・確認ポイント"
    - `pip list` … `numpy 2.x`、`pandas 2.x` のように「名前 バージョン」が縦に並びます。
    - `pip show pandas` … `Name: pandas` / `Version: …` / `Location: …`（保存場所）などが表示されます。
    - `pip freeze > requirements.txt` … 画面には何も出ませんが、同じフォルダに `requirements.txt` ができ、`pandas==2.2.0` のような形で一覧が入っています。このファイルがあれば、別のPCで `pip install -r requirements.txt` を実行するだけで同じ環境を再現できます。

    !!! note "conda環境の人へ"
        conda で入れたパッケージも `pip list` にはおおむね表示されます。ただし「環境まるごと」を厳密に再現したいときは、conda 側の `conda env export > environment.yml` の方が確実です（詳しくは第19回で扱います）。

---

### 次回予告

[第2回：はじめてのPython](lesson-02.md) では、Python を電卓のように使いながら、変数・データ型・分子量計算を学びます。
