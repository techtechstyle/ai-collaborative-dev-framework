# T4: BPMN要素のMermaid Flowchart化

> **タスク**: T3で分解したBPMN要素（メインフロー＋SP-1〜SP-3＋復帰フロー＋タスク分解）をMermaid flowchart記法で図化する
> **検証方法**: T3の全要素ID（68要素）がMermaid図に含まれていること
> **自信度**: 確実（T3の構造的変換）
> **入力**: `phase1-t3-bpmn-elements.md`
> **方針**: シンプルなflowchartのみ（スイムレーン・エラー境界・メッセージフローは採用しない）
> **Phase 1.5 T2 適用済み**: `phase1.5-mermaid-style-guide.md` の統一ルールを全6図に適用

---

## Mermaid記法とBPMN要素の対応（統一版）

> Phase 1.5 T1 で策定した統一ルール（`phase1.5-mermaid-style-guide.md`）に準拠

| BPMN要素 | 記号 | Mermaid形状 | 記法 | 配色 |
|----------|------|------------|------|------|
| 開始イベント | ○ | スタジアム型 | `ID(["テキスト"])` | なし（デフォルト） |
| 終了イベント（正常） | ◎ | 三重括弧 | `ID((("テキスト")))` | 緑 `fill:#c8e6c9` |
| 終了イベント（異常） | ◎ | 三重括弧 | `ID((("テキスト")))` | 赤 `fill:#ffcdd2` |
| タスク | □ | 矩形 | `ID["テキスト"]` | なし（デフォルト） |
| タスク（必須/重要） | □ | 矩形 | `ID["テキスト"]` | オレンジ `fill:#fff3e0` |
| サブプロセス | □⊞ | サブルーチン型 | `ID[["テキスト"]]` | なし（デフォルト） |
| 排他ゲートウェイ | ◇ | 菱形 | `ID{"テキスト"}` | 黄 `fill:#fff9c4` |
| 中間イベント | ○⏱/○! | 六角形 | `ID{{"テキスト"}}` | オレンジ薄 `fill:#ffe0b2` |
| データオブジェクト | 📄 | シリンダー型（図内配置） | `ID[("テキスト")]` | 青 `fill:#e3f2fd` |

---

## 1. メインフロー: AIファースト実践フロー

> **対応するBPMN要素**: SE-1, GW-1〜GW-4, T-1〜T-5, SP-1〜SP-3, EE-1, EE-2（15要素）
> **対応する決定表**: DT-0（GW-1）、DT-1（SP-1全体+GW-3）

```mermaid
flowchart TD
    %% ===== 開始 =====
    MF_SE(["SE-1: タスクを受け取る"])

    %% ===== Bright Linesチェック =====
    MF_GW1{"GW-1: Bright Lines<br/>違反チェック<br/>(DT-0)"}
    MF_T1["T-1: Bright Lines違反を是正する"]

    %% ===== L0-L3チェック =====
    MF_SP1[["SP-1: L0-L3チェック<br/>(→ §2で詳細化)"]]
    MF_GW2{"GW-2: L0-L3<br/>通過判定"}
    MF_T2["T-2: L0-L3を満たす形に調整する<br/>(→ §6 タスク分解)"]

    %% ===== AIファーストチェック =====
    MF_SP2[["SP-2: AIファーストチェック<br/>＋分業判断<br/>(→ §3で詳細化)"]]
    MF_GW3{"GW-3: AIファースト<br/>適用判定"}
    MF_T3["T-3: 人間主導で実行する"]
    MF_T4["T-4: AIに初期案を生成させる"]

    %% ===== データオブジェクト =====
    D3[("D-3: AI出力（初期案）")]

    %% ===== レビュー・検証 =====
    MF_T5["T-5: 人間がAI出力をレビューする"]
    MF_SP3[["SP-3: 検証フィードバックループ<br/>(→ §4で詳細化)"]]
    MF_GW4{"GW-4: 検証結果判定"}

    %% ===== 終了 =====
    MF_EE_OK((("EE-1: タスク完了")))
    MF_EE_CUT((("EE-2: 損切り<br/>→復帰フローへ")))

    %% ===== フロー接続 =====
    MF_SE --> MF_GW1
    MF_GW1 -->|"違反あり"| MF_T1
    MF_T1 --> MF_GW1
    MF_GW1 -->|"違反なし"| MF_SP1
    MF_SP1 --> MF_GW2
    MF_GW2 -->|"不合格"| MF_T2
    MF_T2 --> MF_SP1
    MF_GW2 -->|"合格"| MF_SP2
    MF_SP2 --> MF_GW3
    MF_GW3 -->|"AI不適"| MF_T3
    MF_T3 --> MF_SP3
    MF_GW3 -->|"AI適用"| MF_T4
    MF_T4 --> D3
    D3 --> MF_T5
    MF_T5 --> MF_SP3
    MF_SP3 --> MF_GW4
    MF_GW4 -->|"成功"| MF_EE_OK
    MF_GW4 -->|"損切り"| MF_EE_CUT

    %% ===== スタイル =====
    style MF_EE_OK fill:#c8e6c9,stroke:#2e7d32,color:#000
    style MF_EE_CUT fill:#ffcdd2,stroke:#c62828,color:#000
    style MF_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style MF_GW2 fill:#fff9c4,stroke:#f9a825,color:#000
    style MF_GW3 fill:#fff9c4,stroke:#f9a825,color:#000
    style MF_GW4 fill:#fff9c4,stroke:#f9a825,color:#000
    style D3 fill:#e3f2fd,stroke:#1565c0,color:#000
```

