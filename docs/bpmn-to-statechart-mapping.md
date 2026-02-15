# T4: BPMN→ステートチャート構造マッピング

> **タスク**: フェーズ1のBPMN要素を、ステートチャートの構成要素（状態、遷移、アクション、ガード）へのマッピングルールとして整理する
> **検証方法**: T3の68個のBPMN要素のうち、データオブジェクト（D-1〜D-5の5個）を除く63要素すべてにマッピングルールが定義されていること。加えてT5の33要素（データオブジェクト3個除く）にも同ルールが適用可能であること
> **自信度**: おそらく（一部のBPMN要素は1対1でマッピングできない可能性あり）
> **入力**: `phase1-t3-bpmn-elements.md`、`phase1-t5-losscutrecovery-bpmn.md`、`docs/xstate-concepts-mapping.md`（T2成果物）

---

## 1. マッピングルール一覧

BPMN要素種別ごとに、ステートチャート（XState v5）への変換ルールを定義する。

---

### ルール MR-1: 開始イベント → 初期状態指定

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 開始イベント（○） |
| **ステートチャート要素** | `initial` プロパティ |
| **変換規則** | 開始イベントの直後の要素を `initial` で指定する。開始イベント自体は独立した状態にならない |
| **XState記法** | `initial: '最初の状態名'` |

**変換例**:
```
BPMN:  SE-1 → GW-1（Bright Linesチェック）
XState: initial: 'brightLinesCheck'
```

**設計判断**: 開始イベントがサブプロセス内にある場合（SP1-SE, SP2-SE等）、そのサブプロセスに対応するネスト状態の `initial` として設定する。

**適用対象**: SE-1, SP1-SE, SP2-SE, SP3-SE, RF-SE, TD-SE, LC-SE, ES-SE（計8要素、T3: 6 + T5: 2。RF-SEはT3概要とT5詳細で共通）

---

### ルール MR-2: 終了イベント → 最終状態

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 終了イベント（◎） |
| **ステートチャート要素** | 最終状態 `{ type: 'final' }` |
| **変換規則** | 終了イベントを `{ type: 'final' }` の状態として定義する。サブプロセス内の最終状態が達成されると `onDone` で親に通知される |
| **XState記法** | `stateName: { type: 'final' }` |

**変換例**:
```
BPMN:  EE-1（タスク完了）
XState: taskComplete: { type: 'final' }
```

**サブプロセス内の複数終了イベント**: SP-1のように2つの終了（SP1-EE-OK / SP1-EE-NG）がある場合、それぞれ別の最終状態として定義し、`onDone` のガード条件または `output` で親が分岐する。

```typescript
// SP-1 の2つの終了イベント
states: {
  passed: { type: 'final' },  // SP1-EE-OK
  failed: { type: 'final' },  // SP1-EE-NG
}
```

**適用対象**:

| 分類 | 要素ID | 要素数 |
|------|--------|--------|
| T3メインフロー | EE-1, EE-2 | 2 |
| T3 SP-1 | SP1-EE-OK, SP1-EE-NG | 2 |
| T3 SP-2 | SP2-EE-AI, SP2-EE-HM | 2 |
| T3 SP-3 | SP3-EE-OK, SP3-EE-NG | 2 |
| T3 RF/TD | RF-EE, TD-EE | 2 |
| T5 LC | LC-EE-CONT, LC-EE-CUT | 2 |
| T5 ES | ES-EE-ESC, ES-EE-SELF | 2 |
| **合計** | | **14** |

（T3: 10 + T5: 5 = 15 だが、SP3-EE-NG → LC-EE-CUT の詳細化により、SP3-EE-NGはT5のLC-EE-CUTに置き換えられる。カウントは各要素IDをユニークに数える）

---

### ルール MR-3: タスク → 状態 + entryアクション

| 項目 | 内容 |
|------|------|
| **BPMN要素** | タスク（□） |
| **ステートチャート要素** | 状態 + `entry` アクション |
| **変換規則** | タスクを1つの状態として定義し、タスクの処理内容を `entry` アクションとして記述する。タスク完了は次の遷移を引き起こすイベントとして表現する |
| **XState記法** | `stateName: { entry: [{ type: 'actionName' }], on: { EVENT: 'nextState' } }` |

