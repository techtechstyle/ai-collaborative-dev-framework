# T6: メインフロー＋SP-2のステートチャート仕様

> **タスク**: メインフロー（Bright Lines → L0-L3チェック → AIファースト判断 → 実行 → 検証）とSP-2（AIファーストチェック＋分業判断）のステートチャートを定義する
> **検証方法**: INV-MF1〜MF6、INV-SP2-1〜SP2-4の10個の不変条件が設計上すべて保証されていること
> **自信度**: おそらく
> **入力**: `phase1-t3-bpmn-elements.md`（§1, §3）、`phase1-t4-mermaid-flowcharts.md`（§1）、`phase1-t7-invariants.md`（INV-MF, INV-SP2）、`docs/bpmn-to-statechart-mapping.md`（MR-1〜MR-9）、`docs/spec-l0l4-hierarchy.md`（T5成果物）

---

## 1. スコープ

| 対象 | 詳細度 | 理由 |
|------|--------|------|
| メインフロー（SE-1〜EE-1/EE-2） | **内部詳細定義** | INV-MF1〜MF6の保証に全要素の遷移定義が必要 |
| SP-2（AIファーストチェック＋分業判断） | **内部詳細定義** | INV-SP2-1〜SP2-4の保証にSP-2内部構造が必要 |
| SP-1（L0-L3チェック） | **T5成果物を組込** | T5で定義済み。メインフローのネスト状態として接続 |
| SP-3（検証フィードバックループ） | **外部インターフェースのみ** | INV-MF5の保証に「T-3/T-5からSP-3への合流」が必要。SP-3内部はT7 |
| TD（タスク分解） | **外部インターフェースのみ** | T-2から呼び出されるが、TD内部はT6スコープ外 |

### T5からの申し送り対応

| # | 申し送り | T6での対応 |
|---|---------|-----------|
| 1 | Bright Linesチェック（DT-0）のガード実装 | §3.5 `brightLinesCheck`状態で定義 → INV-H5の構造的保証 |
| 2 | SP-2の内部定義 | §4で詳細定義 |
| 3 | `isSP1Passed`ガードの実装 | §3.5 `l0l3Check`の`onDone`で`output`を参照 |
| 4 | INV-H4（競合解決）の完全な実装 | §6.3 設計上の保証として記載。DT-10の完全実装はT8統合仕様 |
| 5 | onDone通知方法は`output`を踏襲 | SP-2、SP-3でも`output`プロパティを採用 |

---

## 2. メインフローの全体構造

### 2.1 状態階層図

```
mainFlowMachine（最上位マシン）
│
├── brightLinesCheck      ← GW-1（DT-0）: INV-MF1, INV-H5
├── brightLinesFix        ← T-1: INV-MF2
│
├── l0l3Check（SP-1）     ← T5で定義済み（compound state）
├── l0l3Adjust            ← T-2: INV-MF3
│
├── aiFirstCheck（SP-2）  ← 本T6で詳細定義（compound state）
│   ├── taskAnalysis
│   ├── divisionDecision
│   ├── promptSelection
│   ├── aiLead（final）
│   └── humanLead（final）
│
├── humanExecution         ← T-3: 人間主導パス
├── aiGeneration           ← T-4: AI主導パス
├── humanReview            ← T-5: INV-MF4
│
├── verificationLoop（SP-3）← T7で詳細定義（外部IFのみ）
│
├── taskComplete（final）  ← EE-1: INV-MF6
└── lossCutExit（final）   ← EE-2: INV-MF6
```

### 2.2 状態遷移の全体フロー