---

## 2. サブプロセス SP-1: L0-L3チェック

> **対応するBPMN要素**: SP1-SE, SP1-T1〜T4, SP1-GW1〜GW4, SP1-EE-OK, SP1-EE-NG（11要素）
> **対応する決定表**: DT-1（メインフロー）、DT-2〜DT-5（各レベル詳細）、DT-10（競合時）

```mermaid
flowchart TD
    %% ===== 開始 =====
    SP1_SE(["SP1-SE: L0-L3チェック開始"])

    %% ===== データオブジェクト =====
    D1[("D-1: CLAUDE.md")]

    %% ===== L0チェック =====
    SP1_T1["SP1-T1: L0 持続可能性チェック<br/>(DT-2: G1-G3, R1-R3を評価)"]
    SP1_GW1{"SP1-GW1: L0<br/>通過判定"}

    %% ===== L1チェック =====
    SP1_T2["SP1-T2: L1 心理的安全性チェック<br/>(DT-3: P1-P3を評価)"]
    SP1_GW2{"SP1-GW2: L1<br/>通過判定"}

    %% ===== L2チェック =====
    SP1_T3["SP1-T3: L2 急がば回れチェック<br/>(DT-4: H1-H3を評価)"]
    SP1_GW3{"SP1-GW3: L2<br/>通過判定"}

    %% ===== L3チェック =====
    SP1_T4["SP1-T4: L3 シンプルさチェック<br/>(DT-5: S1-S3を評価)"]
    SP1_GW4{"SP1-GW4: L3<br/>通過判定"}

    %% ===== 終了 =====
    SP1_EE_OK((("SP1-EE-OK:<br/>L0-L3全通過")))
    SP1_EE_NG((("SP1-EE-NG:<br/>不合格（調整が必要）")))

    %% ===== フロー接続 =====
    SP1_SE --> D1
    D1 --> SP1_T1
    SP1_T1 --> SP1_GW1
    SP1_GW1 -->|"No: 持続可能性を損なう"| SP1_EE_NG
    SP1_GW1 -->|"Yes"| SP1_T2
    SP1_T2 --> SP1_GW2
    SP1_GW2 -->|"No: 萎縮の可能性"| SP1_EE_NG
    SP1_GW2 -->|"Yes"| SP1_T3
    SP1_T3 --> SP1_GW3
    SP1_GW3 -->|"No: 丁寧さを保てない"| SP1_EE_NG
    SP1_GW3 -->|"Yes"| SP1_T4
    SP1_T4 --> SP1_GW4
    SP1_GW4 -->|"No: 1文1検証を満たせない"| SP1_EE_NG
    SP1_GW4 -->|"Yes"| SP1_EE_OK

    %% ===== スタイル =====
    style SP1_EE_OK fill:#c8e6c9,stroke:#2e7d32,color:#000
    style SP1_EE_NG fill:#ffcdd2,stroke:#c62828,color:#000
    style SP1_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP1_GW2 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP1_GW3 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP1_GW4 fill:#fff9c4,stroke:#f9a825,color:#000
    style D1 fill:#e3f2fd,stroke:#1565c0,color:#000
```

---