**変換例**:
```
BPMN:  SP1-T1（L0 持続可能性チェック）
XState:
  l0Check: {
    entry: [{ type: 'evaluateL0' }],
    on: {
      EVALUATION_COMPLETE: [
        { guard: 'isL0Pass', target: 'l1Check' },
        { target: 'failed' },
      ],
    },
  }
```

**タスクの分類とアクション設計**:

| タスクの性質 | entryアクションの内容 | 完了イベントの種類 |
|------------|--------------------|--------------------|
| チェック・評価系 | 評価ロジックを実行 | 評価結果イベント（PASS/FAIL） |
| 実行・処理系 | 処理を実行（API呼出等） | 完了イベント |
| 記録系 | データをcontextに書き込む | 書込完了イベント |
| 分析系 | 入力データを分析 | 分析完了イベント |

**適用対象**:

| 分類 | 要素ID | 要素数 |
|------|--------|--------|
| T3メイン | T-1〜T-5 | 5 |
| T3 SP-1 | SP1-T1〜T4 | 4 |
| T3 SP-2 | SP2-T1〜T3 | 3 |
| T3 SP-3 | SP3-T1〜T6 | 6 |
| T3 RF概要 | RF-T1, RF-T2（T5で詳細化） | (T5にカウント) |
| T3 TD | TD-T1〜T5 | 5 |
| T5 LC | LC-T1〜T2 | 2 |
| T5 RF | RF-T1〜T11 | 11 |
| T5 ES | ES-T1〜T2 | 2 |
| **合計** | | **38** |

---

### ルール MR-4: サブプロセス → ネスト状態（compound state）

| 項目 | 内容 |
|------|------|
| **BPMN要素** | サブプロセス（□⊞） |
| **ステートチャート要素** | ネスト状態（compound state） |
| **変換規則** | サブプロセスを子状態を持つ複合状態として定義する。サブプロセスの開始イベントは子の `initial`、終了イベントは子の最終状態に対応する。親状態は `onDone` で子の完了を受け取る |
| **XState記法** | `subProcess: { initial: 'first', states: { ... }, onDone: 'nextState' }` |

**変換例**:
```
BPMN:  SP-1（L0-L3チェック）
XState:
  l0l3Check: {
    initial: 'l0Check',
    states: {
      l0Check: { ... },
      l1Check: { ... },
      l2Check: { ... },
      l3Check: { ... },
      passed: { type: 'final' },
      failed: { type: 'final' },
    },
    onDone: [
      { guard: 'isSP1Passed', target: 'aiFirstCheck' },
      { target: 'l0l3Adjust' },
    ],
  }
```

**ネスト深度の制約**: リスク分析に基づき、3〜4層に抑える。

```
層1: メインフロー
 └ 層2: SP-1, SP-2, SP-3, LC, RF
    └ 層3: SP-3内の並行状態リージョン、RF内のES
       └ 層4: （これ以上は避ける）
```

**適用対象**: SP-1, SP-2, SP-3, RF-SP1（計4要素。T3: 3 + T5: 1）

---

### ルール MR-5: 排他ゲートウェイ（XOR） → ガード付き遷移の配列

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 排他ゲートウェイ（◇ XOR） |
| **ステートチャート要素** | ガード付き遷移の配列 |
| **変換規則** | ゲートウェイを独立した状態にせず、直前のタスク状態からのガード付き遷移として表現する。ガードの配列は上から順に評価され、最初にtrueになったものが採用される（ヒットポリシーFと同等） |
| **XState記法** | `on: { EVENT: [{ guard: 'cond1', target: 'a' }, { target: 'b' }] }` |

**重要**: これがBPMN→ステートチャート変換で最も本質的な構造変化である。BPMNではゲートウェイは独立したノードだが、ステートチャートでは遷移の条件分岐として吸収される。

**変換例**:
```
BPMN:  SP1-T1 → SP1-GW1 --[Yes]-→ SP1-T2
                          --[No]--→ SP1-EE-NG

XState:
  l0Check: {
    on: {
      EVALUATION_COMPLETE: [
        { guard: 'isL0Pass', target: 'l1Check' },  // GW1 Yes
        { target: 'failed' },                        // GW1 No
      ],
    },
  }
```

**ゲートウェイのパターン分類**:

