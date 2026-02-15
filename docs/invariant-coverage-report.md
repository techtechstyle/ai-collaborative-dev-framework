# T16: 不変条件カバレッジレポート

> **タスク**: T7の全69個の不変条件に対して、XState実装のどのテストケースで検証されているかの対応表を作成し、カバレッジを確認する
> **入力**: `phase1-t7-invariants.md`（69不変条件）、T9〜T15の全9テストファイル（161テスト + 1 skip）
> **検証方法**: 全69不変条件に対して「カバー済み/未カバー（理由付き）」の分類が完了していること
> **自信度**: 確実（各テストファイルのdescribe/itブロックを全数照合）

---

## サマリー

| 指標 | 値 |
|------|-----|
| 不変条件 総数 | 69 |
| カバー済み | 62 |
| 未カバー（理由付きスキップ） | 7 |
| **カバレッジ率（全体）** | **89.9%** |
| **カバレッジ率（実装対象のみ）** | **100%** |

### 未カバー7件の内訳

| 理由 | 件数 | 対象 |
|------|------|------|
| TD（タスク分解）マシン未実装（Phase 2） | 4 | INV-TD1〜TD4 |
| TD依存（TD未実装に起因） | 2 | INV-DO2, INV-CF5 |
| 型レベル制約（ステートマシンテスト対象外） | 1 | INV-CA3 |

---

## テストファイル一覧

| # | ファイル | テスト数 | 主な検証対象 |
|---|---------|---------|-------------|
| 1 | `l0l4-hierarchy.test.ts` | 29 | INV-H1〜H5, INV-SP1-1〜SP1-5 |
| 2 | `sp2-division.test.ts` | 22 | INV-SP2-1〜SP2-4 |
| 3 | `verification-loop.test.ts` | 28 | INV-SP3-1〜SP3-5, INV-LC5(SP-3レベル) |
| 4 | `losscut-judgment.test.ts` | 19 | INV-LC1〜LC5 |
| 5 | `recovery-flow.test.ts` | 29 | INV-RF1〜RF6, INV-ES1〜ES3 |
| 6 | `main-flow.test.ts` | 14 | INV-MF1〜MF6, INV-CF1, INV-CF4 |
| 7 | `main-flow-t10d.test.ts` | 5 | INV-CA2 |
| 8 | `integrated-flow.test.ts` | 7+1skip | INV-CF1〜CF5 |
| 9 | `invariants-integration.test.ts` | 8 | INV-BL1〜BL3, INV-DO3〜DO6, INV-CA1 |
| | **合計** | **161+1skip** | **14グループ / 69不変条件** |

---

## 1. INV-H: 階層不変条件群（5/5 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-H1** | L0〜L4の評価順序は逆転しない | ✅ カバー | l0l4-hierarchy.test.ts | 「全通過パスでL0→L1→L2→L3の順序が保証される」「l1Check状態でL0のイベントを送信しても状態が変わらない」 | stateHistory配列で遷移順序を記録・検証。逆方向イベント送信の無視を確認 |
| **INV-H2** | 上位レベルでNoの場合、下位レベルの評価は行われない | ✅ カバー | l0l4-hierarchy.test.ts | 「L0不合格時、L1〜L3の評価結果がnullのまま」「L1不合格時、L2〜L3の評価結果がnullのまま」「failed状態でイベントを送信しても受け付けない」 | context.evaluationResultsの各レベルがnull（未評価）であることを検証 |
| **INV-H3** | L4はL0〜L3すべて通過後にのみ適用される | ✅ カバー | l0l4-hierarchy.test.ts | 「全通過時のoutputでallPassed=trueが返る」「一部不合格時のoutputでallPassed=falseが返る」「passed最終状態に到達するには4レベル全通過が必要」 | output.allPassedフラグの検証。SP-2への遷移条件として機能 |
| **INV-H4** | 競合発生時、上位レベルの判断が常に優先される | ✅ カバー | l0l4-hierarchy.test.ts | 「L0不合格 → L0の判断が最終結果を決定する」「L1不合格 → L2以降は未評価」 | 逐次評価の構造的保証。上位Noで即failed到達を検証 |
| **INV-H5** | Bright Linesは全レベルに先行する | ✅ カバー | l0l4-hierarchy.test.ts + main-flow.test.ts | 「SP-1マシンの初期状態はl0Check」「SP-1マシンの状態にbrightLinesCheckは存在しない」+ INV-MF1テスト | SP-1はBL後に呼び出される前提を構造検証。MainFlowのbrightLinesCheck→sp1Check順序で補完 |

