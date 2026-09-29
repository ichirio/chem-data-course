# 第7回　GitとGitHub：仲間・研究室で共有リポジトリを使う

!!! abstract "この回のゴール"
    - Git と GitHub の違いをイメージできる
    - リポジトリを **clone**（手元に複製）できる
    - **add → commit → push** の基本サイクルを回せる
    - **pull** で相手の変更を取り込める
    - 自分の書いたコードを、共有リポジトリに置けるようになる
    - 所要時間の目安: 60分

第1回で Git は入れて、名前とメールも設定しました（まだの人は第1回ステップ4へ）。今回はそれを使います。

---

## 1. Git と GitHub って何が違う？

- **Git** … あなたのパソコンの中で、ファイルの**変更履歴を記録**する仕組み。「いつ・何を・なぜ変えたか」を残せる。**タイムマシン**のようなもの。
- **GitHub** … その履歴を**インターネット上に置いて共有**する場所。**クラウド保存＋共同作業の広場**。

!!! note "なぜ使うの？（研究者にこそ効く）"
    - **やり直せる**：壊しても過去の状態に戻れる。安心して実験できる。
    - **共有できる**：仲間・研究室で同じコードを見て、育てられる。
    - **証拠が残る**：いつ何をしたかの記録は、研究の再現性そのもの。
    - **ポートフォリオになる**：GitHub は「作った証拠」の置き場所。就職でも見られます。

---

## 2. GitHub アカウントを用意する