| パターン | BPMN構造 | XState変換 | 適用例 |
|---------|---------|-----------|--------|
| 二分岐（Yes/No） | タスク→GW→2パス | ガード1つ+フォールバック | SP1-GW1〜GW4, LC-GW1〜GW4 |
| 多分岐（3つ以上） | タスク→GW→N個のパス | ガードN-1個+フォールバック | RF-GW2（4パス: A/B/C/D） |
| ループ分岐 | タスク→GW→元のタスクに戻る | 自己遷移または親状態への遷移 | GW-2（L0-L3不合格→調整→再チェック） |
| 逐次ゲートウェイ | GW→GW→GW→... | 状態チェーンまたはガード合成 | LC-GW1→GW2→GW3→GW4（短絡評価） |

**逐次ゲートウェイの変換選択肢**（Step Bで確定）:

損切り判断のLC-GW1〜GW4のような逐次ゲートウェイは2つの実装方法がある:

- **方法A: 状態チェーン** — 各ゲートウェイを独立した状態にする（INV-LC4の短絡評価を自然に表現）
- **方法B: ガード合成** — `or` ガードで1つの遷移にまとめる（シンプルだが短絡評価の可視性が低い）

```typescript
// 方法A: 状態チェーン（短絡評価が明示的）
check3Times: {
  always: [
    { guard: 'isErrorCount3OrMore', target: 'lossCut' },
    { target: 'check30Min' },
  ],
},
check30Min: {
  always: [
    { guard: 'isOver30Min', target: 'lossCut' },
    { target: 'checkComplexity' },
  ],
},
// ...

// 方法B: ガード合成（シンプル）
always: [
  {
    guard: { type: 'or', guards: ['isErrorCount3OrMore', 'isOver30Min', ...] },
    target: 'lossCut',
  },
  { target: 'continueFix' },
],
```

**適用対象**:

| 分類 | 要素ID | 要素数 |
|------|--------|--------|
| T3メイン | GW-1〜GW-4 | 4 |
| T3 SP-1 | SP1-GW1〜GW4 | 4 |
| T3 SP-2 | SP2-GW1〜GW2 | 2 |
| T3 SP-3 | SP3-GW1〜GW4 | 4 |
| T3 RF/TD | RF-GW1, TD-GW1〜GW2 | 3 |
| T5 LC | LC-GW1〜GW4 | 4 |
| T5 RF | RF-GW1〜GW3（T3のRF-GW1を含む） | 3 |
| T5 ES | ES-GW1〜GW2 | 2 |
| **合計** | | **26** |

---

### ルール MR-6: 中間イベント（タイマー） → 遅延遷移

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 中間イベント（タイマー）（○⏱） |
| **ステートチャート要素** | 遅延遷移 `after` |
| **変換規則** | タイマーイベントを、対象状態の `after` プロパティで定義する。タイマー発火時の遷移先はガード付きにできる |
| **XState記法** | `after: { milliseconds: 'targetState' }` |

**変換例**:
```
BPMN:  SP3-IE1（30分経過）→ LC-SE
XState:
  fixing: {
    after: {
      1800000: 'lossCutJudgment',  // 30分 = 1,800,000ms
    },
  }
```

**T3での検証結果**: `vi.useFakeTimers()` + `vi.advanceTimersByTime()` で遅延遷移のテストが可能であることを確認済み。

**適用対象**: SP3-IE1（計1要素）

---

### ルール MR-7: 中間イベント（エラー） → コンテキスト条件 + ガード

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 中間イベント（エラー）（○!） |
| **ステートチャート要素** | `context` でカウンタ管理 + ガード条件 |
| **変換規則** | エラー回数のカウンタを `context` で管理し、閾値到達時にガードが true になることで遷移する。BPMNの中間エラーイベントのような「割り込み」は、イベントハンドラ内のガードで表現する |
| **XState記法** | `guard: ({ context }) => context.errorCount >= 3` |

**変換例**:
```
BPMN:  SP3-IE2（同一エラー3回目）→ LC-SE
XState:
  on: {
    CHECK_FAIL: [
      {
        guard: ({ context }) => context.errorCount >= 2,
        target: 'lossCutJudgment',
        actions: assign({ errorCount: ({ context }) => context.errorCount + 1 }),
      },
      {
        actions: assign({ errorCount: ({ context }) => context.errorCount + 1 }),
      },
    ],
  }
```

**T3での検証結果**: `context` + `guard` + `assign` の組合せで3回ルールの実装が可能であることを確認済み。

**適用対象**: SP3-IE2（計1要素）

---

### ルール MR-8: 並列ゲートウェイ（AND） → 並行状態

