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
time              str
absorbance    float64
dtype: object
```

`time` の型は `str`（＝ただの文字列。pandas 2.x では `object` と表示）です。このままでは時間の計算ができません。

---

## 1. 文字列を日時型に変換する

`pd.to_datetime()` で、文字列を**日時型（datetime64）**に変えます。

```python
log["time"] = pd.to_datetime(log["time"])
print(log.dtypes)
```

出力:

```text
time          datetime64[us]
absorbance           float64
dtype: object
```

型が `datetime64[us]` になりました（`[us]` は時刻をマイクロ秒単位で持つという意味。pandas 2.x では `[ns]`＝ナノ秒と表示されます）。これで時間としての計算ができます。

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

## AIに任せるときの指示と確認

!!! example "AIへの頼み方（例）"
    pandas の DataFrame `log`（列: time は "2026-04-01 09:00" 形式の文字列、absorbance は吸光度）について、
    time を `pd.to_datetime` で日時型に変換し、最初の測定からの**経過時間（分）**を elapsed_min 列として追加してください。
    経過時間は「時刻の差 → `.dt.total_seconds()` → 60 で割る」で計算し、`.dt.minute` は使わないでください。
    変換後の dtypes と、time・elapsed_min・absorbance の3列を表示してください。

!!! warning "AIの出力で確かめること"
    - `print(log.dtypes)` で time が `datetime64[...]` になっているか。`str`（pandas 2.x では `object`）のままなら、変換を代入し忘れています。
    - 経過時間に `.dt.minute`（「時刻の分の部分」）を使っていないか。1時間を超えると 0, 30, 0, 30 のように**戻って**しまいます。経過時間は単調に増えるはずなので、`elapsed_min.is_monotonic_increasing` が True かを確認します。
    - `astype(int)` で切り捨てていないか。秒まで記録されたデータだと 29.9 分が 29 分になります。必要なら `.round()` してから整数にします。
    - 日付の書式。`"04/01/2026"` のような文字列は 4月1日とも 1月4日とも読めます。AI に `format="%Y-%m-%d %H:%M"` を明示させ、変換後の最初の行が意図した日時かを print で確かめます。

---

## 演習問題

**問1.** 本文の `log` を作り、`time` を日時型に変換してから `dtypes` を表示し、`datetime64[us]`（pandas 2.x では `datetime64[ns]`）になっていることを確認してください。

**問2.** `.dt` を使って、`time` から「時（hour）」の列だけを取り出して表示してください。

**問3.** 反応開始からの経過時間を**分**で求め、`absorbance` と並べて表示してください（本文と同じ手順）。60分後の吸光度はいくつですか？

**問4.**（検証型）次は AI が「反応開始からの経過時間（分）」を求めたコードとその出力です（`log["time"]` は日時型に変換済み）。問題点を指摘し、直してください。

```python
log["elapsed_min"] = log["time"].dt.minute - log["time"].dt.minute.iloc[0]
print(log[["time", "elapsed_min", "absorbance"]])
```

出力:
```text
                 time  elapsed_min  absorbance
0 2026-04-01 09:00:00            0        0.10
1 2026-04-01 09:30:00           30        0.24
2 2026-04-01 10:00:00            0        0.41
3 2026-04-01 10:30:00           30        0.55
```

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
    time          datetime64[us]
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

??? success "問4 の解答"
    **誤り**：`.dt.minute` は「時刻のうち**分の部分**（0〜59）」を取り出すだけです。10:00 の分の部分は 0 なので、経過時間が 30 分から 0 分に**戻って**しまいました。

    **確かめ方**：経過時間は時間とともに増え続けるはずです。吸光度は増えているのに経過時間が 0 に戻っている点で気づけます（`log["elapsed_min"].is_monotonic_increasing` が False になります）。

    **直し方**：日時どうしを引き算して時間差にし、`.dt.total_seconds()` で秒にしてから 60 で割ります。

    ```python
    log["elapsed_min"] = (log["time"] - log["time"].iloc[0]).dt.total_seconds() / 60
    print(log[["time", "elapsed_min", "absorbance"]])
    ```

    出力:
    ```text
                     time  elapsed_min  absorbance
    0 2026-04-01 09:00:00          0.0        0.10
    1 2026-04-01 09:30:00         30.0        0.24
    2 2026-04-01 10:00:00         60.0        0.41
    3 2026-04-01 10:30:00         90.0        0.55
    ```

---

## この回のまとめ

- 文字列の日時は `pd.to_datetime()` で日時型（datetime64）に変換。
- 読み込み時は `pd.read_csv(..., parse_dates=["列"])` でその場変換。
- `.dt.hour` `.dt.minute` などで成分を取り出す。
- 日時の引き算は時間差。`.dt.total_seconds()/60` で経過分に。

### 次回予告

[第41回：実験データの前処理レシピ](lesson-41.md) では、現実の「汚れたデータ」（余分な空白・表記ゆれ・数値でない値）を、順を追ってきれいにする定石を学びます。