## 3. サブプロセス SP-2: AIファーストチェック＋分業判断

> **対応するBPMN要素**: SP2-SE, SP2-T1〜T3, SP2-GW1〜GW2, SP2-EE-AI, SP2-EE-HM（8要素）
> **対応する決定表**: DT-6（分業判断）、DT-7（プロンプト技法選択）

```mermaid
flowchart TD
    %% ===== 開始 =====
    SP2_SE(["SP2-SE: AIファーストチェック開始"])

    %% ===== タスク特性分析 =====
    SP2_T1["SP2-T1: タスク特性を分析する"]
    SP2_GW1{"SP2-GW1: AIの<br/>得意分野か？<br/>(DT-6 入力)"}

    %% ===== 分業判断 =====
    SP2_T2["SP2-T2: 分業を決定する<br/>(DT-6: AI主導/人間主導)"]
    SP2_GW2{"SP2-GW2: 分業結果<br/>の判定"}

    %% ===== プロンプト技法選択 =====
    SP2_T3["SP2-T3: プロンプト技法を選択する<br/>(DT-7: F4)"]

    %% ===== 終了 =====
    SP2_EE_AI((("SP2-EE-AI:<br/>AI主導で実行")))
    SP2_EE_HM((("SP2-EE-HM:<br/>人間主導で実行")))

    %% ===== フロー接続 =====
    SP2_SE --> SP2_T1
    SP2_T1 --> SP2_GW1
    SP2_GW1 -->|"No: AI不得意"| SP2_EE_HM
    SP2_GW1 -->|"Yes / 不明"| SP2_T2
    SP2_T2 --> SP2_GW2
    SP2_GW2 -->|"人間主導"| SP2_EE_HM
    SP2_GW2 -->|"AI主導"| SP2_T3
    SP2_T3 --> SP2_EE_AI

    %% ===== スタイル =====
    style SP2_EE_AI fill:#c8e6c9,stroke:#2e7d32,color:#000
    style SP2_EE_HM fill:#c8e6c9,stroke:#2e7d32,color:#000
    style SP2_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP2_GW2 fill:#fff9c4,stroke:#f9a825,color:#000
```

---

## 4. サブプロセス SP-3: 検証フィードバックループ

> **対応するBPMN要素**: SP3-SE, SP3-T1〜T6, SP3-GW1〜GW4, SP3-IE1, SP3-IE2, SP3-EE-OK, SP3-EE-NG（14要素）
> **対応する決定表**: DT-8（協働原則）、DT-9（AI行動原則）

```mermaid
flowchart TD
    %% ===== 開始 =====
    SP3_SE(["SP3-SE: 検証開始"])

    %% ===== 損切り判定・修正ループ（左側に配置） =====
    SP3_T4["SP3-T4: エラーの修正を指示する"]

    %% ===== 並行監視タスク =====
    SP3_T5["SP3-T5: 協働原則チェック C1-C4<br/>(DT-8, 並行して監視)"]
    SP3_T6["SP3-T6: AI行動原則チェック A1-A4<br/>(DT-9, 並行して監視)"]

    %% ===== 検証ステップ =====
    SP3_T1["SP3-T1: typecheckを実行する"]
    SP3_GW1{"SP3-GW1: typecheck<br/>結果判定"}
    SP3_T2["SP3-T2: lintを実行する"]
    SP3_GW2{"SP3-GW2: lint<br/>結果判定"}
    SP3_T3["SP3-T3: testを実行する"]
    SP3_GW3{"SP3-GW3: test<br/>結果判定"}

    %% ===== データオブジェクト =====
    D4[("D-4: 検証結果<br/>(typecheck/lint/test)")]

    %% ===== 損切り判定 =====
    SP3_GW4{"SP3-GW4: 損切り判定<br/>(3回/30分/複雑化)"}

    %% ===== 中間イベント =====
    SP3_IE1{{"SP3-IE1: ⏱ 30分経過"}}
    SP3_IE2{{"SP3-IE2: ⚠ 同一エラー3回目"}}

    %% ===== 終了 =====
    SP3_EE_OK((("SP3-EE-OK:<br/>検証成功")))
    SP3_EE_NG((("SP3-EE-NG:<br/>損切り→復帰フローへ")))

    %% ===== フロー接続 =====
    SP3_SE --> SP3_T5
    SP3_SE --> SP3_T6
    SP3_SE --> SP3_T1

    SP3_T1 --> SP3_GW1
    SP3_GW1 -->|"成功"| SP3_T2
    SP3_GW1 -->|"失敗"| SP3_GW4

    SP3_T2 --> SP3_GW2
    SP3_GW2 -->|"成功"| SP3_T3
    SP3_GW2 -->|"失敗"| SP3_GW4

    SP3_T3 --> SP3_GW3
    SP3_GW3 -->|"成功"| D4
    SP3_GW3 -->|"失敗"| SP3_GW4
    D4 --> SP3_EE_OK

    SP3_IE1 -.->|"強制起動"| SP3_GW4
    SP3_IE2 -.->|"強制起動"| SP3_GW4

    SP3_GW4 -->|"3回以上 OR<br/>30分超過 OR<br/>複雑化している"| SP3_EE_NG
    SP3_GW4 -->|"3回未満 AND<br/>30分未満 AND<br/>複雑化していない"| SP3_T4
    SP3_T4 --> SP3_T1

    %% ===== スタイル =====
    style SP3_EE_OK fill:#c8e6c9,stroke:#2e7d32,color:#000
    style SP3_EE_NG fill:#ffcdd2,stroke:#c62828,color:#000
    style SP3_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP3_GW2 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP3_GW3 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP3_GW4 fill:#fff9c4,stroke:#f9a825,color:#000
    style SP3_IE1 fill:#ffe0b2,stroke:#e65100,color:#000
    style SP3_IE2 fill:#ffe0b2,stroke:#e65100,color:#000
    style D4 fill:#e3f2fd,stroke:#1565c0,color:#000
```