```
SE-1（initial: brightLinesCheck）
    │
    ▼
brightLinesCheck ──[違反あり]──→ brightLinesFix ──→ brightLinesCheck（ループ）
    │                                                    ← INV-MF2
    │[違反なし]
    ▼                                                    ← INV-MF1
l0l3Check（SP-1）
    │
    ├── onDone [不合格] → l0l3Adjust → l0l3Check（ループ）← INV-MF3
    │
    └── onDone [合格] → aiFirstCheck（SP-2）
                │
                ├── onDone [人間主導] → humanExecution → verificationLoop
                │                                        ← INV-MF5
                └── onDone [AI主導] → aiGeneration → humanReview → verificationLoop
                                                      ← INV-MF4, INV-MF5
                                        │
                                        ▼
                                   verificationLoop（SP-3）
                                        │
                                        ├── onDone [成功] → taskComplete     ← INV-MF6
                                        └── onDone [損切り] → lossCutExit   ← INV-MF6
```

---

## 3. メインフローの詳細仕様

### 3.1 マシン定義

**マシンID**: `mainFlowMachine`
**種類**: 最上位ステートマシン

### 3.2 コンテキスト定義

```typescript
type MainFlowContext = {
  /** Bright Lines違反の内容（GW-1で検出） */
  brightLinesViolation: BrightLinesViolation | null;
  /** SP-1の評価結果（T5のoutputから受け取り） */
  l0l3Result: L0L3CheckOutput | null;
  /** SP-2の分業結果 */
  divisionResult: DivisionResult | null;
  /** AI出力（D-3） */
  aiOutput: unknown | null;
  /** SP-3の検証結果 */
  verificationResult: VerificationResult | null;
};

type BrightLinesViolation = {
  violatedRule: 'BL1' | 'BL2' | 'BL3' | 'BL4';
  description: string;
};

type L0L3CheckOutput = {
  result: { l0: LevelResult | null; l1: LevelResult | null; l2: LevelResult | null; l3: LevelResult | null };
  allPassed: boolean;
};

type DivisionResult = {
  lead: 'ai' | 'human';
  promptTechnique?: string;  // AI主導の場合のみ
};

type VerificationResult = {
  passed: boolean;
  lossCut: boolean;
};
```

### 3.3 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `brightLinesCheck` | 通常状態（initial） | `evaluateBrightLines` | GW-1 |
| `brightLinesFix` | 通常状態 | `fixBrightLinesViolation` | T-1 |
| `l0l3Check` | compound state | （T5で定義） | SP-1 |
| `l0l3Adjust` | 通常状態 | `adjustForL0L3` | T-2 |
| `aiFirstCheck` | compound state | （§4で定義） | SP-2 |
| `humanExecution` | 通常状態 | `executeHumanLead` | T-3 |
| `aiGeneration` | 通常状態 | `generateWithAI` | T-4 |
| `humanReview` | 通常状態 | `reviewAIOutput` | T-5 |
| `verificationLoop` | compound state | （T7で定義） | SP-3 |
| `taskComplete` | 最終状態 | — | EE-1 |
| `lossCutExit` | 最終状態 | — | EE-2 |

### 3.4 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `BRIGHT_LINES_EVALUATED` | Bright Linesチェック完了 | `evaluateBrightLines`アクション |
| `BRIGHT_LINES_FIXED` | Bright Lines是正完了 | `fixBrightLinesViolation`アクション |
| `L0L3_ADJUSTMENT_COMPLETE` | L0-L3調整完了 | `adjustForL0L3`アクション |
| `HUMAN_EXECUTION_COMPLETE` | 人間主導実行完了 | `executeHumanLead`アクション |
| `AI_GENERATION_COMPLETE` | AI初期案生成完了 | `generateWithAI`アクション |
| `HUMAN_REVIEW_COMPLETE` | 人間レビュー完了 | `reviewAIOutput`アクション |

**注記**: SP-1, SP-2, SP-3の完了はイベントではなく`onDone`で処理される（MR-4: サブプロセス → ネスト状態のonDone）。

### 3.5 ガード条件定義