---

## 2. INV-MF: メインフロー不変条件群（6/6 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-MF1** | すべてのタスクはBright Linesチェック通過後に実行 | ✅ カバー | main-flow.test.ts | 「初期状態はbrightLinesCheckである」「BRIGHT_LINES_PASS → sp1Checkに遷移する」 | 初期状態とBL通過後の遷移先を検証 |
| **INV-MF2** | Bright Lines違反検出時、是正完了まで進行しない | ✅ カバー | main-flow.test.ts | 「BRIGHT_LINES_FAIL → brightLinesFixに遷移する」「brightLinesFix → BRIGHT_LINES_FIXED → brightLinesCheckに戻る」 | 是正→再チェックループの存在を検証 |
| **INV-MF3** | SP-1不合格の場合、SP-2には進めない | ✅ カバー | main-flow.test.ts | 「SP-1不合格 → taskAdjustmentに遷移」「taskAdjustment → ADJUSTMENT_DONE → sp1Checkに戻る」 | SP-1 failモックでSP-2未到達（sp2Result=null）を検証 |
| **INV-MF4** | AI適用パスで人間レビュー（T-5）は省略不可 | ✅ カバー | main-flow.test.ts + invariants-integration.test.ts | 「AI主導 → aiExecution → humanReview → sp3Verification」+「#3: AI実行後T-5経由必須」 | aiExecution後の遷移先がhumanReview（sp3Verificationではない）を検証 |
| **INV-MF5** | 両パスがSP-3を通過する | ✅ カバー | main-flow.test.ts + invariants-integration.test.ts | 「人間主導パスでもSP-3を経由して完了する」+「#2: 人間主導パスもSP-3経由」 | 人間主導パスでsp3Result≠nullを検証 |
| **INV-MF6** | 終了はtaskCompletedまたはrecoveryExitの2つのみ | ✅ カバー | main-flow.test.ts | 「SP-3成功 → taskCompleted（EE-1）」「SP-3損切り → RF → recoveryExit（EE-2）」 | 2パターンの終了を明示的に検証。output.completionTypeで区別 |

---

## 3. INV-SP1: L0-L3チェック不変条件群（5/5 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-SP1-1** | L0チェックは常に最初に実行 | ✅ カバー | l0l4-hierarchy.test.ts | 「マシン開始時の初期状態がl0Checkである」 | 初期状態の検証 |
| **INV-SP1-2** | 各レベルは直前レベル通過後にのみ実行 | ✅ カバー | l0l4-hierarchy.test.ts | 「L0通過 → L1に遷移」「L0→L1→L2」「L0→L1→L2→L3」「L0→L1→L2→L3→passed」 | 各段階の逐次遷移を4テストで網羅 |
| **INV-SP1-3** | いずれかNoで即failed、他レベルはスキップ | ✅ カバー | l0l4-hierarchy.test.ts | 「L0不合格 → 即座にfailed」「L1不合格 → failed」「L2不合格 → failed」「L3不合格 → failed」 | 全4レベルの不合格パスを個別検証 |
| **INV-SP1-4** | 結果はpassedまたはfailedの2つのみ | ✅ カバー | l0l4-hierarchy.test.ts | 「全通過時: passed最終状態に到達する」「不合格時: failed最終状態に到達する」「マシン定義に最終状態はpassedとfailedの2つのみ」 | output検証 + マシン定義構造検査（finalState抽出） |
| **INV-SP1-5** | 「要対応」が1つでもあれば不合格 | ✅ カバー | l0l4-hierarchy.test.ts | 「issues配列に1つの項目 → failed」「issues配列に複数の項目 → failed」「issues配列が空 → 次レベルに遷移」 | issues配列の長さとpassed/failedの対応を検証 |

---

