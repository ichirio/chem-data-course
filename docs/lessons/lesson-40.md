# 第40回　日付・時刻データの扱い

!!! abstract "この回のゴール"
    - 文字列の日時を、pandas の**日時型**に変換する
    - `.dt` で「時」「分」などの成分を取り出す
    - 経過時間を計算する（反応のモニタリングなど）
    - 所要時間の目安: 60分
    - 使うデータ：**反応モニタリング**（時刻ごとの吸光度）

反応の進行、装置のログ、実験の記録——時刻つきデータは化学で頻出します。

```python
import pandas as pd

log = pd.DataFrame({
    "time":       ["2026-04-01 09:00", "2026-04-01 09:30", "2026-04-01 10:00", "2026-04-01 10:30"],
    "absorbance": [0.10, 0.24, 0.41, 0.55],
})
print(log.dtypes)
```

出力:

```text
time           object
absorbance    float64
dtype: object
```

`time` の型は `object`（＝ただの文字列）です。このままでは時間の計算ができません。

---

## 1. 文字列を日時型に変換する

`pd.to_datetime()` で、文字列を**日時型（datetime64）**に変えます。

```python
log["time"] = pd.to_datetime(log["time"])
print(log.dtypes)
```

出力:

```text
time          datetime64[ns]
absorbance           float64
dtype: object
```

型が `datetime64[ns]` になりました。これで時間としての計算ができます。

!!! tip "読み込み時に一発変換もできる"
    CSV から読むときは `parse_dates` を使うと、その場で日時型にできます。
    ```python
    df = pd.read_csv("log.csv", parse_dates=["time"])
    ```

---

## 2. .dt で成分を取り出す

日時型の列は `.dt` を通して「年・月・日・時・分」などを取り出せます。

```python
log["hour"] = log["time"].dt.hour
log["minute"] = log["time"].dt.minute
print(log)
```

出力:

```text
                 time  absorbance  hour  minute
0 2026-04-01 09:00:00        0.10     9       0
1 2026-04-01 09:30:00        0.24     9      30
2 2026-04-01 10:00:00        0.41    10       0
3 2026-04-01 10:30:00        0.55    10      30
```

`.dt.year` / `.dt.month` / `.dt.day` / `.dt.hour` / `.dt.minute` / `.dt.dayofweek`（曜日）などが使えます。

---

## 3. 経過時間を計算する

日時どうしの引き算は「時間の差」になります。反応開始（最初の行）からの**経過分**を求めてみましょう。

```python
start = log["time"].iloc[0]                       # 最初の時刻
elapsed = (log["time"] - start).dt.total_seconds() / 60   # 秒→分
log["elapsed_min"] = elapsed.astype(int)
print(log[["time", "absorbance", "elapsed_min"]])
```

出力:

```text
                 time  absorbance  elapsed_min
0 2026-04-01 09:00:00        0.10            0
1 2026-04-01 09:30:00        0.24           30
2 2026-04-01 10:00:00        0.41           60
3 2026-04-01 10:30:00        0.55           90
```

!!! note "差は「timedelta（時間差）」になる"
    日時の引き算の結果は「2時間30分」のような**時間差**の型です。`.dt.total_seconds()` で秒に直し、60で割れば分、3600で割れば時間になります。この `elapsed_min` と `absorbance` を使えば、次の第5部で**反応の時間変化のグラフ**が描けます。

---

## 演習問題

**問1.** 本文の `log` を作り、`time` を日時型に変換してから `dtypes` を表示し、`datetime64[ns]` になっていることを確認してください。

**問2.** `.dt` を使って、`time` から「時（hour）」の列だけを取り出して表示してください。

**問3.** 反応開始からの経過時間を**分**で求め、`absorbance` と並べて表示してください（本文と同じ手順）。60分後の吸光度はいくつですか？

---

## 解答

??? success "問1 の解答"
    ```python
    import pandas as pd
    log = pd.DataFrame({
        "time":       ["2026-04-01 09:00", "2026-04-01 09:30", "2026-04-01 10:00", "2026-04-01 10:30"],
        "absorbance": [0.10, 0.24, 0.41, 0.55],
    })
    log["time"] = pd.to_datetime(log["time"])
    print(log.dtypes)
    ```

    出力:
    ```text
    time          datetime64[ns]
    absorbance           float64
    dtype: object
    ```

??? success "問2 の解答"
    ```python
    print(log["time"].dt.hour)
    ```

    出力:
    ```text
    0     9
    1     9
    2    10
    3    10
    Name: time, dtype: int32
    ```

??? success "問3 の解答"
    ```python
    start = log["time"].iloc[0]
    log["elapsed_min"] = ((log["time"] - start).dt.total_seconds() / 60).astype(int)
    print(log[["elapsed_min", "absorbance"]])
    ```

    出力:
    ```text
       elapsed_min  absorbance
    0            0        0.10
    1           30        0.24
    2           60        0.41
    3           90        0.55
    ```

    → 60分後（elapsed_min = 60）の吸光度は **0.41** です。

---

## この回のまとめ

- 文字列の日時は `pd.to_datetime()` で日時型（datetime64）に変換。
- 読み込み時は `pd.read_csv(..., parse_dates=["列"])` でその場変換。
- `.dt.hour` `.dt.minute` などで成分を取り出す。
- 日時の引き算は時間差。`.dt.total_seconds()/60` で経過分に。

### 次回予告

[第41回：実験データの前処理レシピ](lesson-41.md) では、現実の「汚れたデータ」（余分な空白・表記ゆれ・数値でない値）を、順を追ってきれいにする定石を学びます。