| ガード名 | 条件 | 対応GW | 保証する不変条件 |
|---------|------|--------|----------------|
| `hasBrightLinesViolation` | `context.brightLinesViolation !== null` | GW-1 | INV-MF1, INV-H5 |
| `isSP1Passed` | `event.output.allPassed === true` | GW-2 | INV-MF3, INV-H3 |
| `isAiLead` | `event.output.lead === 'ai'` | GW-3 | — |
| `isVerificationPassed` | `event.output.passed === true` | GW-4 | INV-MF6 |

### 3.6 遷移表

| 現在の状態 | イベント/条件 | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|-------------|--------|-----------|--------|------------|
| `brightLinesCheck` | `BRIGHT_LINES_EVALUATED` | `hasBrightLinesViolation` | `assignViolation` | `brightLinesFix` | INV-MF2 |
| `brightLinesCheck` | `BRIGHT_LINES_EVALUATED` | （フォールバック） | `clearViolation` | `l0l3Check` | INV-MF1, INV-H5 |
| `brightLinesFix` | `BRIGHT_LINES_FIXED` | — | — | `brightLinesCheck` | INV-MF2 |
| `l0l3Check` | onDone | `isSP1Passed` | `assignL0L3Result` | `aiFirstCheck` | INV-MF3, INV-H3 |
| `l0l3Check` | onDone | （フォールバック） | `assignL0L3Result` | `l0l3Adjust` | INV-MF3 |
| `l0l3Adjust` | `L0L3_ADJUSTMENT_COMPLETE` | — | — | `l0l3Check` | INV-MF3 |
| `aiFirstCheck` | onDone | `isAiLead` | `assignDivisionResult` | `aiGeneration` | — |
| `aiFirstCheck` | onDone | （フォールバック） | `assignDivisionResult` | `humanExecution` | — |
| `humanExecution` | `HUMAN_EXECUTION_COMPLETE` | — | — | `verificationLoop` | INV-MF5 |
| `aiGeneration` | `AI_GENERATION_COMPLETE` | — | `assignAiOutput` | `humanReview` | INV-MF4 |
| `humanReview` | `HUMAN_REVIEW_COMPLETE` | — | — | `verificationLoop` | INV-MF4, INV-MF5 |
| `verificationLoop` | onDone | `isVerificationPassed` | — | `taskComplete` | INV-MF6 |
| `verificationLoop` | onDone | （フォールバック） | — | `lossCutExit` | INV-MF6 |

### 3.7 XState v5 擬似コード（メインフロー）

