# コマンドリファレンス：Kv_WriteBit

## 概要

**Kv_WriteBit** は、KEYENCE KV 上位リンクのセット／リセットコマンド（1 点時 `ST` / `RS`）または連続書込コマンド（複数点時 `WRS`）を用いて、PLC の **リレー／ビットデバイス（R, MR, LR, CR, B, VB, TS, CS、および DM100.05 等のワード内ビット）へ ON/OFF 状態を書込** する通信制御系コマンドです。
書き込み完了後、KV 本体からの正常確認応答（`OK` / `CC` / `CF`）を自動検証します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Kv_WriteBit, [Device], [Values], "[Reserved]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `Device` | **必須** | 書き込みを開始する先頭リレー／ビットデバイス名（対応デバイス: `R`［接頭辞省略可］, `MR`, `LR`, `CR`, `B`［16進アドレス］, `VB`［16進アドレス］, `TS`, `CS`、およびワードデバイスの `.00`〜`.15` ビット指定。例: `"MR100"`, `"R500"`, `"B0"`, `"DM100.05"`）。 |
| 第 2 引数 | `Values` | **必須** | 書き込むビット状態（`"1"`/`"ON"`/`"TRUE"` が ON、それ以外が OFF。カンマまたは空白区切りで指定し、1 点時は `ST`/`RS`、2 点以上時は `WRS` 電文を自動構築します。例: `"ON"` または `"1, 0, 1"`）。 |
| 第 3 引数 | `Reserved` | 任意 | 互換用の予備引数（`ValidateParamRange(2, 3)` により 3 引数まで許容されますが、指定された値は参照されません）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 複数データのカンマ区切り値（例: `"1, 0, 1"`）を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Kv_Write_Bit`, `Analysis_Kv_WriteBit`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_KV_WRITEBIT", ""
; 1. 接続開始
,SEQUENCE, 010, Kv_Open, "192.168.0.10", "8501", "TCP"
; 2. MR100 を単一点セット (ST MR100)
,SEQUENCE, 020, Kv_WriteBit, "MR100", "ON"
; 3. MR200〜MR202 に 1, 0, 1 を連続書込 (WRS MR200 3 1 0 1)
,SEQUENCE, 030, Kv_WriteBit, "MR200", "1, 0, 1"
; 4. 切断
,SEQUENCE, 040, Kv_Close
,SEQUENCE, 050, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: `Kv_Open` により KEYENCE KV（`192.168.0.10:8501`, TCP）との通信回線が開設されている状態。
* **実行後の状態**: ステップ `020` で `"ST MR100\r"` が、ステップ `030` で `"WRS MR200 3 1 0 1\r"` が送受信され、それぞれ `"OK"` 応答が確認されて `param.IsCommandGo = true` となり、ステップ `040` の `Kv_Close` へ進みます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Driver as "TermSequenceDriver (実行エンジン)"
    participant Cmd as "Kv_WriteBitCommand"
    participant Engine as "KvDeviceParser / KvUpperLinkEngine"
    participant Trans as "KvTransportClient.Shared"

    Driver->>Driver: ライセンス検証 (ValidateStep)
    Driver->>Cmd: Func(parent, param)
    activate Cmd
    Cmd->>Cmd: ValidateParamRange(2, 3) ＆ $PARAM$ 展開
    Cmd->>Engine: Parse(devStr) ＆ ParseBitValues(valsStr) ＆ BuildBitWriteCommand(ST / RS / WRS)
    Cmd->>Trans: TransceiveCommand(reqCmd, param)
    alt 正常確認応答 (OK / CC / CF)
        Cmd-->>Driver: 正常完了 (IsCommandGo = true)
    else 無応答タイムアウト または KV エラー応答 (E0〜E9 等)
        Cmd->>Cmd: SetErrorAndHalt(-1, errMsg)
        Cmd-->>Driver: 異常停止 (IsCommandGo = false)
    else 引数範囲外 または デバイス表記不正例外
        Cmd->>Cmd: _log.LogError / _log.LogCritical ＆ SetErrorAndHalt
        Cmd-->>Driver: 異常停止 (IsCommandGo = false)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常や実行時エラーに対し、解析・実行エンジンが実行するセーフティ規約です。

1. **引数範囲およびデバイス構文の検証**:
   - 引数が 1 個以下または 4 個以上の場合は `ValidateParamRange(2, 3)` により `_log.LogError` と `SetErrorAndHalt` で遮断します。未対応デバイスやアドレス不正により `KvDeviceParser.Parse` が例外を送出した場合は、`catch` 節で `_log.LogCritical` に記録し `SetErrorAndHalt(-1, ex.Message)` で安全停止します。
2. **書込応答タイムアウトおよび KV エラーコード時の停止**:
   - 受信データが空／空白のみ（タイムアウト）の場合、または KV 本体から `E0`〜`E9` エラーや非 `OK` 応答が返された場合は、`SetErrorAndHalt(-1, ...)` を呼び出してシーケンスを即座に安全停止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Kv_WriteBitCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `KvTransportClient.Shared.IsSimulationMode` を開始前の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 環境下で引数 `"MR100"`, `"ON"` および擬似応答 `"OK\r\n"` を与えて実行 | `!param.HasError` かつ `param.IsCommandGo == true`、`param.SndData == "ST MR100"` となる | 単一点ビット ON 指定時のセットコマンド（`ST`）自動選択および `OK` 応答検証 |
| **2** | 異常系 | 引数 0 個（`new TermSequence { Name = "Kv_WriteBit" }`）で `Func(null, invalidParam)` を実行 | `ValidateParamRange(2, 3)` により拒否され、`invalidParam.HasError == true` かつ `!invalidParam.IsCommandGo` となる | 必須引数欠落時のバリデーション拒否検証 |
