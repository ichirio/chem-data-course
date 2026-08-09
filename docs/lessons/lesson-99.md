# 第99回　クラスタリング（化合物のグループ分け）

!!! abstract "この回のゴール"
    - 教師なし学習の**クラスタリング**を知る
    - `KMeans` で化合物を似た者どうしにグループ分けする
    - 結果を可視化・解釈する
    - 所要時間の目安: 60分
    - 使うテーマ：**化合物ライブラリの自動グループ分け**

**クラスタリング**は「正解なし」で、データを**似た者どうしのかたまり**に分ける教師なし学習です。「この化合物ライブラリには、どんなグループがあるか？」を自動で見つけます。

`ml99.py` を作りましょう（第97回の記述子 `df` を使います）。

---

## 1. KMeans でグループ分け

**KMeans** は、指定した数（k）のグループにデータを分ける定番手法です。標準化（第97回）してから使います。

```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans

features = ["MW", "LogP", "TPSA", "HBD", "HBA", "RotB"]
X_scaled = StandardScaler().fit_transform(df[features])

km = KMeans(n_clusters=3, random_state=42, n_init=10)
km.fit(X_scaled)
df["cluster"] = km.labels_          # 各分子のグループ番号

print(df[["name", "cluster"]].to_string(index=False))
```

出力:

```text
       name  cluster
   methanol        0
    ethanol        0
    benzene        0
    toluene        0
     octane        0
    aspirin        1
   caffeine        1
    glucose        2
     hexane        0
naphthalene        0
  ibuprofen        1
acetic_acid        0
```

3つのグループに自動で分かれました。読み解くと:

- **クラスタ0**：小さめ・単純な分子（アルコール・炭化水素）
- **クラスタ1**：医薬品的な分子（aspirin, caffeine, ibuprofen）
- **クラスタ2**：glucose（水酸基が多く極性が特異）だけ独立

正解を与えていないのに、化学的に意味のあるグループができました。

---

## 2. 可視化する

PCA（第98回）で2次元にして、クラスタを色分けで表示します。

```python
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA

pcs = PCA(n_components=2).fit_transform(X_scaled)

plt.figure(figsize=(6, 5))
for c in range(3):
    mask = km.labels_ == c
    plt.scatter(pcs[mask, 0], pcs[mask, 1], label=f"cluster {c}", s=60)
plt.xlabel("PC1")
plt.ylabel("PC2")
plt.title("KMeans Clusters (on PCA)")
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("clusters.png", dpi=100)
plt.show()
```

生成した図:

![KMeansクラスタリング](../images/lesson99_cluster.png)

色分けされたグループが、PCAの地図の上で分かれて見えます。glucose（クラスタ2）が右端に離れて独立しているのが分かります。

!!! note "k（グループ数）の決め方"
    KMeans は「いくつに分けるか（k）」を先に決める必要があります。適切な k は、**エルボー法**（クラスタ内のばらつきの減り方を見る）や**シルエット係数**で判断します。まずはいくつか試して、化学的に意味のある分け方を探すのが実践的です。

---

## 3. クラスタリングの使いどころ

!!! success "教師なしで構造を発見"
    - **化合物ライブラリの俯瞰**：大量の化合物にどんなグループがあるか把握。
    - **多様性の確保**：各クラスタから代表を選び、偏りなくスクリーニング。
    - **異常検知**：どのクラスタにも属さない分子（外れ値）を見つける。
    
    「ラベルがないデータから構造を見つける」のがクラスタリングの価値。正解を知らなくても、データが教えてくれます。

---

## 演習問題

**問1.** 第97回の記述子を標準化し、`KMeans(n_clusters=3, random_state=42, n_init=10)` でクラスタリングして、各分子のクラスタ番号を表示してください。

**問2.** クラスタ数を `n_clusters=2` に変えて実行し、2グループに分けるとどう分かれるか見てください（大きく2つに分かれるはずです）。

**問3.** PCA で2次元にして、クラスタを色分けした散布図を描いてください。グループが空間的に分かれているか確認しましょう。

---

## 解答

??? success "問1 の解答"
    ```python
    from sklearn.preprocessing import StandardScaler
    from sklearn.cluster import KMeans
    features = ["MW","LogP","TPSA","HBD","HBA","RotB"]
    Xs = StandardScaler().fit_transform(df[features])
    km = KMeans(n_clusters=3, random_state=42, n_init=10).fit(Xs)
    df["cluster"] = km.labels_
    print(df[["name","cluster"]].to_string(index=False))
    ```
    3つのグループ（単純分子・医薬品的分子・glucose）に分かれます。

??? success "問2 の解答"
    ```python
    km2 = KMeans(n_clusters=2, random_state=42, n_init=10).fit(Xs)
    df["cluster2"] = km2.labels_
    print(df[["name","cluster2"]].to_string(index=False))
    ```
    2グループにすると、より大まかな分け方（例：小分子 vs 大きい/複雑な分子）になります。

??? success "問3 の解答"
    ```python
    import matplotlib.pyplot as plt
    from sklearn.decomposition import PCA
    pcs = PCA(n_components=2).fit_transform(Xs)
    for c in sorted(set(km.labels_)):
        mask = km.labels_ == c
        plt.scatter(pcs[mask,0], pcs[mask,1], label=f"cluster {c}", s=60)
    plt.legend(); plt.xlabel("PC1"); plt.ylabel("PC2"); plt.tight_layout()
    plt.show()
    ```

---

## この回のまとめ

- クラスタリングは教師なしで「似た者どうし」に分ける。
- `KMeans(n_clusters=k, random_state=, n_init=10)` で k 個のグループに。
- 標準化してから適用、PCA で可視化。
- k の決め方はエルボー法など。化合物の俯瞰・多様性確保・異常検知に。

### 次回予告

[第100回：まとめ演習（QSAR）](lesson-100.md) では、第8部の総仕上げとして、分子の構造から物性を予測する QSAR を、一気通貫で作ります。