```typescript
import { setup, assign } from 'xstate';

const mainFlowMachine = setup({
  types: {
    context: {} as MainFlowContext,
    events: {} as
      | { type: 'BRIGHT_LINES_EVALUATED'; violation: BrightLinesViolation | null }
      | { type: 'BRIGHT_LINES_FIXED' }
      | { type: 'L0L3_ADJUSTMENT_COMPLETE' }
      | { type: 'HUMAN_EXECUTION_COMPLETE' }
      | { type: 'AI_GENERATION_COMPLETE'; output: unknown }
      | { type: 'HUMAN_REVIEW_COMPLETE' },
  },
  guards: {
    hasBrightLinesViolation: ({ event }) =>
      event.type === 'BRIGHT_LINES_EVALUATED' && event.violation !== null,
    isSP1Passed: (_, params) => params.output.allPassed === true,
    isAiLead: (_, params) => params.output.lead === 'ai',
    isVerificationPassed: (_, params) => params.output.passed === true,
  },
  actions: {
    evaluateBrightLines: () => { /* DT-0 評価ロジック */ },
    fixBrightLinesViolation: () => { /* BL1-BL4 是正処理 */ },
    adjustForL0L3: () => { /* タスク分解（TD）等の調整処理 */ },
    executeHumanLead: () => { /* 人間主導実行 */ },
    generateWithAI: () => { /* AI初期案生成 */ },
    reviewAIOutput: () => { /* 人間レビュー */ },
  },
}).createMachine({
  id: 'mainFlow',
  initial: 'brightLinesCheck',
  context: {
    brightLinesViolation: null,
    l0l3Result: null,
    divisionResult: null,
    aiOutput: null,
    verificationResult: null,
  },
  states: {
    // --- Bright Linesチェック（DT-0） ---
    brightLinesCheck: {
      entry: [{ type: 'evaluateBrightLines' }],
      on: {
        BRIGHT_LINES_EVALUATED: [
          {
            guard: 'hasBrightLinesViolation',
            actions: assign({
              brightLinesViolation: ({ event }) => event.violation,
            }),
            target: 'brightLinesFix',
          },
          {
            actions: assign({ brightLinesViolation: () => null }),
            target: 'l0l3Check',
          },
        ],
      },
    },

    // --- Bright Lines是正（T-1） ---
    brightLinesFix: {
      entry: [{ type: 'fixBrightLinesViolation' }],
      on: {
        BRIGHT_LINES_FIXED: 'brightLinesCheck',  // 再チェックループ
      },
    },

    // --- L0-L3チェック（SP-1: T5で定義済み） ---
    l0l3Check: {
      // T5の l0l3CheckMachine をネスト状態として組み込む
      initial: 'l0Check',
      states: {
        // ... T5 §3.8 の状態定義を展開 ...
        l0Check: { /* T5で定義 */ },
        l1Check: { /* T5で定義 */ },
        l2Check: { /* T5で定義 */ },
        l3Check: { /* T5で定義 */ },
        passed: { type: 'final' },
        failed: { type: 'final' },
      },
      output: ({ context }) => ({
        // T5 §3.8 の output 定義
        allPassed: true, // 実際はコンテキストから算出
      }),
      onDone: [
        {
          guard: 'isSP1Passed',
          target: 'aiFirstCheck',
        },
        {
          target: 'l0l3Adjust',
        },
      ],
    },

    // --- L0-L3調整（T-2） ---
    l0l3Adjust: {
      entry: [{ type: 'adjustForL0L3' }],
      on: {
        L0L3_ADJUSTMENT_COMPLETE: 'l0l3Check',  // 再チェックループ
      },
    },

    // --- AIファーストチェック＋分業判断（SP-2: §4で定義） ---
    aiFirstCheck: {
      // §4 の詳細定義を展開
      initial: 'taskAnalysis',
      states: {
        taskAnalysis: { /* §4で定義 */ },
        divisionDecision: { /* §4で定義 */ },
        promptSelection: { /* §4で定義 */ },
        aiLead: { type: 'final' },
        humanLead: { type: 'final' },
      },
      output: ({ context }) => ({
        lead: 'ai', // 実際はコンテキストから算出
      }),
      onDone: [
        {
          guard: 'isAiLead',
          target: 'aiGeneration',
        },
        {
          target: 'humanExecution',
        },
      ],
    },

    // --- 人間主導実行（T-3） ---
    humanExecution: {
      entry: [{ type: 'executeHumanLead' }],
      on: {
        HUMAN_EXECUTION_COMPLETE: 'verificationLoop',
      },
    },

    // --- AI初期案生成（T-4） ---
    aiGeneration: {
      entry: [{ type: 'generateWithAI' }],
      on: {
        AI_GENERATION_COMPLETE: {
          actions: assign({
            aiOutput: ({ event }) => event.output,
          }),
          target: 'humanReview',
        },
      },
    },

    // --- 人間レビュー（T-5） ---
    humanReview: {
      entry: [{ type: 'reviewAIOutput' }],
      on: {
        HUMAN_REVIEW_COMPLETE: 'verificationLoop',
      },
    },

    // --- 検証フィードバックループ（SP-3: T7で定義） ---
    verificationLoop: {
      // T7で内部定義。ここでは外部IFのみ
      initial: 'typecheck', // T7で確定
      states: {
        // ... T7で定義 ...
      },
      output: ({ context }) => ({
        passed: true,
        lossCut: false,
      }),
      onDone: [
        {
          guard: 'isVerificationPassed',
          target: 'taskComplete',
        },
        {
          target: 'lossCutExit',
        },
      ],
    },

    // --- 最終状態 ---
    taskComplete: { type: 'final' },   // EE-1
    lossCutExit: { type: 'final' },     // EE-2
  },
});
```

