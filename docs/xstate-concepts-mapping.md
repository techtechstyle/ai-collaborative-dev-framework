# T2: XState基本概念 ↔ フレームワーク要素の対応表

> **タスク**: XState v5の7つの基本概念（状態、遷移、イベント、アクション、ガード、ネスト状態、並行状態）を、フレームワークの具体例に対応づけて理解する
> **検証方法**: 7つの概念すべてにフレームワーク内の具体的な対応要素が記載されていること
> **自信度**: 確実（概念レベルの対応づけ）
> **入力**: XState v5公式ドキュメント、`phase1-t3-bpmn-elements.md`、`phase1-t7-invariants.md`

---

## 概要

XState v5はHarelステートチャートのTypeScript実装である。フレームワークのBPMNフロー（Phase 1成果物）をXStateに変換するために、まずXStateの基本概念とフレームワーク要素の対応関係を整理する。

**XState v5の7つの基本概念:**

| # | XState概念 | 英語 | 一言説明 |
|---|-----------|------|---------|
| 1 | 状態 | State | システムが「今どこにいるか」 |
| 2 | 遷移 | Transition | ある状態から別の状態への移動ルール |
| 3 | イベント | Event | 遷移を引き起こすトリガー |
| 4 | アクション | Action | 遷移時に実行される副作用 |
| 5 | ガード | Guard | 遷移を許可するかどうかの条件判定 |
| 6 | ネスト状態 | Nested/Hierarchical State | 状態の中に子状態を持つ階層構造 |
| 7 | 並行状態 | Parallel State | 同時に複数の状態が存在する構造 |

---

## 1. 状態（State）

### XState v5での定義

状態はステートマシンが「今どの段階にいるか」を表す。XStateでは`states`オブジェクトのキーとして定義する。

```typescript
import { createMachine } from 'xstate';

const machine = createMachine({
  id: 'example',
  initial: 'idle',      // 初期状態
  states: {
    idle: {},            // 状態1
    processing: {},      // 状態2
    done: { type: 'final' },  // 最終状態
  },
});
```

### フレームワークとの対応

| XState要素 | フレームワーク要素 | BPMN要素ID | 説明 |
|-----------|-----------------|-----------|------|
| `initial` state | 開始イベント | SE-1 | 「タスクを受け取る」 |
| `final` state | 終了イベント | EE-1, EE-2 | 「タスク完了」「損切り→復帰へ」 |
| 通常 state | タスク | T-1〜T-5 | 各作業ステップ |
| 通常 state | ゲートウェイの出力先 | GW-1〜GW-4の各パス | 判定後の分岐先 |

**具体例: メインフローの状態**

```
SE-1（開始）→ brightLinesCheck → l0l3Check → aiFirstCheck → ...
```

これをXStateで表すと:

```typescript
const mainFlow = createMachine({
  id: 'mainFlow',
  initial: 'brightLinesCheck',
  states: {
    brightLinesCheck: {},       // GW-1: Bright Lines違反チェック
    brightLinesFix: {},         // T-1: 違反是正
    l0l3Check: {},              // SP-1: L0-L3チェック（ネスト状態）
    l0l3Adjust: {},             // T-2: L0-L3調整
    aiFirstCheck: {},           // SP-2: AIファーストチェック（ネスト状態）
    humanExecution: {},         // T-3: 人間主導実行
    aiGeneration: {},           // T-4: AI初期案生成
    humanReview: {},            // T-5: 人間レビュー
    verificationLoop: {},       // SP-3: 検証フィードバックループ（ネスト状態）
    taskComplete: { type: 'final' },  // EE-1: タスク完了
    lossCutExit: { type: 'final' },   // EE-2: 損切り→復帰
  },
});
```

### 対応の考え方

| BPMN要素種別 | XState状態の種類 | 備考 |
|-------------|----------------|------|
| 開始イベント | `initial`で指定される最初の状態 | XStateには「開始イベント」ノードはなく、最初の状態がそれに相当 |
| 終了イベント | `{ type: 'final' }` | 最終状態。到達するとマシンは停止する |
| タスク | 通常の状態 | entryアクションで処理を実行する（§4参照） |
| サブプロセス | ネスト状態（§6参照） | 内部に子状態を持つ複合状態 |
| ゲートウェイ | 状態ではなく、遷移のガード条件（§5参照）で表現 | BPMNではノードだが、XStateでは遷移の一部 |