| 項目 | 内容 |
|------|------|
| **BPMN要素** | 並列ゲートウェイ（◇+）— T3/T5では明示的なBPMN要素IDはないが、SP-3の構造として存在 |
| **ステートチャート要素** | 並行状態 `type: 'parallel'` |
| **変換規則** | 並行して実行されるフローを、並行状態の各リージョンとして定義する |
| **XState記法** | `type: 'parallel', states: { regionA: { ... }, regionB: { ... } }` |

**変換例**:
```
BPMN:  SP-3内の検証実行（逐次）と原則監視（並行）
XState:
  sp3: {
    type: 'parallel',
    states: {
      verification: { ... },      // 検証の逐次実行
      principleMonitor: { ... },   // 協働原則＋AI行動原則の監視
    },
  }
```

**適用対象**: SP-3の並行構造（INV-SP3-3で定義された並行監視。独立したBPMN要素IDはないが、構造変換として必要）

---

### ルール MR-9: データオブジェクト → context

| 項目 | 内容 |
|------|------|
| **BPMN要素** | データオブジェクト（📄） |
| **ステートチャート要素** | `context` プロパティ |
| **変換規則** | フロー内で参照・生成されるデータを `context` のフィールドとして定義する。データの更新は `assign` アクションで行う |
| **XState記法** | `context: { fieldName: initialValue }` |

**注記**: データオブジェクトはマッピングルールのカウント対象外（検証条件より除外）だが、Step Bの仕様定義で必要になるため、ルールとして記載する。

**適用対象**: D-1〜D-8（計8要素、カウント対象外）

---

## 2. 全要素マッピング表

T3（68要素）+ T5（36要素）の全BPMN要素に対するマッピングを以下に示す。

### 2.1 開始イベント（MR-1適用）

| # | 要素ID | 名称 | 所属 | マッピング先（initial） |
|---|--------|------|------|----------------------|
| 1 | SE-1 | タスクを受け取る | メイン | `mainFlow.initial: 'brightLinesCheck'` |
| 2 | SP1-SE | L0-L3チェック開始 | SP-1 | `l0l3Check.initial: 'l0Check'` |
| 3 | SP2-SE | AIファーストチェック開始 | SP-2 | `aiFirstCheck.initial: 'taskAnalysis'` |
| 4 | SP3-SE | 検証開始 | SP-3 | `verificationLoop.initial`（並行状態のため各リージョンで設定） |
| 5 | RF-SE | 復帰フロー開始 | RF | `recoveryFlow.initial: 'problemDescription'` |
| 6 | TD-SE | タスク分解開始 | TD | `taskDecomposition.initial: 'checkOneVerify'` |
| 7 | LC-SE | 損切り判断開始 | LC | `lossCutJudgment.initial: 'recordErrorState'` |
| 8 | ES-SE | エスカレーション判断開始 | ES | `escalationJudgment.initial: 'immediateCheck'` |

**カウント: 8要素 → MR-1適用済み**

### 2.2 終了イベント（MR-2適用）

| # | 要素ID | 名称 | 所属 | マッピング先（final state） |
|---|--------|------|------|--------------------------|
| 1 | EE-1 | タスク完了 | メイン | `taskComplete: { type: 'final' }` |
| 2 | EE-2 | 損切り→復帰へ | メイン | `lossCutExit: { type: 'final' }` |
| 3 | SP1-EE-OK | L0-L3全通過 | SP-1 | `passed: { type: 'final' }` |
| 4 | SP1-EE-NG | L0-L3不合格 | SP-1 | `failed: { type: 'final' }` |
| 5 | SP2-EE-AI | AI主導で実行 | SP-2 | `aiLead: { type: 'final' }` |
| 6 | SP2-EE-HM | 人間主導で実行 | SP-2 | `humanLead: { type: 'final' }` |
| 7 | SP3-EE-OK | 検証成功 | SP-3 | `verificationPassed: { type: 'final' }` |
| 8 | SP3-EE-NG | 損切り→復帰 | SP-3 | `verificationFailed: { type: 'final' }` |
| 9 | RF-EE | メインフローSE-1に戻る | RF | `returnToMain: { type: 'final' }` |
| 10 | TD-EE | 分解完了 | TD | `decompositionDone: { type: 'final' }` |
| 11 | LC-EE-CONT | 修正継続 | LC | `continueFix: { type: 'final' }` |
| 12 | LC-EE-CUT | 損切り確定 | LC | `lossCutConfirmed: { type: 'final' }` |
| 13 | ES-EE-ESC | エスカレーション実施 | ES | `escalate: { type: 'final' }` |
| 14 | ES-EE-SELF | 自力で対応 | ES | `handleSelf: { type: 'final' }` |