---

## 4. SP-2（AIファーストチェック＋分業判断）の詳細仕様

### 4.1 マシン定義

**マシンID**: `aiFirstCheckMachine`（メインフローの`aiFirstCheck`としてネスト）
**種類**: compound state

### 4.2 コンテキスト定義

```typescript
type AIFirstCheckContext = {
  /** タスク特性の分析結果（SP2-T1） */
  taskCharacteristics: TaskCharacteristics | null;
  /** 分業判断結果（SP2-T2 / DT-6） */
  divisionDecision: DivisionDecision | null;
  /** プロンプト技法（SP2-T3 / DT-7）— AI主導の場合のみ */
  promptTechnique: PromptTechnique | null;
};

type TaskCharacteristics = {
  /** F1: AIが得意な作業か？ */
  isAiSuitable: boolean | null;  // null = 不明（DT-6 ルール6）
  /** F2: 一貫性 vs 創造性 */
  consistencyVsCreativity: 'consistency' | 'creativity' | null;
  /** F3: 網羅性チェックが必要か？ */
  needsCompletenessCheck: boolean;
};

type DivisionDecision = {
  lead: 'ai' | 'human' | 'undecided';
  /** DT-6で合致したルール# */
  matchedRule: number;
};

type PromptTechnique =
  | 'zero-shot'
  | 'chain-of-thought'
  | 'tree-of-thoughts'
  | 'react'
  | 'self-consistency';
```

### 4.3 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `taskAnalysis` | 通常状態（initial） | `analyzeTaskCharacteristics` | SP2-T1 |
| `divisionDecision` | 通常状態 | `decideDivision` | SP2-T2 |
| `promptSelection` | 通常状態 | `selectPromptTechnique` | SP2-T3 |
| `aiLead` | 最終状態 | — | SP2-EE-AI |
| `humanLead` | 最終状態 | — | SP2-EE-HM |

### 4.4 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `TASK_ANALYSIS_COMPLETE` | タスク特性分析完了 | `analyzeTaskCharacteristics` |
| `DIVISION_DECIDED` | 分業判断完了 | `decideDivision` |
| `PROMPT_SELECTED` | プロンプト技法選択完了 | `selectPromptTechnique` |

### 4.5 ガード条件定義

| ガード名 | 条件 | 対応GW | 対応DT |
|---------|------|--------|--------|
| `isAiSuitable` | `context.taskCharacteristics.isAiSuitable !== false` | SP2-GW1 | DT-6入力列F1 |
| `isAiLeadDecision` | `context.divisionDecision.lead === 'ai'` | SP2-GW2 | DT-6出力列 |

**SP2-GW1のガード設計**: `isAiSuitable`はfalseの場合のみ人間主導に直行。`true`と`null`（不明）の場合はDT-6の詳細判断（SP2-T2）に進む。これはT3 §3.2のフロー「SP2-GW1 --[No: AI不得意]→ SP2-EE-HM / SP2-GW1 --[Yes/不明]→ SP2-T2」に対応。

### 4.6 遷移表

| 現在の状態 | イベント | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|---------|--------|-----------|--------|------------|
| `taskAnalysis` | `TASK_ANALYSIS_COMPLETE` | `isAiSuitable` | `assignTaskCharacteristics` | `divisionDecision` | INV-SP2-1 |
| `taskAnalysis` | `TASK_ANALYSIS_COMPLETE` | （フォールバック: AI不得意） | `assignTaskCharacteristics` | `humanLead` | INV-SP2-1 |
| `divisionDecision` | `DIVISION_DECIDED` | `isAiLeadDecision` | `assignDivisionDecision` | `promptSelection` | INV-SP2-2 |
| `divisionDecision` | `DIVISION_DECIDED` | （フォールバック: 人間主導） | `assignDivisionDecision` | `humanLead` | — |
| `promptSelection` | `PROMPT_SELECTED` | — | `assignPromptTechnique` | `aiLead` | INV-SP2-2 |