**重要な違い**: BPMNのゲートウェイ（◇）はXStateでは独立した状態にならない。代わりにガード付き遷移で表現する。これはPhase 2で最も注意が必要な変換ポイントである。

---

## 2. 遷移（Transition）

### XState v5での定義

遷移は「あるイベントが発生したとき、現在の状態から別の状態に移動する」ルール。`on`プロパティで定義する。

```typescript
const machine = createMachine({
  id: 'example',
  initial: 'idle',
  states: {
    idle: {
      on: {
        START: 'processing',    // STARTイベント → processing状態へ
      },
    },
    processing: {
      on: {
        COMPLETE: 'done',       // COMPLETEイベント → done状態へ
        FAIL: 'error',          // FAILイベント → error状態へ
      },
    },
    done: { type: 'final' },
    error: {},
  },
});
```

### フレームワークとの対応

| XState遷移 | フレームワークの遷移 | 対応する不変条件 |
|-----------|-------------------|----------------|
| `brightLinesCheck → l0l3Check` | GW-1 [違反なし] → SP-1 | INV-MF1 |
| `brightLinesCheck → brightLinesFix` | GW-1 [違反あり] → T-1 | INV-MF2 |
| `brightLinesFix → brightLinesCheck` | T-1 → GW-1（再チェック） | INV-MF2 |
| `l0l3Check → aiFirstCheck` | GW-2 [合格] → SP-2 | INV-MF3 |
| `l0l3Check → l0l3Adjust` | GW-2 [不合格] → T-2 | INV-MF3 |
| `verificationLoop → taskComplete` | GW-4 [成功] → EE-1 | INV-MF6 |
| `verificationLoop → lossCutExit` | GW-4 [損切り] → EE-2 | INV-MF6 |

### 遷移の種類

XState v5では以下の遷移が利用可能:

| 遷移の種類 | XState記法 | フレームワークでの用途 |
|-----------|-----------|-------------------|
| イベント遷移 | `on: { EVENT: 'target' }` | 検証結果による分岐（CHECK_PASS/FAIL） |
| 自動遷移 | `always: [{ target: 'next' }]` | 条件を満たしたら即座に遷移（ガードと組合せ） |
| 遅延遷移 | `after: { 1800000: 'timeout' }` | 30分タイマー（INV-LC5） — T3で詳細学習 |

---

## 3. イベント（Event）

### XState v5での定義

イベントは遷移を引き起こすトリガー。外部から`send()`で発火するか、マシン内部で`raise()`する。

```typescript
// イベントの型定義
type Events =
  | { type: 'CHECK_PASS' }
  | { type: 'CHECK_FAIL'; errorCount: number }
  | { type: 'TIMEOUT' };

const machine = createMachine({
  types: {} as {
    events: Events;
  },
  // ...
});
```

### フレームワークとの対応

| イベント名（候補） | フレームワークの事象 | 発生元 |
|------------------|-------------------|--------|
| `TASK_RECEIVED` | タスクを受け取る | 外部（人間の操作） |
| `BL_VIOLATION` | Bright Lines違反検出 | GW-1の判定結果 |
| `BL_CLEARED` | Bright Lines違反なし | GW-1の判定結果 |
| `LEVEL_PASS` | 各レベル（L0〜L3）通過 | SP1-GW1〜GW4の判定結果 |
| `LEVEL_FAIL` | 各レベル不合格 | SP1-GW1〜GW4の判定結果 |
| `AI_SUITABLE` | AI主導と判定 | SP2-GW2の判定結果 |
| `HUMAN_SUITABLE` | 人間主導と判定 | SP2-GW2の判定結果 |
| `CHECK_PASS` | typecheck/lint/test成功 | SP3-T1〜T3の実行結果 |
| `CHECK_FAIL` | typecheck/lint/test失敗 | SP3-T1〜T3の実行結果 |
| `LOSS_CUT` | 損切り確定 | LC-GW1〜GW4のいずれかYes |
| `CONTINUE_FIX` | 修正継続 | LC-GW4のNo |
| `TIMEOUT_30MIN` | 30分経過 | SP3-IE1（中間タイマーイベント） |
| `ERROR_3RD` | 同一エラー3回目 | SP3-IE2（中間エラーイベント） |
| `RECOVERY_COMPLETE` | 復帰フロー完了 | RF-EE |