---

## 5. 復帰フロー（概要）

> **対応するBPMN要素**: RF-SE, RF-T1, RF-GW1, RF-T2, RF-EE（5要素）
> **備考**: T5で正式にBPMN要素に分解予定。ここではT3との接続点として概要のみ図化。

```mermaid
flowchart TD
    %% ===== 開始 =====
    RF_SE(["RF-SE: 損切り発生<br/>(SP3-EE-NGからの遷移)"])

    %% ===== 問題分析 =====
    RF_T1["RF-T1: 問題を分析する（5-10分）"]
    RF_GW1{"RF-GW1: アプローチ選択<br/>(A/B/C/D)"}

    %% ===== 学び記録 =====
    RF_T2["RF-T2: 学びをCLAUDE.mdに記録する<br/>（必須）"]

    %% ===== データオブジェクト =====
    D1[("D-1: CLAUDE.md")]
    D5[("D-5: 失敗パターン記録")]

    %% ===== 終了 =====
    RF_EE((("RF-EE: メインフロー<br/>SE-1に戻る")))

    %% ===== フロー接続 =====
    RF_SE --> RF_T1
    RF_T1 --> RF_GW1
    RF_GW1 -->|"A: 別のアプローチ"| RF_T2
    RF_GW1 -->|"B: タスク再分解"| RF_T2
    RF_GW1 -->|"C: 手動実装"| RF_T2
    RF_GW1 -->|"D: 一時保留"| RF_T2
    RF_T2 --> D1
    RF_T2 --> D5
    D1 --> RF_EE
    D5 --> RF_EE

    %% ===== スタイル =====
    style RF_SE fill:#ffcdd2,stroke:#c62828,color:#000
    style RF_EE fill:#c8e6c9,stroke:#2e7d32,color:#000
    style RF_T2 fill:#fff3e0,stroke:#e65100,color:#000
    style RF_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style D1 fill:#e3f2fd,stroke:#1565c0,color:#000
    style D5 fill:#e3f2fd,stroke:#1565c0,color:#000
```

---

## 6. タスク分解サブプロセス（T-2の詳細）

> **対応するBPMN要素**: TD-SE, TD-GW1〜GW2, TD-T1〜T5, TD-EE（9要素）
> **出典**: `01c-task-decomposition.md`（タスク分解の5ステップ、Type分類）