### 4.7 DT-6（分業判断）のガードマッピング

DT-6のヒットポリシーU（一意）は、排他的条件によって実現される。

| DT-6ルール# | タスク特性 | F1判断 | 結果 | SP-2での対応 |
|-------------|----------|--------|------|------------|
| 1 | 初期案・たたき台の作成 | はい | AI主導 | `divisionDecision` → `promptSelection` → `aiLead` |
| 2 | スタイル統一・規約遵守 | はい | AI主導 | 同上 |
| 3 | 漏れ・抜けの検出 | はい | AI主導 | 同上 |
| 4 | 設計判断・アーキテクチャ | いいえ | 人間主導 | `taskAnalysis` → `humanLead` |
| 5 | ドメイン固有の判断 | いいえ | 人間主導 | 同上 |
| 6 | 上記以外 | 不明 | 要判断 | `divisionDecision`で個別判断 |

**INV-SP2-4の保証**: DT-6のルール条件が排他的であること。ガード`isAiLeadDecision`は`decideDivision`アクション内でDT-6を評価した結果（`lead === 'ai'`）を判定する。DT-6の排他性は決定表自体の設計で保証される（ヒットポリシーU）。

### 4.8 DT-7（プロンプト技法選択）のマッピング

DT-7はDT-6でAI主導と判断された場合にのみ参照される（INV-DT6）。

```typescript
// DT-7の評価ロジック（SP2-T3のentryアクション内）
function selectPromptTechnique(taskInfo: TaskCharacteristics): PromptTechnique {
  if (taskInfo.complexity === 'simple') return 'zero-shot';        // ルール1
  if (taskInfo.complexity === 'moderate') return 'chain-of-thought'; // ルール2
  if (taskInfo.needsComparison) return 'tree-of-thoughts';          // ルール3
  if (taskInfo.needsExternalInfo) return 'react';                   // ルール4
  return 'self-consistency';                                         // ルール5（最高確実性）
}
```

### 4.9 XState v5 擬似コード（SP-2）

```typescript
const aiFirstCheckMachine = setup({
  types: {
    context: {} as AIFirstCheckContext,
    events: {} as
      | { type: 'TASK_ANALYSIS_COMPLETE'; characteristics: TaskCharacteristics }
      | { type: 'DIVISION_DECIDED'; decision: DivisionDecision }
      | { type: 'PROMPT_SELECTED'; technique: PromptTechnique },
  },
  guards: {
    isAiSuitable: ({ context }) =>
      context.taskCharacteristics?.isAiSuitable !== false,
    isAiLeadDecision: ({ context }) =>
      context.divisionDecision?.lead === 'ai',
  },
  actions: {
    analyzeTaskCharacteristics: () => { /* タスク特性分析 */ },
    decideDivision: () => { /* DT-6 評価ロジック */ },
    selectPromptTechnique: () => { /* DT-7 評価ロジック */ },
  },
}).createMachine({
  id: 'aiFirstCheck',
  initial: 'taskAnalysis',
  context: {
    taskCharacteristics: null,
    divisionDecision: null,
    promptTechnique: null,
  },
  states: {
    // --- タスク特性分析（SP2-T1） ---
    taskAnalysis: {
      entry: [{ type: 'analyzeTaskCharacteristics' }],
      on: {
        TASK_ANALYSIS_COMPLETE: [
          {
            guard: 'isAiSuitable',
            actions: assign({
              taskCharacteristics: ({ event }) => event.characteristics,
            }),
            target: 'divisionDecision',
          },
          {
            actions: assign({
              taskCharacteristics: ({ event }) => event.characteristics,
            }),
            target: 'humanLead',
          },
        ],
      },
    },

    // --- 分業決定（SP2-T2 / DT-6） ---
    divisionDecision: {
      entry: [{ type: 'decideDivision' }],
      on: {
        DIVISION_DECIDED: [
          {
            guard: 'isAiLeadDecision',
            actions: assign({
              divisionDecision: ({ event }) => event.decision,
            }),
            target: 'promptSelection',
          },
          {
            actions: assign({
              divisionDecision: ({ event }) => event.decision,
            }),
            target: 'humanLead',
          },
        ],
      },
    },

    // --- プロンプト技法選択（SP2-T3 / DT-7） ---
    promptSelection: {
      entry: [{ type: 'selectPromptTechnique' }],
      on: {
        PROMPT_SELECTED: {
          actions: assign({
            promptTechnique: ({ event }) => event.technique,
          }),
          target: 'aiLead',
        },
      },
    },

    // --- 最終状態 ---
    aiLead: { type: 'final' },     // SP2-EE-AI
    humanLead: { type: 'final' },   // SP2-EE-HM
  },

  output: ({ context }) => ({
    lead: context.divisionDecision?.lead ?? 'human',
    promptTechnique: context.promptTechnique,
  }),
});
```