### イベントの分類

| 分類 | 説明 | 例 |
|------|------|-----|
| 外部イベント | 人間の操作で発生 | `TASK_RECEIVED`, `CHECK_PASS` |
| 内部イベント | マシン内部で自動発生 | `LEVEL_PASS`（ガード評価結果） |
| タイマーイベント | 時間経過で発生 | `TIMEOUT_30MIN`（XStateの`after`で実装） |

**設計判断（Step Bで確定）**: 上記イベント名は仮のもの。Step B（T5〜T8）の仕様定義で正式に決定する。

---

## 4. アクション（Action）

### XState v5での定義

アクションは遷移時に実行される副作用。3種類のタイミングがある。

```typescript
const machine = createMachine({
  id: 'example',
  initial: 'idle',
  states: {
    idle: {
      entry: [{ type: 'logEntry' }],  // この状態に入ったとき
      exit: [{ type: 'logExit' }],    // この状態を出るとき
      on: {
        START: {
          target: 'processing',
          actions: [{ type: 'recordStart' }],  // 遷移時
        },
      },
    },
    processing: {},
  },
});
```

### フレームワークとの対応

| アクション（候補） | タイミング | フレームワークの処理 | 対応要素 |
|------------------|----------|-------------------|---------|
| `recordBrightLinesViolation` | entry | Bright Lines違反を記録 | T-1 entry |
| `evaluateL0` | entry | L0持続可能性チェックを実行 | SP1-T1 entry |
| `evaluateL1` | entry | L1心理的安全性チェックを実行 | SP1-T2 entry |
| `evaluateL2` | entry | L2急がば回れチェックを実行 | SP1-T3 entry |
| `evaluateL3` | entry | L3シンプルさチェックを実行 | SP1-T4 entry |
| `selectPromptTechnique` | entry | プロンプト技法を選択 | SP2-T3 entry |
| `runTypecheck` | entry | typecheckを実行 | SP3-T1 entry |
| `runLint` | entry | lintを実行 | SP3-T2 entry |
| `runTest` | entry | testを実行 | SP3-T3 entry |
| `incrementErrorCount` | 遷移時 | エラーカウンタを+1 | CHECK_FAIL遷移 |
| `recordErrorState` | entry | エラー状態を記録 | LC-T1 entry |
| `updateClaudeMd` | entry | CLAUDE.mdに失敗パターンを記録 | RF-T9 entry（INV-RF2） |
| `recordAvoidanceStrategy` | entry | 次回の回避策を明記 | RF-T10 entry（INV-RF3） |

### アクションの3つのタイミング

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  exit アクション   →   遷移アクション   →   entry アクション  │
  │  (元の状態を出る)     (遷移中に実行)      (新しい状態に入る)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

**フレームワークでの主な用途**:
- `entry`: BPMNのタスク（□）に対応。状態に入ったら処理を実行する
- 遷移時: カウンタの更新、ログ記録など
- `exit`: 状態を離れる前のクリーンアップ（フレームワークでは使用頻度低い）

---

## 5. ガード（Guard）

### XState v5での定義

ガードは遷移を許可するかどうかの条件判定関数。`true`を返した場合のみ遷移が実行される。

```typescript
const machine = createMachine({
  id: 'example',
  initial: 'checking',
  states: {
    checking: {
      on: {
        CHECK_RESULT: [
          {
            guard: 'isPass',         // ガード: trueなら遷移
            target: 'passed',
          },
          {
            target: 'failed',        // ガードなし: フォールバック
          },
        ],
      },
    },
    passed: {},
    failed: {},
  },
}, {
  guards: {
    isPass: ({ context, event }) => event.score >= 70,
  },
});
```