## 4. INV-SP2: AIファーストチェック不変条件群（4/4 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-SP2-1** | タスク特性分析は分業判断の前に実行 | ✅ カバー | sp2-division.test.ts | 「初期状態はanalyzingTask」「analyzingTaskでDECIDE_DIVISIONは受け付けない」「analyzingTaskでSELECT_PROMPTは受け付けない」 | 初期状態検証 + 後続イベントの無視確認（3テスト） |
| **INV-SP2-2** | AI主導時、プロンプト技法選択は省略不可 | ✅ カバー | sp2-division.test.ts | 「AI主導判定後、selectingPromptを経由する」「selectingPromptでSELECT_PROMPTを送ると初めてaiLedExitに遷移」 | selectingPrompt状態の存在 + done前の中間状態確認 |
| **INV-SP2-3** | 結果はAI主導/人間主導の二択のみ | ✅ カバー | sp2-division.test.ts | 「AI主導パス: aiLedExit」「人間主導パス（GW1経由）: humanLedExit」「人間主導パス（GW2経由）: humanLedExit」「最終状態は2つのみ」 | 全パスの最終状態検証 + マシン定義構造検査 |
| **INV-SP2-4** | DT-6の各ルールは排他的条件 | ✅ カバー | sp2-division.test.ts | 「initialDraft → GW1通過」「styleUnification → GW1通過」「gapDetection → GW1通過」「designDecision → 即humanLedExit」「domainSpecific → 即humanLedExit」「unknown → GW1通過」 | 全6タスク特性の分岐を個別検証（排他性） |

---

## 5. INV-SP3: 検証フィードバックループ不変条件群（5/5 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-SP3-1** | 検証順序はtypecheck→lint→testで固定 | ✅ カバー | verification-loop.test.ts | 「初期状態がtypecheck」「全通過でtypecheck→lint→test→verificationPassed」「typecheckでLINT_COMPLETEは無視」「修正後ループでもtypecheckから再開」 | stateHistory配列 + 不正イベント無視 + ループ後の再開状態 |
| **INV-SP3-2** | 前段成功なしに次段に進めない | ✅ カバー | verification-loop.test.ts | 「typecheck成功 → lint」「typecheck失敗 → LC」「lint成功 → test」「lint失敗 → LC」 | 成功/失敗両パスの遷移先を各段階で検証 |
| **INV-SP3-3** | 協働/AI行動原則チェックが常に並行監視 | ✅ カバー | verification-loop.test.ts + invariants-integration.test.ts | 「typecheck/lint/test各状態のentryに原則チェックアクション含む」+「#8: DT-8/DT-9が各3回ずつ呼出し」 | マシン定義構造検査（entryアクション） + provide()カウンター注入 |
| **INV-SP3-4** | 検証失敗時にLCが起動 | ✅ カバー | verification-loop.test.ts | 「typecheck失敗 → LC」「lint失敗 → LC」「test失敗 → LC」「errorCount増加」「errorHistory記録」 | 全3段階の失敗→LC遷移 + context更新 |
| **INV-SP3-5** | 正常終了は全3段階成功の場合のみ | ✅ カバー | verification-loop.test.ts | 「全通過 → verificationPassed」「全通過時のoutputが正しい」「verificationPassedへの遷移元はtest状態のみ」 | output検証 + マシン定義構造検査（遷移元限定） |

---

## 6. INV-LC: 損切り判断不変条件群（5/5 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-LC1** | エラー状態記録は損切り判定の前に実行 | ✅ カバー | losscut-judgment.test.ts | 「LCの初期状態がrecordErrorState」「ERROR_STATE_RECORDED後にcheck3Timesに遷移」「recordErrorStateを経由せず直接遷移不可」 | 初期状態 + イベント順序の強制 |
| **INV-LC2** | 4条件はOR条件で評価（1つでも該当で損切り） | ✅ カバー | losscut-judgment.test.ts | 「条件1: errorCount>=3」「条件2: 30分経過」「条件3: コード複雑度増加」「条件4: 同一エラー再発」 | 各条件を個別にtrueにして損切り確定を検証（4テスト） |
| **INV-LC3** | 全条件非該当の場合のみ修正継続 | ✅ カバー | losscut-judgment.test.ts | 「全条件No → continueFix」「デフォルトコンテキスト → continueFix」 | 4条件全Noでcontinuefixを検証 |
| **INV-LC4** | 評価順序は3回→30分→複雑化→再発、短絡評価 | ✅ カバー | losscut-judgment.test.ts | 「3回Yesで後続不評価」「3回No・30分Yes」「3回No・30分No・複雑化Yes」「全NoからYes再発」「マシン定義の状態チェーン順序」 | 各条件の短絡評価 + マシン定義構造検査（checkState順序） |
| **INV-LC5** | 30分経過/3回目で損切り判断を強制起動 | ✅ カバー | losscut-judgment.test.ts + verification-loop.test.ts | LC内: 「errorCount=3で即損切り」「30分経過で損切り」 SP-3レベル: 「30分経過でtypecheckからLC強制遷移」「30分経過でlintからLC強制遷移」「29分59秒では未発火」 | LCガード検証 + SP-3の`after(1800000)`タイマー検証（vi.useFakeTimers） |