---

## 5. 外部インターフェース定義

### 5.1 SP-3（検証フィードバックループ）への接続

SP-3はT7で内部定義される。T6では以下の外部インターフェースを定義する。

**SP-3への入力（前提条件）**:
- `humanExecution`または`humanReview`から遷移してくる
- どちらのパスからも同一の`verificationLoop`状態に合流する（INV-MF5）

**SP-3からの出力（`output`）**:
```typescript
type SP3Output = {
  passed: boolean;    // 全検証成功
  lossCut: boolean;   // 損切り判断に至ったか
};
```

**親マシンでの受け取り**:
- `passed === true` → `taskComplete`（EE-1）
- `passed === false` → `lossCutExit`（EE-2）

### 5.2 TD（タスク分解）への接続

TD（タスク分解サブプロセス）はT-2（`l0l3Adjust`）から呼び出される。

**設計判断**: TDのステートチャート定義はT6のスコープ外とする。`l0l3Adjust`のentryアクション内でTDの概念的な処理（5ステップ分解）を実行し、完了後に`L0L3_ADJUSTMENT_COMPLETE`イベントを発火する設計とする。TDを独立したネスト状態にするかは、Step Cの実装時に判断する。

---

## 6. 不変条件の保証マッピング

### 6.1 INV-MF: メインフロー不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-MF1** | すべてのタスクはBright Linesチェック（GW-1）を通過した後にのみ実行される | §3.6: `brightLinesCheck`がinitial状態。違反なしの場合のみ`l0l3Check`に遷移 | `brightLinesCheck`は全状態遷移の入口。フォールバック（違反なし）でのみ後続に進める。他の状態から`l0l3Check`への直接遷移は定義されていない |
| **INV-MF2** | Bright Lines違反検出時、是正完了まで進行しない | §3.6: `brightLinesFix`→`brightLinesCheck`のループ | `brightLinesFix`から後続状態への直接遷移は定義されていない。必ず`brightLinesCheck`に戻り、再評価される |
| **INV-MF3** | SP-1不合格時、SP-2には進めない | §3.6: `l0l3Check`のonDone。`isSP1Passed`がfalseの場合`l0l3Adjust`→`l0l3Check`のループ | `aiFirstCheck`への遷移は`isSP1Passed`ガードがtrueの場合のみ |
| **INV-MF4** | AI適用パスで人間レビュー（T-5）は省略できない | §3.6: `aiGeneration`→`humanReview`→`verificationLoop`の一方向遷移 | `aiGeneration`から`verificationLoop`への直接遷移は定義されていない。必ず`humanReview`を経由 |
| **INV-MF5** | 人間主導パスもAI適用パスもSP-3を通過する | §3.6: `humanExecution`→`verificationLoop`、`humanReview`→`verificationLoop` | 両パスともに遷移先が`verificationLoop`。他の最終状態への直接遷移は定義されていない |
| **INV-MF6** | メインフローの終了は`taskComplete`または`lossCutExit`の2つのみ | §3.3: 最終状態が`taskComplete`と`lossCutExit`の2つのみ | 状態定義に2つの最終状態しか存在しない |