### フレームワークとの対応

ガードはBPMNのゲートウェイ（◇）の条件判定に対応する。これがBPMN→XState変換の核心部分である。

| ガード（候補） | 対応するBPMN要素 | 判定内容 | 対応する決定表 |
|-------------|----------------|---------|--------------|
| `hasBrightLinesViolation` | GW-1 | BL1-BL4のいずれかに違反 | DT-0 |
| `isL0Pass` | SP1-GW1 | L0持続可能性の全原則を通過 | DT-2 |
| `isL1Pass` | SP1-GW2 | L1心理的安全性の全原則を通過 | DT-3 |
| `isL2Pass` | SP1-GW3 | L2急がば回れの全原則を通過 | DT-4 |
| `isL3Pass` | SP1-GW4 | L3シンプルさの全原則を通過 | DT-5 |
| `isAiSuitable` | SP2-GW1 | AIの得意分野に該当 | DT-6 |
| `isAiLead` | SP2-GW2 | AI主導で実行すべき | DT-6 |
| `isTypecheckPass` | SP3-GW1 | typecheckが成功 | — |
| `isLintPass` | SP3-GW2 | lintが成功 | — |
| `isTestPass` | SP3-GW3 | testが成功 | — |
| `isErrorCount3OrMore` | LC-GW1 | 同一エラー3回以上 | INV-LC2 |
| `isOver30Min` | LC-GW2 | 30分超過 | INV-LC2 |
| `isGrowingComplexity` | LC-GW3 | コードが複雑化している | INV-LC2 |
| `isRecurringError` | LC-GW4 | 同じエラーが再発 | INV-LC2 |

### ガードの配列による条件分岐パターン

XStateでは同一イベントに対してガードの配列で分岐を表現する。BPMNのXORゲートウェイに直接対応する。

```typescript
// BPMNの SP1-GW1（L0通過判定）をXStateで表現
on: {
  EVALUATION_COMPLETE: [
    {
      guard: 'isL0Pass',
      target: 'l1Check',        // Yes → 次レベルへ
    },
    {
      target: 'l0l3Failed',     // No → 不合格
    },
  ],
}
```

**重要**: ガードの配列は上から順に評価され、最初にtrueになったものが採用される。これはDT-0〜DT-1のヒットポリシーF（最初一致）と同じ評価ロジックである。

### ガードの合成

XState v5では `and`, `or`, `not` でガードを合成できる。損切り4条件のOR評価（INV-LC2）に有用。

```typescript
// 損切り4条件のOR評価
guard: {
  type: 'or',
  guards: [
    'isErrorCount3OrMore',
    'isOver30Min',
    'isGrowingComplexity',
    'isRecurringError',
  ],
}
```

ただし、INV-LC4（短絡評価：最初のYesで即座に損切り確定）を表現するには、ガード合成ではなく逐次状態遷移のほうが適切かもしれない。この設計判断はStep B（T7仕様定義）で確定する。

---

## 6. ネスト状態（Nested / Hierarchical State）

### XState v5での定義

ネスト状態は状態の中に子状態を持つ階層構造。XStateではstatesの中にさらにstatesを定義する。

```typescript
const machine = createMachine({
  id: 'example',
  initial: 'parent',
  states: {
    parent: {
      initial: 'child1',          // 親状態に入ると最初にchild1
      states: {
        child1: {
          on: { NEXT: 'child2' },
        },
        child2: {
          on: { NEXT: 'child3' },
        },
        child3: { type: 'final' }, // 子の最終状態
      },
      onDone: 'nextParent',        // 子が全て完了 → 親を抜ける
    },
    nextParent: {},
  },
});
```

### フレームワークとの対応

フレームワークのBPMNサブプロセス（□⊞）がネスト状態に直接対応する。

