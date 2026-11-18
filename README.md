<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn_Keyence - キーエンス KV 上位リンク通信エクステンション

👉 [← TermAddIn 統合本体に戻る](https://github.com/Monaca-App/TermAddIn)  
👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-TermAddIn%20Extension-orange.svg)]()
[![License](https://img.shields.io/badge/License-TermAddIn__Keyence%20License-blue.svg)]()

## 📖 概要

**TermAddIn_Keyence** は、Excel 総合通信アドイン **TermAddIn** 上で動作する **キーエンス（KEYENCE）製 PLC（KV-8000 / KV-7500 / KV-7300 / KV-5500 / KV-5000 / KV-3000 / KV-1000 / KV-Nano シリーズ）向け KV 上位リンク通信専用エクステンション（機能ドメイン単一 DLL）** です。

KV シリーズの内蔵 Ethernet ポート・Ethernet ユニット（KV-EP21V / KV-LE21V 等）・RS-232C シリアルポートと、 **Ethernet TCP / UDP 上位リンク（標準ポート 8501）** および **RS-232C シリアル上位リンク（`CR` / `CQ` セッション自動送出対応）** を用いてダイレクトに交信します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままで KV 各種データメモリ・リレー（DM, EM, FM, ZF, W, TM, Z, R, MR, LR, CR, B 等）の一括読出・一括書込、および任意上位リンクコマンド送受信** を高速・安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Keyence` 個別ライセンス（料金未定・準備中）** が適用されます（なお、 **TermAddIn 本体** は現在 **近日公開** であり、全機能を無償でご試用いただける **評価期間は公開時に設定** いたします）。

---

## ✨ 主な機能・特徴

1. **Ethernet（TCP / UDP ポート 8501）& RS-232C シリアル上位リンク両対応**
   * 設定レスで高速交信できる **Ethernet TCP / UDP 上位リンク（標準ポート 8501）** に加え、シリアルポート経由での **`CR` / `CQ` 自動セッション管理付き RS-232C 上位リンク** にも対応。仮想シリアルツール [c2c](https://github.com/Monaca-App/c2c) の KEYENCE KV 仮想スレーブとも直結テストが可能。
2. **KV デバイス & データ形式サフィックス（`.U` / `.S` / `.H` / `.D` / `.L`）完全対応**
   * `DM`, `EM`, `FM`, `ZF`, `TM`, `Z`, `T`, `C`（10進）、`W`（16進リンクレジスタ）、`R`, `MR`, `LR`, `CR`（10進リレー）、`B`, `VB`（16進リレー）およびデータ形式指定サフィックスを自動識別。
3. **単一点・連続アクセス電文の自動最適化（RD/RDS, WR/WRS, ST/RS）**
   * `Kv_ReadWord`, `Kv_WriteWord`, `Kv_ReadBit`, `Kv_WriteBit` が指定点数に応じて単一点コマンド（`RD`, `WR`, `ST`, `RS`）と連続コマンド（`RDS`, `WRS`）を自動選択し、`E0`〜`E6` エラー応答も厳密に検知。
4. **任意上位リンク ASCII コマンド送受信（Kv_Send / Kv_Receive）**
   * `?K`（機種確認）、`?M`（運転モード確認）、`M1` / `M0`（モード切替）、`?E`（CPU エラー確認）、`WRT`（時計書込）など全上位リンクコマンドを送受信可能。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 4.8 標準マネージド通信基盤のみで内製実装されており、外部の専用 ActiveX やネイティブ DLL に依存しないため、Excel 32bit 版・64bit 版の双方で完全動作。

---

## 🔌 提供コマンド一覧（全 9 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

### 回線制御・通信条件設定コマンド（3 コマンド）

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Kv_Open`** | `Kv_Open, "[Endpoint]", "[PortOrOption]", "[Protocol]"` | KV 上位リンク回線（TCP/UDP 8501 または Serial）を開設 | [詳細説明](./Manual/Keyence/Kv_Open.Md) |
| **`Kv_Close`** | `Kv_Close` | 通信回線を安全に切断（Serial 時は `CQ` 自動送出可） | [詳細説明](./Manual/Keyence/Kv_Close.Md) |
| **`Kv_Mode`** | `Kv_Mode, "[Protocol]", "[DataSuffix]", "[AutoSession]"` | トランスポート方式・既定データサフィックス・自動 `CR`/`CQ` 設定 | [詳細説明](./Manual/Keyence/Kv_Mode.Md) |

### デバイスアクセス・高水準コマンド（4 コマンド）

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Kv_ReadWord`** | `Kv_ReadWord, "[Device]", [Points], "[DestKey]", "[Format]"` | ワードデバイス読出（`RD` / `RDS`） | [詳細説明](./Manual/Keyence/Kv_ReadWord.Md) |
| **`Kv_WriteWord`** | `Kv_WriteWord, "[Device]", "[Values]"` | ワードデバイス書込（`WR` / `WRS`、`OK` 応答検証） | [詳細説明](./Manual/Keyence/Kv_WriteWord.Md) |
| **`Kv_ReadBit`** | `Kv_ReadBit, "[Device]", [Points], "[DestKey]"` | リレー／ビット状態読出（`RD` / `RDS`） | [詳細説明](./Manual/Keyence/Kv_ReadBit.Md) |
| **`Kv_WriteBit`** | `Kv_WriteBit, "[Device]", "[Values]"` | リレー／ビット書込（1点 `ST`/`RS`、複数点 `WRS`） | [詳細説明](./Manual/Keyence/Kv_WriteBit.Md) |

### 汎用上位リンク送受信コマンド（2 コマンド）

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Kv_Send`** | `Kv_Send, "[Command]"` | 任意の KV 上位リンク ASCII コマンド（`?K`, `?M` 等）を送信 | [詳細説明](./Manual/Keyence/Kv_Send.Md) |
| **`Kv_Receive`** | `Kv_Receive, "[DestKey]", "[Format]"` | 上位リンク応答を受信・`E0`〜`E6` エラー検証後にデータ抽出 | [詳細説明](./Manual/Keyence/Kv_Receive.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **対象 PLC** | キーエンス KV-8000 / KV-7500 / KV-7300 / KV-5500 / KV-5000 / KV-3000 / KV-1000 / KV-Nano シリーズ（内蔵 Ethernet ポート、KV-EP21V、KV-LE21V、KV-L21V 等） |
| **対応プロトコル** | KEYENCE KV 上位リンク通信（Ethernet TCP/IP・UDP/IP 標準ポート 8501、RS-232C シリアル上位リンク） |

---

## 🔄 通信シーケンスの概要

### KV データメモリ一括読出の流れ

```mermaid
sequenceDiagram
    autonumber
    actor User as "Excel ワークシート"
    participant Term as "TermAddIn (Keyence)"
    participant PLC as "KEYENCE KV シリーズ PLC"

    User->>Term: "Kv_Open, '192.168.0.10', '8501', 'TCP'"
    Term->>PLC: "TCP ポート 8501 接続確立 (Serial時は CR -> CC)"
    PLC-->>Term: "接続完了"

    User->>Term: "Kv_ReadWord, 'DM100', 2, 'RCV_DM100', 'DEC'"
    Note over Term: "上位リンク連続読出コマンド自動生成\n'RDS DM100.U 2\r'"
    Term->>PLC: "コマンド送信"
    PLC-->>Term: "ASCII 応答返信 ('01234 05678\r\n')"
    Note over Term: "エラーコード検証 & 数値整形\nRCV_DM100 = '1234,5678'"
    Term-->>User: "結果セルへ自動書込展開"

    User->>Term: "Kv_Close"
    Term->>PLC: "回線切断 (Serial時は CQ 送出)"
```

---

## 🔰 シーケンス記述例・同梱サンプル資材（`Sample/Keyence/`）

Ethernet TCP 上位リンクを用いた KV データメモリ（`DM100`〜`DM101`）一括読出の構成例です。

```csv
; 1. KEYENCE KV 上位リンク通信回線をオープン（TCP/IP, Port 8501）
,SEQUENCE, 010, Kv_Open, "192.168.0.10", "8501", "TCP"
; 2. DM100 から 2 ワード一括読出を実行（10進形式）
,SEQUENCE, 020, Kv_ReadWord, "DM100", 2, "PLC_DM100", "DEC"
; 3. 通信回線をクローズして終了
,SEQUENCE, 030, Kv_Close
,SEQUENCE, 040, Exit
```

### 同梱サンプルマクロ・AI自動生成プロンプト集
Ethernet（TCP/UDP）および RS-232C シリアル上位リンクに対応した動作確認用マクロスクリプト（CSV）および AI 自動生成プロンプトが **`Sample/Keyence/`** に収録されています（Excel リボンの **「マニュアル配置 (`Make_Doc`)」** 実行時には `<設定場所>\Sample\TermAddIn\Keyence\` へ自動展開されます）。

* **[`Sample/Keyence/Keyence_KV.csv`](./Sample/Keyence/Keyence_KV.csv)**: KEYENCE KV 上位リンク汎用マクロスクリプトサンプル
* **[`Sample/Keyence/Keyence_KV_prompt.md`](./Sample/Keyence/Keyence_KV_prompt.md)**: KEYENCE KV 上位リンク用マクロスクリプト AI 自動生成プロンプト

---

## 💳 ライセンス・購入プラン（TermAddIn_Keyence）

> [!NOTE]
> **料金プランおよび正式販売について**  
> 本エクステンション（`TermAddIn_Keyence`）の販売料金およびライセンスプラン（買い切り・サブスクリプション）は **現在検討中（未定）** です。  
> 正式な価格およびお申込み受付開始時期が決定次第、本ドキュメントおよび [Monaca アプリ工房ポータル](https://github.com/Monaca-App/store) にてご案内いたします。  
> なお、[TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) は現在 **近日公開** となっており、全機能を無償でご試用いただける **評価期間は公開時に設定** いたします。

| プラン | 価格 | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **未定** | 準備中 |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **未定** | 準備中 |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **未定** | 準備中 |

* ※ **ライセンスファイル名**: `TermAddIn_Keyence.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Keyence.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。