---

## 7. INV-RF: 復帰フロー不変条件群（6/6 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-RF1** | 問題分析はRFの最初のステップとして必須 | ✅ カバー | recovery-flow.test.ts | 「初期状態がproblemAnalysis.verbalizeProblem」「Step 1はT1→T2→T3の順序」「verbalizeProblemでCAUSE_ANALYZEDは無視」 | 初期状態 + 逐次遷移順序 + 不正イベント無視 |
| **INV-RF2** | CLAUDE.md記録は全パスから必須（省略不可） | ✅ カバー | recovery-flow.test.ts | 「アプローチA完了 → recordToClaudeMd」「アプローチB完了 → recordToClaudeMd」「アプローチC完了 → recordToClaudeMd」「アプローチD(ES-ESC経由) → recordToClaudeMd」「GW1直行 → recordToClaudeMd」 | 全5パス（A/B/C/D-ESC/D-GW1）でrecordToClaudeMd到達を個別検証 |
| **INV-RF3** | 回避策文書化はRF-T9の後に実行 | ✅ カバー | recovery-flow.test.ts | 「CLAUDE_MD_RECORDED → documentWorkaround」「WORKAROUND_DOCUMENTED → teamShareDecision」 | recordToClaudeMd→documentWorkaroundの遷移順序を検証 |
| **INV-RF4** | 復帰フローは常にrecoveryCompleteで終了 | ✅ カバー | recovery-flow.test.ts | 「チーム共有なし → recoveryComplete」「チーム共有あり → shareWithTeam → recoveryComplete」「output === { recovered: true }」 | 両パスでrecoveryComplete(final)到達 + output検証 |
| **INV-RF5** | アプローチ選択はStep 1完了後にのみ実行 | ✅ カバー | recovery-flow.test.ts | 「Step 1完了後にapproachSelectionに到達」「Step 1途中からapproachSelection関連に遷移不可」 | Step 1完了後の到達 + 途中からの不正遷移不可を検証 |
| **INV-RF6** | アプローチは4パターン排他選択 | ✅ カバー | recovery-flow.test.ts | 「A → directResolution」「B → redecompose」「C → resetContext」「null → escalationJudgment（D）」「approachSelectionのalways遷移最後がフォールバック」 | 全4パターンの遷移先 + マシン定義構造検査（always遷移の最後がガードなしフォールバック） |

---

## 8. INV-ES: エスカレーション判断不変条件群（3/3 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-ES1** | GW1（即座）がGW2（30分以内）に先行 | ✅ カバー | recovery-flow.test.ts | 「ESの初期状態がcheckImmediate」「マシン定義でcheckImmediate→check30Minの順序」 | 初期状態 + マシン定義構造検査（状態定義順序） |
| **INV-ES2** | セキュリティ等で即座にエスカレーション | ✅ カバー | recovery-flow.test.ts | 「セキュリティ問題あり → executeImmediate → escalationConfirmed」「データ損失リスクあり → 即座にエスカレーション確定」 | hasSecurityIssue/hasDataLossRiskフラグでGW1即発火を検証 |
| **INV-ES3** | 結果は「実施」or「自力」の2つのみ | ✅ カバー | recovery-flow.test.ts | 「ESのfinal状態がescalationConfirmedとselfResolutionの2つのみ」「ES-ESC → consultTeam」「ES-SELF → approachSelection」 | マシン定義構造検査（finalState抽出） + 両パスの遷移先検証 |

---

## 9. INV-TD: タスク分解不変条件群（0/4 — 全件スキップ）