| ネスト状態 | BPMNサブプロセス | 子状態の内容 | 階層の深さ |
|-----------|----------------|------------|----------|
| `l0l3Check` | SP-1 | L0→L1→L2→L3の逐次チェック（4タスク＋4ゲートウェイ） | 2層 |
| `aiFirstCheck` | SP-2 | タスク特性分析→分業判断→プロンプト選択 | 2層 |
| `verificationLoop` | SP-3 | typecheck→lint→test＋協働原則監視（並行状態） | 2〜3層 |
| `taskDecomposition` | TD | 5ステップの逐次実行 | 2層 |
| `lossCutJudgment` | LC | 4条件の逐次判定 | 2層 |
| `recoveryFlow` | RF | 分析→アプローチ選択→記録→復帰 | 2〜3層 |

### 具体例: SP-1（L0-L3チェック）のネスト構造

```typescript
const sp1Machine = createMachine({
  id: 'sp1',
  initial: 'l0Check',
  states: {
    l0Check: {
      // SP1-T1: L0持続可能性チェック
      on: {
        EVALUATION_COMPLETE: [
          { guard: 'isL0Pass', target: 'l1Check' },  // SP1-GW1 Yes
          { target: 'failed' },                        // SP1-GW1 No
        ],
      },
    },
    l1Check: {
      // SP1-T2: L1心理的安全性チェック
      on: {
        EVALUATION_COMPLETE: [
          { guard: 'isL1Pass', target: 'l2Check' },  // SP1-GW2 Yes
          { target: 'failed' },                        // SP1-GW2 No
        ],
      },
    },
    l2Check: {
      // SP1-T3: L2急がば回れチェック
      on: {
        EVALUATION_COMPLETE: [
          { guard: 'isL2Pass', target: 'l3Check' },  // SP1-GW3 Yes
          { target: 'failed' },                        // SP1-GW3 No
        ],
      },
    },
    l3Check: {
      // SP1-T4: L3シンプルさチェック
      on: {
        EVALUATION_COMPLETE: [
          { guard: 'isL3Pass', target: 'passed' },   // SP1-GW4 Yes
          { target: 'failed' },                        // SP1-GW4 No
        ],
      },
    },
    passed: { type: 'final' },   // SP1-EE-OK
    failed: { type: 'final' },   // SP1-EE-NG
  },
});
```

この構造により、以下の不変条件が構造的に保証される:
- **INV-SP1-1**: `initial: 'l0Check'` → L0が常に最初
- **INV-SP1-2**: 各状態のon遷移がガード条件でのみ次に進む → 逐次実行
- **INV-SP1-3**: ガード失敗時は`failed`に直行 → 他レベルはスキップ
- **INV-SP1-4**: 終了は`passed`か`failed`の2つのみ

### ネスト深度の設計指針

タスク定義のリスク分析より、3〜4層程度に抑える設計が推奨される。

```
メインフロー（層1）
  └── SP-1: L0-L3チェック（層2）
         └── 各レベルチェック（層3）— これ以上深くしない
```

---

## 7. 並行状態（Parallel State）

### XState v5での定義

並行状態は複数の子状態が同時にアクティブになる構造。`type: 'parallel'`で定義する。

```typescript
const machine = createMachine({
  id: 'example',
  type: 'parallel',              // 並行状態の宣言
  states: {
    regionA: {
      initial: 'idle',
      states: {
        idle: { on: { START_A: 'running' } },
        running: { on: { DONE_A: 'complete' } },
        complete: { type: 'final' },
      },
    },
    regionB: {
      initial: 'idle',
      states: {
        idle: { on: { START_B: 'running' } },
        running: { on: { DONE_B: 'complete' } },
        complete: { type: 'final' },
      },
    },
  },
});
```

### フレームワークとの対応

フレームワークで並行状態が必要なのは **SP-3（検証フィードバックループ）** である。

SP-3では、検証の逐次実行（typecheck→lint→test）と原則監視（協働原則C1-C4＋AI行動原則A1-A4）が並行して動作する（INV-SP3-3）。

```
SP-3（並行状態）
├── region: 検証実行
│     typecheck → lint → test → 完了/失敗
│
└── region: 原則監視
      協働原則チェック（DT-8）
      AI行動原則チェック（DT-9）
      → 常時監視、違反検出時にイベント発火
```

### 具体例: SP-3の並行構造