**カウント: 14要素 → MR-2適用済み**

### 2.3 タスク（MR-3適用）

| # | 要素ID | 名称 | 所属 | 状態名（候補） | entryアクション（候補） |
|---|--------|------|------|--------------|---------------------|
| 1 | T-1 | Bright Lines違反を是正 | メイン | `brightLinesFix` | `fixBrightLinesViolation` |
| 2 | T-2 | L0-L3を満たす形に調整 | メイン | `l0l3Adjust` | `adjustForL0L3` |
| 3 | T-3 | 人間主導で実行 | メイン | `humanExecution` | `executeHumanLead` |
| 4 | T-4 | AIに初期案を生成させる | メイン | `aiGeneration` | `generateWithAI` |
| 5 | T-5 | 人間がAI出力をレビュー | メイン | `humanReview` | `reviewAIOutput` |
| 6 | SP1-T1 | L0持続可能性チェック | SP-1 | `l0Check` | `evaluateL0` |
| 7 | SP1-T2 | L1心理的安全性チェック | SP-1 | `l1Check` | `evaluateL1` |
| 8 | SP1-T3 | L2急がば回れチェック | SP-1 | `l2Check` | `evaluateL2` |
| 9 | SP1-T4 | L3シンプルさチェック | SP-1 | `l3Check` | `evaluateL3` |
| 10 | SP2-T1 | タスク特性を分析 | SP-2 | `taskAnalysis` | `analyzeTaskCharacteristics` |
| 11 | SP2-T2 | 分業を決定 | SP-2 | `divisionDecision` | `decideDivision` |
| 12 | SP2-T3 | プロンプト技法を選択 | SP-2 | `promptSelection` | `selectPromptTechnique` |
| 13 | SP3-T1 | typecheckを実行 | SP-3 | `typecheck` | `runTypecheck` |
| 14 | SP3-T2 | lintを実行 | SP-3 | `lint` | `runLint` |
| 15 | SP3-T3 | testを実行 | SP-3 | `test` | `runTest` |
| 16 | SP3-T4 | エラーの修正を指示 | SP-3 | （LC-T2に統合） | — |
| 17 | SP3-T5 | 協働原則チェック | SP-3 | `collaborationCheck` | `checkCollaborationPrinciples` |
| 18 | SP3-T6 | AI行動原則チェック | SP-3 | `aiPrincipleCheck` | `checkAIPrinciples` |
| 19 | TD-T1 | 最終成果物を特定 | TD | `identifyDeliverables` | `identifyFinalDeliverables` |
| 20 | TD-T2 | 依存関係を整理 | TD | `organizeDependencies` | `organizeDeps` |
| 21 | TD-T3 | 実装単位に分割 | TD | `splitToUnits` | `splitToImplementationUnits` |
| 22 | TD-T4 | 1文1検証単位に調整 | TD | `adjustToOneVerify` | `adjustToOneStatementOneVerify` |
| 23 | TD-T5 | Type 1/2を判定 | TD | `typeClassification` | `classifyType` |
| 24 | LC-T1 | エラー状態を記録 | LC | `recordErrorState` | `recordCurrentErrorState` |
| 25 | LC-T2 | エラーの修正を指示 | LC | `issueFix` | `issueFixInstruction` |
| 26 | RF-T1 | 問題を言語化 | RF | `problemDescription` | `describeTheProblem` |
| 27 | RF-T2 | 原因を分析 | RF | `causeAnalysis` | `analyzeCause` |
| 28 | RF-T3 | 問題の本質を特定 | RF | `essenceIdentification` | `identifyEssence` |
| 29 | RF-T4 | A: 人間が直接解決 | RF | `approachA_solve` | `solveDirectly` |
| 30 | RF-T5 | A: AIに解説を依頼 | RF | `approachA_explain` | `requestExplanation` |
| 31 | RF-T6 | B: 問題を再分解 | RF | `approachB_redecompose` | `redecomposeTask` |
| 32 | RF-T7 | C: コンテキストをリセット | RF | `approachC_reset` | `resetContext` |
| 33 | RF-T8 | D: チームメンバーに相談 | RF | `approachD_consult` | `consultTeam` |
| 34 | RF-T9 | CLAUDE.mdに失敗パターンを追記 | RF | `recordToClaudeMd` | `updateClaudeMd` |
| 35 | RF-T10 | 次回の回避策を明記 | RF | `recordAvoidance` | `recordAvoidanceStrategy` |
| 36 | RF-T11 | チームに共有 | RF | `shareWithTeam` | `shareWithTeam` |
| 37 | ES-T1 | 即座にエスカレーション実施 | ES | `escalateImmediately` | `executeImmediateEscalation` |
| 38 | ES-T2 | 30分以内にエスカレーション検討 | ES | `considerEscalation` | `considerEscalationWithin30Min` |