| ID | 不変条件 | カバー状態 | 理由 |
|----|---------|-----------|------|
| **INV-TD1** | 「1文1検証を満たしていない」場合にのみ5ステップ実行 | ⏭️ スキップ | TDマシン未実装（Phase 2拡張候補） |
| **INV-TD2** | 分解の5ステップの実行順序は固定 | ⏭️ スキップ | TDマシン未実装（Phase 2拡張候補） |
| **INV-TD3** | Type判定は最終ステップとして必ず実行 | ⏭️ スキップ | TDマシン未実装（Phase 2拡張候補） |
| **INV-TD4** | 分解完了後は必ずメインフローに戻る | ⏭️ スキップ | TDマシン未実装（Phase 2拡張候補） |

> **備考**: integrated-flow.test.ts #8 が `it.skip` として登録済み。Phase 2でTDマシン実装後にテスト追加予定。

---

## 10. INV-DT: 決定表不変条件群（8/8 カバー）

| ID | 不変条件 | カバー状態 | 検証元テスト | 検証方式 |
|----|---------|-----------|------------|---------|
| **INV-DT1** | DT-0は他の全DTに先行して評価 | ✅ カバー | main-flow.test.ts INV-MF1 | brightLinesCheck（DT-0対応）が初期状態であり、SP-1（DT-1対応）に先行することを検証 |
| **INV-DT2** | DT-0のヒットポリシーF（BL1-BL4いずれかで即停止） | ✅ カバー | main-flow.test.ts INV-MF2 | BRIGHT_LINES_FAIL（1件のviolation）で即brightLinesFix遷移。是正ループの存在を検証 |
| **INV-DT3** | DT-1のルール評価順序L0→L1→L2→L3、最初のNoで分岐 | ✅ カバー | l0l4-hierarchy.test.ts INV-H1 + INV-SP1-2 + INV-SP1-3 | stateHistory配列でL0→L1→L2→L3順序を検証。各レベルNo時のfailed即遷移 |
| **INV-DT4** | DT-2〜DT-5のヒットポリシーC（1つでも要対応で不合格） | ✅ カバー | l0l4-hierarchy.test.ts INV-SP1-5 | issues配列に1つ/複数の項目がある場合のfailed遷移。空配列で次レベル遷移 |
| **INV-DT5** | DT-6のヒットポリシーU（タスク特性で一意に1ルール合致） | ✅ カバー | sp2-division.test.ts INV-SP2-4 | 全6タスク特性の個別分岐検証。各特性が一意の遷移先を持つことを確認 |
| **INV-DT6** | DT-7はDT-6でAI主導の場合にのみ参照 | ✅ カバー | sp2-division.test.ts INV-SP2-2 | AI主導判定→selectingPrompt（DT-7対応）経由を検証。人間主導パスはselectingPromptをスキップ |
| **INV-DT7** | DT-8/DT-9は常に適用（全ルール常時評価） | ✅ カバー | verification-loop.test.ts INV-SP3-3 + invariants-integration.test.ts #8 | entryアクション構造検査 + provide()カウンター注入で各3回呼出しを定量検証 |
| **INV-DT8** | DT-10で上位レベルが常に優先 | ✅ カバー | l0l4-hierarchy.test.ts INV-H4 | 逐次評価構造により上位Noが下位に先行。L0不合格で即failed（下位未評価）の構造的保証 |

---

## 11. INV-DO: データオブジェクト不変条件群（5/6 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-DO1** | CLAUDE.mdはRF-T9で必ず更新 | ✅ カバー | recovery-flow.test.ts INV-RF2 + integrated-flow.test.ts #6 | RF全5パスでrecordToClaudeMd到達を検証。#6ではRF実マシンでの通過確認 |
| **INV-DO2** | タスク一覧はTD-T5の出力としてのみ生成 | ⏭️ スキップ | — | TDマシン未実装（INV-TD依存） |
| **INV-DO3** | AI出力はT-5入力として必ず参照 | ✅ カバー | invariants-integration.test.ts #4 | T-4後: isAiGenerated=true, humanReviewed=false → T-5後: humanReviewed=true を段階的に検証 |
| **INV-DO4** | 検証結果はSP3で生成されGW判定入力に使用 | ✅ カバー | invariants-integration.test.ts #5 | typecheck失敗→errorHistory蓄積→修正後全通過→sp3Result転写を検証 |
| **INV-DO5** | エラー状態記録はLC判定入力として使用 | ✅ カバー | invariants-integration.test.ts #6 | 3回失敗→errorHistory 3件蓄積→LC損切り→lastError/errorHistory/sp3Result転写を検証 |
| **INV-DO6** | 失敗パターン記録はCLAUDE.mdに追記統合 | ✅ カバー | invariants-integration.test.ts #7 | provide()でrecordFailurePattern/documentWorkaroundに順序追跡注入。callOrder検証 |