```typescript
const sp3Machine = createMachine({
  id: 'sp3',
  type: 'parallel',
  states: {
    // リージョン1: 検証の逐次実行
    verification: {
      initial: 'typecheck',
      states: {
        typecheck: {
          on: {
            CHECK_PASS: 'lint',
            CHECK_FAIL: 'verificationFailed',
          },
        },
        lint: {
          on: {
            CHECK_PASS: 'test',
            CHECK_FAIL: 'verificationFailed',
          },
        },
        test: {
          on: {
            CHECK_PASS: 'verificationPassed',
            CHECK_FAIL: 'verificationFailed',
          },
        },
        verificationPassed: { type: 'final' },
        verificationFailed: { type: 'final' },
      },
    },
    // リージョン2: 原則監視（常時アクティブ）
    principleMonitor: {
      initial: 'monitoring',
      states: {
        monitoring: {
          on: {
            COLLABORATION_VIOLATION: 'violated',  // DT-8違反
            AI_PRINCIPLE_VIOLATION: 'violated',    // DT-9違反
          },
        },
        violated: { type: 'final' },
      },
    },
  },
});
```

### 並行状態の注意点

並行状態はリージョン間の同期が複雑になりうる。特に:

- 検証が失敗した場合、原則監視も停止させる必要がある（リージョン間の連動）
- 原則違反が検出された場合、検証を中断させる必要がある（リージョン間のイベント通信）

これらの実装方法はT3（学習タスク）で実際にコードを書いて検証する。

---

## まとめ: BPMN → XState 変換の対応関係

| BPMN要素 | XState要素 | 変換規則 |
|----------|-----------|---------|
| 開始イベント（○） | `initial` 状態 | 最初の状態として設定 |
| 終了イベント（◎） | `{ type: 'final' }` | 最終状態として設定 |
| タスク（□） | 状態 + `entry`アクション | 状態に入ったら処理を実行 |
| サブプロセス（□⊞） | ネスト状態 | 子状態を持つ階層構造 |
| 排他ゲートウェイ（◇ XOR） | ガード付き遷移の配列 | 条件ごとに分岐先を指定 |
| 並列ゲートウェイ（◇+ AND） | `type: 'parallel'` | 複数リージョンを同時実行 |
| 中間イベント（タイマー）（○⏱） | `after` 遅延遷移 | T3で詳細学習 |
| 中間イベント（エラー）（○!） | ガード条件 + `context` | カウンタベースの判定 |
| データオブジェクト（📄） | `context`（コンテキスト） | マシンの状態データとして保持 |
| フロー接続（→） | 遷移（`on` / `always`） | イベントまたは自動遷移 |

### 追記: context（コンテキスト）について

上記の表でデータオブジェクトの対応先として記載した`context`は、7つの基本概念には含まれないが、XStateの重要な機能である。

```typescript
const machine = createMachine({
  context: {
    errorCount: 0,           // D-4に対応: エラーカウンタ
    startTime: 0,            // 30分タイマーの基準時刻
    claudeMdUpdated: false,  // D-1に対応: CLAUDE.md更新フラグ
  },
  // ...
});
```

| contextフィールド（候補） | 対応するデータオブジェクト | 用途 |
|------------------------|----------------------|------|
| `errorCount` | D-4（検証結果） | 同一エラー3回の判定（INV-LC2） |
| `startTime` | — | 30分経過の判定（INV-LC2） |
| `claudeMdUpdated` | D-1（CLAUDE.md） | RF-T9の実行確認（INV-RF2） |
| `taskList` | D-2（タスク一覧） | 分解後のタスク管理 |
| `failurePattern` | D-5（失敗パターン記録） | 復帰フローでの記録 |

---

## 次のステップ

本ドキュメントで7つの基本概念の理解と対応づけが完了した。続くタスクは以下の通り:

- **T3**: 履歴状態（history state）と遅延遷移（delayed transition）の理解 — 復帰フローと30分ルールの実装に必要
- **T4**: BPMN→ステートチャートの詳細マッピングルール — 68個のBPMN要素すべてに対する変換規則を策定