**カウント: 38要素 → MR-3適用済み**

（SP3-T4はT5のLC-T2に統合されているため、実質37ユニーク要素。ただし要素IDとしては両方カウント）

### 2.4 サブプロセス（MR-4適用）

| # | 要素ID | 名称 | マッピング先 | 子状態の数 |
|---|--------|------|------------|----------|
| 1 | SP-1 | L0-L3チェック | `l0l3Check`（compound state） | 6（l0〜l3 + passed + failed） |
| 2 | SP-2 | AIファーストチェック+分業判断 | `aiFirstCheck`（compound state） | 5（analysis + decision + prompt + aiLead + humanLead） |
| 3 | SP-3 | 検証フィードバックループ | `verificationLoop`（parallel state） | 2リージョン（verification + principleMonitor） |
| 4 | RF-SP1 | エスカレーション判断 | `escalationJudgment`（compound state） | 4（immediateCheck + consider + escalate + handleSelf） |

**カウント: 4要素 → MR-4適用済み**

### 2.5 排他ゲートウェイ（MR-5適用）

| # | 要素ID | 名称 | 吸収先の状態 | ガード条件（候補） |
|---|--------|------|-----------|----------------|
| 1 | GW-1 | Bright Lines違反チェック | `brightLinesCheck` | `hasBrightLinesViolation` |
| 2 | GW-2 | L0-L3通過判定 | `l0l3Check`（onDone） | `isSP1Passed` |
| 3 | GW-3 | AIファースト適用判定 | `aiFirstCheck`（onDone） | `isAiLead` |
| 4 | GW-4 | 検証結果判定 | `verificationLoop`（onDone） | `isVerificationPassed` |
| 5 | SP1-GW1 | L0通過判定 | `l0Check` | `isL0Pass` |
| 6 | SP1-GW2 | L1通過判定 | `l1Check` | `isL1Pass` |
| 7 | SP1-GW3 | L2通過判定 | `l2Check` | `isL2Pass` |
| 8 | SP1-GW4 | L3通過判定 | `l3Check` | `isL3Pass` |
| 9 | SP2-GW1 | AIの得意分野か | `taskAnalysis` | `isAiSuitable` |
| 10 | SP2-GW2 | 分業結果の判定 | `divisionDecision` | `isAiLead` |
| 11 | SP3-GW1 | typecheck結果判定 | `typecheck` | `isTypecheckPass` |
| 12 | SP3-GW2 | lint結果判定 | `lint` | `isLintPass` |
| 13 | SP3-GW3 | test結果判定 | `test` | `isTestPass` |
| 14 | SP3-GW4 | 損切り判定 | （LC-GW1〜GW4に詳細化） | — |
| 15 | RF-GW1 | エスカレーション必要性判定 | `essenceIdentification` | `needsEscalation` |
| 16 | TD-GW1 | 1文1検証を満たしているか | `checkOneVerify` | `meetsOneStatementOneVerify` |
| 17 | TD-GW2 | Type判定 | `typeClassification` | `isType1` |
| 18 | LC-GW1 | 3回ルール判定 | `recordErrorState` / 独立状態 | `isErrorCount3OrMore` |
| 19 | LC-GW2 | 時間ルール判定 | （逐次or合成、Step Bで確定） | `isOver30Min` |
| 20 | LC-GW3 | 複雑化ルール判定 | （逐次or合成、Step Bで確定） | `isGrowingComplexity` |
| 21 | LC-GW4 | 再発ルール判定 | （逐次or合成、Step Bで確定） | `isRecurringError` |
| 22 | RF-GW2 | アプローチ選択 | `essenceIdentification` | `isApproachA` / `B` / `C` / `D` |
| 23 | RF-GW3 | チーム共有が必要か | `recordAvoidance` | `needsTeamSharing` |
| 24 | ES-GW1 | 即座にエスカレーション必要か | `immediateCheck` | `needsImmediateEscalation` |
| 25 | ES-GW2 | 30分以内にエスカレーション検討か | `considerEscalation` | `shouldConsiderEscalation` |