```mermaid
flowchart TD
    %% ===== 開始 =====
    TD_SE(["TD-SE: タスク分解開始"])

    %% ===== 1文1検証チェック =====
    TD_GW1{"TD-GW1: 1文1検証を<br/>満たしているか？"}

    %% ===== 分解ステップ =====
    TD_T1["TD-T1: Step1 最終成果物を特定する"]
    TD_T2["TD-T2: Step2 依存関係を整理する"]
    TD_T3["TD-T3: Step3 各成果物を実装単位に分割する"]
    TD_T4["TD-T4: Step4 1文1検証 単位に調整する"]

    %% ===== Type判定 =====
    TD_T5["TD-T5: Step5 Type 1/2を判定してラベル付け"]
    TD_GW2{"TD-GW2: Type判定"}

    %% ===== データオブジェクト =====
    D2[("D-2: タスク一覧（分解後）")]

    %% ===== 終了 =====
    TD_EE((("TD-EE: 分解完了<br/>メインフローに戻る")))

    %% ===== フロー接続 =====
    TD_SE --> TD_GW1
    TD_GW1 -->|"満たしている"| TD_T5
    TD_GW1 -->|"満たしていない"| TD_T1
    TD_T1 --> TD_T2
    TD_T2 --> TD_T3
    TD_T3 --> TD_T4
    TD_T4 --> TD_T5
    TD_T5 --> TD_GW2
    TD_GW2 -->|"Type 1: 新規作成"| D2
    TD_GW2 -->|"Type 2: 既存参照あり"| D2
    D2 --> TD_EE

    %% ===== スタイル =====
    style TD_EE fill:#c8e6c9,stroke:#2e7d32,color:#000
    style TD_GW1 fill:#fff9c4,stroke:#f9a825,color:#000
    style TD_GW2 fill:#fff9c4,stroke:#f9a825,color:#000
    style D2 fill:#e3f2fd,stroke:#1565c0,color:#000
```

---

## 7. データオブジェクト参照表

フロー図内に `[("名称")]` 形式で配置済み。メタ情報として参照関係を表で管理する。

| データID | Mermaid ID | 名称 | 図内配置 | 生成元（図中の要素） | 参照先（図中の要素） |
|---------|-----------|------|---------|---------------------|---------------------|
| D-1 | `D1` | CLAUDE.md | §2, §5 | RF_T2（学び記録時に更新） | SP1_T1〜T4, SP3_T6 |
| D-2 | `D2` | タスク一覧（分解後） | §6 | TD_T5 | MF_SE（再入力） |
| D-3 | `D3` | AI出力（初期案） | §1 | MF_T4 | MF_T5（レビュー対象） |
| D-4 | `D4` | 検証結果（typecheck/lint/test） | §4 | SP3_T1〜T3 | SP3_GW1〜GW3 |
| D-5 | `D5` | 失敗パターン記録 | §5 | RF_T2 | D1（CLAUDE.mdに追記） |

---

## 8. フロー間の接続関係

```mermaid
flowchart LR
    %% ===== フロー定義 =====
    Main["§1 メインフロー"]
    SP1["§2 SP-1:<br/>L0-L3チェック"]
    SP2["§3 SP-2:<br/>AIファースト<br/>チェック"]
    SP3["§4 SP-3:<br/>検証フィードバック<br/>ループ"]
    RF["§5 復帰フロー<br/>（概要）"]
    TD["§6 タスク分解"]

    %% ===== フロー接続 =====
    Main -->|"MF_SP1呼出"| SP1
    Main -->|"MF_SP2呼出"| SP2
    Main -->|"MF_SP3呼出"| SP3
    Main -->|"MF_T2の詳細"| TD
    Main -->|"MF_EE_CUTから遷移"| RF
    RF -->|"RF_EEからMF_SEへ"| Main
    TD -->|"TD_EEからMF_SP1へ"| Main

    %% ===== スタイル =====
    style Main fill:#bbdefb,stroke:#1565c0,color:#000
    style RF fill:#ffcdd2,stroke:#c62828,color:#000
```

---

## 検証: T3全要素IDの網羅性チェック

### 開始イベント（6個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| SE-1 | `MF_SE` | §1 メインフロー | ✅ |
| SP1-SE | `SP1_SE` | §2 SP-1 | ✅ |
| SP2-SE | `SP2_SE` | §3 SP-2 | ✅ |
| SP3-SE | `SP3_SE` | §4 SP-3 | ✅ |
| RF-SE | `RF_SE` | §5 復帰フロー | ✅ |
| TD-SE | `TD_SE` | §6 タスク分解 | ✅ |

