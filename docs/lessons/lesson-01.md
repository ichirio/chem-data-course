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

## ステップ1　Miniforge を入れる（Python の土台）

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

プロジェクトごとに Python 環境を分けておくと、後でトラブりません。`chem` という名前の環境をつくり、よく使うライブラリを入れます。

```bash
# chem という名前の環境を作る（Python 3.12 を指定）
conda create -n chem python=3.12 -y

# 作った環境に入る（これ以降、先頭が (chem) になる）
conda activate chem

# 学習で使う定番ライブラリをまとめて入れる
conda install -c conda-forge numpy pandas matplotlib seaborn scikit-learn jupyter -y
```

!!! tip "毎回のはじめに"
    次回以降、作業を始めるときは必ず `conda activate chem` を実行して、先頭が `(chem)` になっていることを確認してください。

化学専用ライブラリ **RDKit** も入れておきましょう（第6部で本格的に使います）。

```bash
conda install -c conda-forge rdkit -y
```

R も入れておきます（第7部で使います。今回は入れるだけでOK）。

```bash
conda install -c conda-forge r-base -y
```

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

!!! info "AIの相棒を入れる（前回の続き）"
    - **息子さん（ChatGPT契約）**: 拡張機能で **「Codex」**（OpenAI製）を入れ、ChatGPTアカウントでサインイン。
    - **お父さん（Claude Pro契約）**: **Claude Code** を使う（VS Code拡張、またはターミナルでCLI）。

    エディタは同じ、AIだけ各自の契約を差し込む——という形になります。

---

## ステップ4　Git を入れて名前を設定する

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

---

## 解答

??? success "問1 の解答・確認ポイント"
    3つとも `x.y.z` の形でバージョンが表示されれば成功です。もし `ModuleNotFoundError` が出たら、`conda activate chem` を忘れていないか、ステップ2の `conda install` が終わっているかを確認してください。

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

---

### 次回予告

[第2回：はじめてのPython](lesson-02.md) では、Python を電卓のように使いながら、変数・データ型・分子量計算を学びます。
