# 化学データ分析コース（Python主・R副）

化学専攻の大学生が、**1回1テーマ**で初歩から Python と R を学ぶ全100回のコースです。
毎回、コピペで動くコード・演習問題・解答がついています。基本すべて無料の道具だけで進みます。

- 対象環境: Ubuntu または Windows 11
- 主言語: **Python**（化学・汎用・AI）／ 副言語: **R**（統計）
- 公開: GitHub Pages（`docs/` を MkDocs Material でサイト化）

## サイトをローカルで見る（任意）

```bash
pip install -r requirements.txt
mkdocs serve
# ブラウザで http://127.0.0.1:8000 を開く
```

## 構成

```
docs/
  index.md            … コースの入口
  roadmap.md          … 全100回の仮タイトル
  lessons/
    lesson-01.md      … 第1回 開発環境の構築
    lesson-02.md      … 第2回 はじめてのPython
    lesson-03.md      … 第3回 リストと辞書で化学を扱う
```

## 進め方

週1回・1テーマ。各回の「演習問題」を解いてから「解答」を開くのがおすすめです。
親子で1つの共有リポジトリを使い、お互いのコードを見せ合うと続きます。