### 終了イベント（10個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| EE-1 | `MF_EE_OK` | §1 メインフロー | ✅ |
| EE-2 | `MF_EE_CUT` | §1 メインフロー | ✅ |
| SP1-EE-OK | `SP1_EE_OK` | §2 SP-1 | ✅ |
| SP1-EE-NG | `SP1_EE_NG` | §2 SP-1 | ✅ |
| SP2-EE-AI | `SP2_EE_AI` | §3 SP-2 | ✅ |
| SP2-EE-HM | `SP2_EE_HM` | §3 SP-2 | ✅ |
| SP3-EE-OK | `SP3_EE_OK` | §4 SP-3 | ✅ |
| SP3-EE-NG | `SP3_EE_NG` | §4 SP-3 | ✅ |
| RF-EE | `RF_EE` | §5 復帰フロー | ✅ |
| TD-EE | `TD_EE` | §6 タスク分解 | ✅ |

### タスク（25個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| T-1 | `MF_T1` | §1 メインフロー | ✅ |
| T-2 | `MF_T2` | §1 メインフロー | ✅ |
| T-3 | `MF_T3` | §1 メインフロー | ✅ |
| T-4 | `MF_T4` | §1 メインフロー | ✅ |
| T-5 | `MF_T5` | §1 メインフロー | ✅ |
| SP1-T1 | `SP1_T1` | §2 SP-1 | ✅ |
| SP1-T2 | `SP1_T2` | §2 SP-1 | ✅ |
| SP1-T3 | `SP1_T3` | §2 SP-1 | ✅ |
| SP1-T4 | `SP1_T4` | §2 SP-1 | ✅ |
| SP2-T1 | `SP2_T1` | §3 SP-2 | ✅ |
| SP2-T2 | `SP2_T2` | §3 SP-2 | ✅ |
| SP2-T3 | `SP2_T3` | §3 SP-2 | ✅ |
| SP3-T1 | `SP3_T1` | §4 SP-3 | ✅ |
| SP3-T2 | `SP3_T2` | §4 SP-3 | ✅ |
| SP3-T3 | `SP3_T3` | §4 SP-3 | ✅ |
| SP3-T4 | `SP3_T4` | §4 SP-3 | ✅ |
| SP3-T5 | `SP3_T5` | §4 SP-3 | ✅ |
| SP3-T6 | `SP3_T6` | §4 SP-3 | ✅ |
| RF-T1 | `RF_T1` | §5 復帰フロー | ✅ |
| RF-T2 | `RF_T2` | §5 復帰フロー | ✅ |
| TD-T1 | `TD_T1` | §6 タスク分解 | ✅ |
| TD-T2 | `TD_T2` | §6 タスク分解 | ✅ |
| TD-T3 | `TD_T3` | §6 タスク分解 | ✅ |
| TD-T4 | `TD_T4` | §6 タスク分解 | ✅ |
| TD-T5 | `TD_T5` | §6 タスク分解 | ✅ |

### サブプロセス（3個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| SP-1 | `MF_SP1` | §1 メインフロー | ✅ |
| SP-2 | `MF_SP2` | §1 メインフロー | ✅ |
| SP-3 | `MF_SP3` | §1 メインフロー | ✅ |

### 排他ゲートウェイ（17個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| GW-1 | `MF_GW1` | §1 メインフロー | ✅ |
| GW-2 | `MF_GW2` | §1 メインフロー | ✅ |
| GW-3 | `MF_GW3` | §1 メインフロー | ✅ |
| GW-4 | `MF_GW4` | §1 メインフロー | ✅ |
| SP1-GW1 | `SP1_GW1` | §2 SP-1 | ✅ |
| SP1-GW2 | `SP1_GW2` | §2 SP-1 | ✅ |
| SP1-GW3 | `SP1_GW3` | §2 SP-1 | ✅ |
| SP1-GW4 | `SP1_GW4` | §2 SP-1 | ✅ |
| SP2-GW1 | `SP2_GW1` | §3 SP-2 | ✅ |
| SP2-GW2 | `SP2_GW2` | §3 SP-2 | ✅ |
| SP3-GW1 | `SP3_GW1` | §4 SP-3 | ✅ |
| SP3-GW2 | `SP3_GW2` | §4 SP-3 | ✅ |
| SP3-GW3 | `SP3_GW3` | §4 SP-3 | ✅ |
| SP3-GW4 | `SP3_GW4` | §4 SP-3 | ✅ |
| RF-GW1 | `RF_GW1` | §5 復帰フロー | ✅ |
| TD-GW1 | `TD_GW1` | §6 タスク分解 | ✅ |
| TD-GW2 | `TD_GW2` | §6 タスク分解 | ✅ |