---

## 12. INV-CF: フロー間接続不変条件群（4/5 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-CF1** | SP-1→SP-2→SP-3の呼出順序は固定 | ✅ カバー | main-flow.test.ts INV-CF1 + integrated-flow.test.ts #2,#3 | outputトラッキングマシンでexecutionOrder検証。#2: 実SP-1→実SP-2。#3: 実SP-1→実SP-2→実SP-3全通過E2E |
| **INV-CF2** | SP-3検証失敗はLCにのみ遷移 | ✅ カバー | integrated-flow.test.ts #4,#5 | #4: SP-3失敗→LC→修正継続→ループ→全通過→taskCompleted。#5: 3回失敗→LC損切り→recoveryFlow遷移 |
| **INV-CF3** | LC→RFへの遷移は一方向 | ✅ カバー | integrated-flow.test.ts #6,#7 | #6: SP-3損切り→RF実マシン起動→recoveryExit。#7: 全実マシンE2Eでの一方向遷移確認 |
| **INV-CF4** | RF完了後はメインフローSE-1に戻る | ✅ カバー | main-flow.test.ts INV-CF4 + integrated-flow.test.ts #6,#7 | RF完了→recoveryExit（MainFlowのfinal）。context.recoveryResult転写確認 |
| **INV-CF5** | TDはT-2からのみ呼び出される | ⏭️ スキップ | integrated-flow.test.ts #8 (it.skip) | TDマシン未実装（INV-TD依存）。skip登録済み |

---

## 13. INV-BL: Bright Lines不変条件群（4/4 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-BL1** | 人間の最終判断権は常に維持 | ✅ カバー | invariants-integration.test.ts #1 | AIパスでhumanReview状態に必ず到達。approved判断がcontext.executionResult.humanReviewedに反映 |
| **INV-BL2** | 本番適用前に検証ステップが必ず存在 | ✅ カバー | invariants-integration.test.ts #2 | 人間主導パスでもSP-3に到達。SP-3経由なしにtaskCompleted不到達を検証 |
| **INV-BL3** | コード採用前に人間による理解確認が存在 | ✅ カバー | invariants-integration.test.ts #3 | EXECUTE_AI後の遷移先がhumanReview（sp3Verificationではない）。T-4→T-5→SP-3の順序保証 |
| **INV-BL4** | AI出力に対する独立検証が常に実施 | ✅ カバー | integrated-flow.test.ts #3 + verification-loop.test.ts INV-SP3-1 | #3: 実SP-3でtypecheck→lint→test全通過E2E。INV-SP3-1: 検証順序固定 |

---

## 14. INV-CA: 協働・AI行動原則不変条件群（2/3 カバー）

| ID | 不変条件 | カバー状態 | テストファイル | テストケース | 検証方式 |
|----|---------|-----------|--------------|-------------|---------|
| **INV-CA1** | DT-8で要対応の場合、是正まで継続されない | ✅ カバー | invariants-integration.test.ts #8 + verification-loop.test.ts INV-SP3-3 | provide()カウンター注入: collaborationCheckCount=3, aiPrincipleCheckCount=3。entryアクション構造検査 |
| **INV-CA2** | DT-9 A4違反検出時、DT-0に戻る | ✅ カバー | main-flow-t10d.test.ts | 「principleViolation=true → brightLinesCheck」「復帰後BRIGHT_LINES_PASSで再度SP-1に進む」「SP-3成功/損切りパスとの分岐区別」「context転写」 | SP-3のonDone第2分岐（principleViolation）でbrightLinesCheckに遷移。5テストで検証 |
| **INV-CA3** | AI出力には常に検証方法と自信度が明示 | ⏭️ スキップ | — | 型レベル制約（A3原則）。ステートマシン遷移テストの対象外。TypeScriptの型定義で保証 |

---

## カバレッジ統計

### グループ別カバレッジ