[github.com](https://github.com/) で無料アカウントを作ります（学習用に、自分専用のアカウントを1つ用意しましょう）。

!!! warning "アカウント作成・パスワードは本人の手で"
    アカウント登録やパスワードの入力は、**必ず本人が自分で**行ってください。ここはAIに任せてはいけない部分です。

誰かと**共有リポジトリ**（例：このコースのリポジトリや、練習用に新しく作ったもの）で共同作業したい場合は、相手を **Collaborator（共同編集者）** として招待すると、お互いに push できます。
（リポジトリの Settings → Collaborators から招待できます。）

---

## 3. clone：リポジトリを手元に複製する

GitHub 上のリポジトリを、自分のパソコンに丸ごとコピーするのが **clone** です。
リポジトリのページで緑色の **「Code」ボタン → HTTPS の URL** をコピーし、ターミナルで次を実行します。

```bash
# 例（URL は自分のものに置き換える）
git clone https://github.com/ichirio/chem-data-course.git

# できたフォルダに入る
cd chem-data-course
```

これで、手元にリポジトリのフォルダができました。VS Code で「フォルダを開く」から、このフォルダを開けば準備完了です。

---

## 4. 基本サイクル：add → commit → push

Git の毎日の使い方は、たった3ステップの繰り返しです。

```mermaid
flowchart LR
    A[ファイルを編集] --> B["git add<br/>（記録する候補に選ぶ）"]
    B --> C["git commit<br/>（履歴に1つ刻む）"]
    C --> D["git push<br/>（GitHubへ送る）"]
```

実際にやってみましょう。手元のフォルダに、練習用のファイルを1つ作ります。

```python title="my_first.py"
# はじめての共有コード
print("これは私が書いた最初のコードです")
```

そして、ターミナルで次を順に実行します。

```bash
# ① いま何が変わったか確認
git status

# ② 記録したいファイルを選ぶ（ファイル名で指定。git add . と書くと「全部」）
git add my_first.py

# ③ 履歴に1つ刻む（-m のあとに「何をしたか」のメモ）
git commit -m "はじめてのコードを追加"

# ④ GitHub へ送る
git push
```

!!! tip "commit メッセージは「何をしたか」を短く"
    `git commit -m "..."` の `...` には、**変更の内容**を書きます。「グルコースの計算を追加」「pH判定のバグを修正」など、後で見て分かる一言に。未来の自分と仲間への手紙です。

push したら、GitHub のページを再読み込みしてみましょう。さっきのファイルが表示されていれば成功です！

---

## 5. pull：相手の変更を取り込む

複数人で同じリポジトリを使うと、相手が push した変更を**自分の手元にも取り込む**必要があります。それが **pull** です。

```bash
# 作業を始める前に、まず最新を取り込む（毎回の習慣に）
git pull
```

!!! success "おすすめの毎日の流れ"
    1. 作業前に **`git pull`**（最新にする）
    2. コードを書く
    3. **`git add` → `git commit` → `git push`**（記録して共有）

    この「pull で始まり push で終わる」を習慣にすると、複数人でぶつからずに共同作業できます。

---

## 6. よくあるつまずき

??? question "push したら「認証して」と言われた"
    はじめての push では GitHub へのログインを求められます。Windows（Git for Windows）では、ブラウザが開いて承認する方式（推奨）に従ってください。Ubuntu では、ここに GitHub のパスワードを入れても通りません（パスワードでの push は廃止されています）。GitHub CLI を入れて（`sudo apt install gh`）`gh auth login` を実行し、画面の指示に従ってブラウザで承認するのが簡単です。パスワードやトークンの入力は、**画面の指示に沿って本人が**行いましょう。

??? question "「Please tell me who you are」と出た"
    第1回の名前・メール設定がまだです。次を一度だけ実行してください。
    ```bash
    git config --global user.name "あなたの名前"
    git config --global user.email "you@example.com"
    ```

??? question "pull したら「conflict（衝突）」と出た"
    同じ場所を二人が別々に直すと起きます。あわてず、ファイルを開いて `<<<<<<<` などの印の部分を見て、どちらを残すか決めて直します。最初のうちは「作業前に必ず pull」を徹底すると、ほとんど起きません。困ったらAIに画面を見せて相談を。

??? question "pull したら「divergent branches」と出て止まった"
    自分も相手も別々に commit していると、Git が「どうまとめるか」を聞いてきます（`fatal: Need to specify how to reconcile divergent branches.`）。最初は次を一度だけ設定しておけば、自動でまとめ（merge）てくれます。
    ```bash
    git config --global pull.rebase false
    ```
    設定したら、もう一度 `git pull` してください。

---

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    Git 初心者です。GitHub から clone した chem-course フォルダに、`molar_mass.py` を新しく作りました。
    フォルダには API キーを書いた `.env` と、自動で作られた `__pycache__/` もあります。
    `molar_mass.py` だけを commit して GitHub に push するまでのコマンドを、1行ずつ意味つきで教えてください。
    `.env` と `__pycache__/` が今後も誤って commit されないよう `.gitignore` の中身も示し、commit の前に「何が記録されるか」を確認するコマンドも入れてください。

!!! warning "AIの出力で確かめること"
    - `git add .` や `git add -A` で丸ごと追加していないか。使うなら直前に `git status` で一覧を見て、`.env`・パスワード・大きな生データが入っていないか確認します。
    - commit の前に `git status --short` を打ち、先頭が `A`（追加予定）の行が意図したファイルだけか。
    - `.gitignore` を作った後、`git status` の一覧から `.env` と `__pycache__/` が消えたか。
    - 一度 push したキーやパスワードは、ファイルを消しても履歴に残ります。うっかり push したら、そのキーは発行元で無効化して作り直す必要があります。
    - `git push --force` や `git reset --hard` など履歴や変更を消すコマンドが含まれていたら、意味を理解するまで実行しない。

---

## 演習問題

**問1.** 共有リポジトリ（なければ GitHub で練習用に1つ新規作成。作成画面で「Add a README file」にチェックを入れておくと、clone 後すぐに push できます）を、自分のパソコンに `git clone` してください。フォルダに入り、`git status` を実行して「何も変更がない」状態を確認しましょう。

**問2.** そのフォルダに、これまでの回で作った好きな `.py` ファイル（例：`molar_mass.py`）を1つ置き、**add → commit → push** の3ステップで GitHub に上げてください。commit メッセージは分かりやすい一言にしましょう。

**問3.** GitHub のリポジトリのページをブラウザで開き、いま push したファイルが表示されていることを確認してください。さらに、ファイルをクリックして中身が読めることも確認しましょう。


**問4.**（検証）フォルダに `molar_mass.py`・`.env`（AI サービスの API キーを書いたファイル）・`__pycache__/`（Python が自動で作るフォルダ）があります。「`molar_mass.py` を GitHub に上げたい」と頼んだところ、AI が次のコマンドを示しました。問題点を指摘し、正しい手順に直してください。

```bash
git add .
git commit -m "update"
git push
```

---

## 解答

??? success "問1 の解答・確認ポイント"
    ```bash
    git clone https://github.com/＜あなた＞/＜リポジトリ名＞.git
    cd ＜リポジトリ名＞
    git status
    ```
    `nothing to commit, working tree clean`（変更なし・きれいな状態）と出れば成功です。

??? success "問2 の解答・確認ポイント"
    ```bash
    # molar_mass.py をフォルダに置いてから
    git add molar_mass.py
    git commit -m "分子量を計算する関数を追加"
    git push
    ```
    `git status` で緑色（add 済み）→ commit で「1 file changed」→ push で「To https://github.com/...」と出れば成功です。

??? success "問3 の解答・確認ポイント"
    GitHub のページを再読み込みすると、ファイル一覧に `molar_mass.py` が現れます。クリックすると中身（コード）が色付きで表示されます。ここまで来たら、あなたのコードはクラウドに安全に保存され、仲間や研究室と共有できる状態です。


??? success "問4 の解答"
    **問題点1：`git add .` で全部追加している。** `.env`（API キー）と `__pycache__/` まで記録され、push すると GitHub 上に公開されてしまいます。一度 push したキーは、後でファイルを消しても履歴に残ります。

    **問題点2：commit メッセージが「update」だけ。** 何をしたのか、後から分かりません。

    **確かめ方**：add した後、commit の前に `git status --short` で「何が記録されるか」を見ます。`git add .` の直後はこうなっています。
    ```text
    A  .env
    A  __pycache__/molar_mass.cpython-312.pyc
    A  molar_mass.py
    ```

    **正しい手順**：まだ commit していなければ `git reset` で add を取り消し、`.gitignore` を作ってから必要なファイルだけを add します。
    ```bash
    git reset                          # add を取り消す（ファイル自体は消えない）
    # .gitignore というファイルを作り、次の2行を書いて保存する
    #   .env
    #   __pycache__/
    git add .gitignore molar_mass.py
    git status --short                 # 記録されるものを確認
    git commit -m "モル質量を計算する関数を追加（.env を .gitignore で除外）"
    git push
    ```

    `git status --short` の出力:
    ```text
    A  .gitignore
    A  molar_mass.py
    ```

    `.env` と `__pycache__/` が一覧から消えていれば成功です。

---

## この回のまとめ

- **Git** は手元の履歴管理、**GitHub** はそれをクラウドで共有する場所
- **clone** でリポジトリを複製、**pull** で最新を取り込む
- 毎日の基本サイクルは **add →（何を選ぶか）→ commit →（履歴に刻む）→ push →（共有）**
- commit メッセージは「何をしたか」を短く。未来の自分への手紙
- 「pull で始まり push で終わる」を習慣に

### 次回予告

[第8回：まとめ演習](lesson-08.md) では、ここまで学んだ **変数・型・リスト・辞書・if・for・関数** を全部使って、対話式の「ミニ分子量電卓アプリ」を作ります。第1部の総仕上げです。