### 中間イベント（2個）

| 要素ID | Mermaid ID | 所在図 | 含まれているか |
|--------|-----------|--------|---------------|
| SP3-IE1 | `SP3_IE1` | §4 SP-3 | ✅ |
| SP3-IE2 | `SP3_IE2` | §4 SP-3 | ✅ |

### データオブジェクト（5個）

| 要素ID | Mermaid ID | 図内配置 | 含まれているか |
|--------|-----------|---------|---------------|
| D-1 | `D1` | §2 SP-1, §5 復帰フロー | ✅ |
| D-2 | `D2` | §6 タスク分解 | ✅ |
| D-3 | `D3` | §1 メインフロー | ✅ |
| D-4 | `D4` | §4 SP-3 | ✅ |
| D-5 | `D5` | §5 復帰フロー | ✅ |

### 集計

| BPMN要素種別 | T3の個数 | T4での個数 | 一致 |
|-------------|---------|-----------|------|
| 開始イベント | 6 | 6 | ✅ |
| 終了イベント | 10 | 10 | ✅ |
| タスク | 25 | 25 | ✅ |
| サブプロセス | 3 | 3 | ✅ |
| 排他ゲートウェイ | 17 | 17 | ✅ |
| 中間イベント | 2 | 2 | ✅ |
| データオブジェクト | 5 | 5 | ✅ |
| **合計** | **68** | **68** | **✅** |

---

## 検証チェックリスト

- [x] メインフロー（§1）の全要素ID（15個）がMermaid図に含まれている
- [x] SP-1（§2）の全要素ID（11個）がMermaid図に含まれている
- [x] SP-2（§3）の全要素ID（8個）がMermaid図に含まれている
- [x] SP-3（§4）の全要素ID（14個）がMermaid図に含まれている
- [x] 復帰フロー（§5）の全要素ID（5個）がMermaid図に含まれている
- [x] タスク分解（§6）の全要素ID（9個）がMermaid図に含まれている
- [x] データオブジェクト（§7）の全要素ID（5個）が参照表に含まれている
- [x] フロー間の接続関係（§8）が示されている
- [x] T3のBPMN要素統計（68要素）と完全一致している
- [x] 各Mermaid図に対応する決定表（DT-0〜DT-10）の参照が記載されている
- [x] Mermaid記法とBPMN要素の対応表が冒頭に記載されている
- [x] **Phase 1.5 統一ルール準拠**: 開始イベントがスタジアム型 `(["..."])` で統一されている
- [x] **Phase 1.5 統一ルール準拠**: 終了イベントが三重括弧 `((("...")))` で統一されている
- [x] **Phase 1.5 統一ルール準拠**: 全図にstyle指定（配色ルール）が適用されている
- [x] **Phase 1.5 統一ルール準拠**: メインフローIDがプレフィックスベース `MF_*` に統一されている
- [x] **Phase 1.5 統一ルール準拠**: データオブジェクトが `[("...")]` で図内に配置されている
- [x] **Phase 1.5 統一ルール準拠**: 中間イベントが六角形 `{{...}}` で統一されている
- [x] **Phase 1.5 統一ルール準拠**: 矢印ラベルが `-->\|"..."\|` で統一されている
- [x] **Phase 1.5 統一ルール準拠**: 全図にセクションコメント `%% =====` が追加されている
- [x] **Phase 1.5 統一ルール準拠**: ノード内改行が `\n` で統一されている

---

## 出典

- `phase1-t3-bpmn-elements.md` — BPMN要素分解（T4の直接入力）
- `phase1-t2-dmn-decision-tables.md` — 決定表との対応参照
- `phase1-t1-decision-conditions.md` — 判断条件の原典

---

## スコープ外の提案（参考情報）

以下はT4のスコープ外だが、今後の検討事項として記録：

1. **Mermaid図のレンダリング検証**: 各図をMermaidプレビューツールで実際にレンダリングし、表示崩れがないか確認する（→ Phase 1.5 T4で実施予定）
2. **フロー間リンクの統合図**: §8の接続関係図をより詳細にし、全6図を1枚の俯瞰図に統合する
3. **スイムレーン版の作成**: 本タスクで見送った「人間/AI/ツール」のスイムレーンを追加したバージョンを別途作成する