| グループ | 総数 | カバー | スキップ | カバー率 |
|---------|------|--------|---------|---------|
| INV-H（階層） | 5 | 5 | 0 | 100% |
| INV-MF（メインフロー） | 6 | 6 | 0 | 100% |
| INV-SP1（L0-L3チェック） | 5 | 5 | 0 | 100% |
| INV-SP2（AIファースト） | 4 | 4 | 0 | 100% |
| INV-SP3（検証ループ） | 5 | 5 | 0 | 100% |
| INV-LC（損切り判断） | 5 | 5 | 0 | 100% |
| INV-RF（復帰フロー） | 6 | 6 | 0 | 100% |
| INV-ES（エスカレーション） | 3 | 3 | 0 | 100% |
| INV-TD（タスク分解） | 4 | 0 | 4 | 0% |
| INV-DT（決定表） | 8 | 8 | 0 | 100% |
| INV-DO（データオブジェクト） | 6 | 5 | 1 | 83% |
| INV-CF（フロー間接続） | 5 | 4 | 1 | 80% |
| INV-BL（Bright Lines） | 4 | 4 | 0 | 100% |
| INV-CA（協働原則） | 3 | 2 | 1 | 67% |
| **合計** | **69** | **62** | **7** | **89.9%** |

### 分類別カバレッジ

| 分類 | 総数 | カバー | スキップ | カバー率 |
|------|------|--------|---------|---------|
| 順序不変条件 | 22 | 20 | 2 | 91% |
| 状態不変条件 | 26 | 22 | 4 | 85% |
| 存在不変条件 | 11 | 10 | 1 | 91% |
| 階層不変条件 | 4 | 4 | 0 | 100% |
| データ不変条件 | 6 | 6 | 0 | 100% |
| **合計** | **69** | **62** | **7** | **89.9%** |

### 検証方式の分布

| 検証方式 | 使用回数 | 説明 |
|---------|---------|------|
| 状態遷移検証 | 47 | `expect(snapshot.value).toBe(...)` による直接的な状態確認 |
| マシン定義構造検査 | 9 | `machine.config.states` の直接検査（final状態数、状態順序等） |
| context/output検証 | 18 | `snapshot.context` / `snapshot.output` のデータ整合性確認 |
| provide()注入検証 | 6 | `provide()` でアクションにカウンター/トラッカーを注入 |
| タイマー検証 | 3 | `vi.useFakeTimers()` + `vi.advanceTimersByTime()` |

---

## スキップ不変条件の詳細理由

### INV-TD1〜TD4（TDマシン未実装）

TDサブプロセスはPhase 2の実装対象として位置づけられている。現在のXState実装ではMainFlowの`taskAdjustment`状態にTD invokeは未定義。`integrated-flow.test.ts #8`が`it.skip`として登録済みであり、TD実装後にテスト追加する計画が明示されている。

### INV-DO2（TD依存）

「タスク一覧（D-2）はTD-T5の出力としてのみ生成される」はTDマシンの内部動作に関する不変条件。TDマシン未実装のため検証不可。INV-TD1〜TD4と同時に対応予定。

### INV-CF5（TD依存）

「TDはT-2からのみ呼び出される」はMainFlow内のTD invoke配置に関する不変条件。TDマシン未実装のため、T-2（taskAdjustment）にinvokeが定義されていない。`integrated-flow.test.ts #8`の`it.skip`コメントに「Step D完了後の拡張候補」と記載済み。

### INV-CA3（型レベル制約）

「AI出力には常に検証方法と自信度が明示される」はA3原則に基づく制約。これはステートマシンの遷移で保証するものではなく、TypeScriptの型定義（`SP3OutputExtended`の`lastError`型、`CheckResult`の`error`型等）で構造的に保証される。ステートマシンテストのスコープ外として正当にスキップ。

---

## 出典

- `phase1-t7-invariants.md` — 69不変条件の定義元
- `src/machines/__tests__/l0l4-hierarchy.test.ts` — T9: 29テスト
- `src/machines/__tests__/sp2-division.test.ts` — T10a: 22テスト
- `src/machines/__tests__/verification-loop.test.ts` — T11: 28テスト
- `src/machines/__tests__/losscut-judgment.test.ts` — T12: 19テスト
- `src/machines/__tests__/recovery-flow.test.ts` — T13: 29テスト
- `src/machines/__tests__/main-flow.test.ts` — T10c: 14テスト
- `src/machines/__tests__/main-flow-t10d.test.ts` — T10d: 5テスト
- `src/machines/__tests__/integrated-flow.test.ts` — T14: 7テスト + 1 skip
- `src/machines/__tests__/invariants-integration.test.ts` — T15: 8テスト