**カウント: 25要素 → MR-5適用済み**

（SP3-GW4はLC-GW1〜GW4に詳細化されたため、ユニーク要素としてはSP3-GW4を含めて25。ただしSP3-GW4単体のマッピングルールは「LC全体に委譲」とする）

### 2.6 中間イベント（MR-6, MR-7適用）

| # | 要素ID | 名称 | 適用ルール | マッピング先 |
|---|--------|------|----------|------------|
| 1 | SP3-IE1 | 30分経過 | MR-6 | `after: { 1800000: 'lossCutJudgment' }` |
| 2 | SP3-IE2 | 同一エラー3回目 | MR-7 | `guard: isErrorCount3OrMore` + `context.errorCount` |

**カウント: 2要素 → MR-6, MR-7適用済み**

---

## 3. マッピングカバレッジ検証

### 3.1 T3 BPMN要素（68個）のカバレッジ

| BPMN要素種別 | T3の要素数 | マッピング済み | 適用ルール |
|-------------|----------|-------------|-----------|
| 開始イベント | 6 | 6 | MR-1 |
| 終了イベント | 10 | 10 | MR-2 |
| タスク | 25 | 25 | MR-3 |
| サブプロセス | 3 | 3 | MR-4 |
| 排他ゲートウェイ | 17 | 17 | MR-5 |
| 中間イベント | 2 | 2 | MR-6, MR-7 |
| データオブジェクト | 5 | (対象外) | MR-9 |
| **合計** | **68** | **63 + 5(対象外)** | |

### 3.2 T5 BPMN要素（36個）のカバレッジ

| BPMN要素種別 | T5の要素数 | マッピング済み | 適用ルール |
|-------------|----------|-------------|-----------|
| 開始イベント | 3 | 3 | MR-1 |
| 終了イベント | 5 | 5 | MR-2 |
| タスク | 15 | 15 | MR-3 |
| サブプロセス | 1 | 1 | MR-4 |
| 排他ゲートウェイ | 9 | 9 | MR-5 |
| データオブジェクト | 3 | (対象外) | MR-9 |
| **合計** | **36** | **33 + 3(対象外)** | |

### 3.3 総合カバレッジ

- **T3**: 63/63 = **100%**（データオブジェクト5個除外）
- **T5**: 33/33 = **100%**（データオブジェクト3個除外）
- **重複要素**（T3概要→T5詳細化）: RF-SE〜RF-EE、SP3-GW4→LC-GW1〜4 — いずれもT5側でマッピング済み

---

## 4. Step Bへの申し送り事項

マッピングルールの適用にあたり、Step B（仕様定義）で確定が必要な設計判断を以下に記録する。

| # | 設計判断 | 選択肢 | 推奨 | 理由 |
|---|---------|--------|------|------|
| 1 | LC-GW1〜GW4の変換方式 | A: 状態チェーン / B: ガード合成 | A | INV-LC4（短絡評価）が構造的に保証される |
| 2 | SP-3の並行状態におけるリージョン間通信 | A: 共有イベント / B: `invoke`で分離 | Step Bで検討 | 実装複雑度とテスタビリティのトレードオフ |
| 3 | 復帰先の方式（INV-RF4） | A: 常にSE-1 / B: 履歴状態で損切り前に復帰 | A | INV-RF4の文言に忠実。Bは将来の拡張候補 |
| 4 | SP-3のonDoneによる親への通知方法 | A: 最終状態のID判定 / B: `output`プロパティ | Step Bで検討 | XState v5のoutput APIの安定性次第 |
| 5 | イベント名の命名規則 | A: SCREAMING_SNAKE / B: camelCase | A | XStateの慣例に従う |

---

## 出典

- `phase1-t3-bpmn-elements.md` — T3 BPMN要素定義（68要素）
- `phase1-t5-losscutrecovery-bpmn.md` — T5 BPMN要素定義（36要素）
- `phase1-t7-invariants.md` — 不変条件リスト（74条件）
- `docs/xstate-concepts-mapping.md` — T2 XState基本概念対応表
- T3学習サンプル — 履歴状態・遅延遷移の実装検証結果