### 6.2 INV-SP2: AIファーストチェック不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-SP2-1** | タスク特性の分析（SP2-T1）は分業判断の前に必ず実行される | §4.3: `taskAnalysis`がinitial状態 | `initial: 'taskAnalysis'`により、SP-2開始時に必ず`taskAnalysis`から始まる。`divisionDecision`への遷移は`taskAnalysis`からのみ |
| **INV-SP2-2** | AI主導の場合、プロンプト技法の選択は省略できない | §4.6: `divisionDecision`→`promptSelection`→`aiLead`の一方向遷移 | `divisionDecision`から`aiLead`への直接遷移は定義されていない。AI主導の場合は必ず`promptSelection`を経由 |
| **INV-SP2-3** | 分業結果は「AI主導」または「人間主導」の2つのみ | §4.3: 最終状態が`aiLead`と`humanLead`の2つのみ | 状態定義に2つの最終状態しか存在しない。すべての遷移パスはこの2状態のいずれかに到達 |
| **INV-SP2-4** | DT-6の各ルールは排他的条件である | §4.7: DT-6ヒットポリシーU + `decideDivision`の評価ロジック | 排他性はDT-6の決定表設計で保証（T2で定義済み）。ガード`isAiLeadDecision`は結果を判定するのみ |

### 6.3 INV-H（T5申し送り）の対応状況

| ID | T5での保証 | T6での追加保証 |
|----|-----------|--------------|
| **INV-H3** | SP-1のonDone + isSP1Passedガード | メインフローの状態遷移で構造的に保証（§3.6）。SP-1を経由せずにSP-2に到達する経路は存在しない |
| **INV-H4** | 逐次評価による構造的上位優先 | DT-10の参照はT8統合仕様で対応。T6では、各レベルの順序がメインフロー構造で固定されていることを保証 |
| **INV-H5** | 前提条件として明記 | `brightLinesCheck`がinitial状態であることで構造的に保証（§3.6）。Bright Lines通過前にSP-1に進む経路は存在しない |

---

## 7. 出典

| ファイル | 参照セクション |
|---------|--------------|
| `phase1-t3-bpmn-elements.md` | §1: メインフロー要素、§3: SP-2要素 |
| `phase1-t4-mermaid-flowcharts.md` | §1: メインフロー図 |
| `phase1-t7-invariants.md` | INV-MF1〜MF6, INV-SP2-1〜SP2-4 |
| `docs/bpmn-to-statechart-mapping.md` | MR-1〜MR-5, MR-9 |
| `docs/spec-l0l4-hierarchy.md` | T5: SP-1仕様、output定義 |
| `phase1-t2-dmn-decision-tables.md` | DT-0, DT-6, DT-7 |

---

## 8. T7への申し送り

| # | 項目 | 内容 |
|---|------|------|
| 1 | SP-3の外部インターフェース | §5.1で定義。`output`は`{ passed: boolean; lossCut: boolean }`。T7はこのインターフェースに準拠してSP-3内部を定義すること |
| 2 | SP-3への入力経路 | `humanExecution`と`humanReview`の2箇所から合流。T7ではどちらのパスからも同一の初期状態に遷移する設計とすること |
| 3 | SP-3からメインフローへの通知 | `verificationLoop`のonDoneで`isVerificationPassed`ガードを評価。trueなら`taskComplete`、falseなら`lossCutExit` |
| 4 | 損切り判断への接続 | SP-3内部で損切り判断（LC）が発生した場合、SP-3のoutputに`lossCut: true`を設定してメインフローの`lossCutExit`に遷移する設計。LCの内部はT7で定義 |
| 5 | 命名規則の継続 | イベント: SCREAMING_SNAKE、状態: camelCase、ガード: camelCase（T5から継続） |
