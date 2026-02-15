# T5: L0-L4階層のステートチャート仕様

> **タスク**: L0→L1→L2→L3→L4の5層意思決定階層を、Harelステートチャートのネスト構造として形式的に定義する
> **検証方法**: INV-H1〜H5、INV-SP1-1〜SP1-5の10個の不変条件が設計上すべて保証されていること
> **自信度**: おそらく（ネスト設計の最適な深さは実装時に調整の可能性あり）
> **入力**: `phase1-t1-decision-conditions.md`、`phase1-t2-dmn-decision-tables.md`（DT-1〜DT-5）、`phase1-t7-invariants.md`（INV-H, INV-SP1）、`docs/bpmn-to-statechart-mapping.md`（MR-1〜MR-5）

---

## 1. スコープと設計方針

### 1.1 T5のスコープ

| 対象 | 詳細度 | 理由 |
|------|--------|------|
| SP-1（L0-L3チェック） | **内部詳細定義** | INV-SP1-1〜SP1-5の保証にSP-1内部構造が必要 |
| L4（SP-2への接続） | **外部インターフェースのみ** | INV-H3の保証に「SP-1通過後にのみ到達可能」が必要。SP-2内部はT6 |
| DT-0（Bright Lines） | **前提条件として定義** | INV-H5の保証に「全レベルに先行」が必要。ガード詳細はT6 |

### 1.2 設計判断（T4申し送り#5の確定）

| # | 設計判断 | 選択 | 理由 |
|---|---------|------|------|
| 5 | イベント名の命名規則 | **A: SCREAMING_SNAKE** | XState慣例に従う。T5〜T8全体で統一 |

---

## 2. 階層全体のスケルトン

INV-H1〜H5を保証するため、L0-L4階層全体の構造を最上位で定義する。

### 2.1 状態階層図

```
hierarchyRoot（最上位マシン）
├── [前提] Bright Linesチェック通過済み（INV-H5）
│
├── l0l3Check（SP-1: compound state）← 本T5で詳細定義
│   ├── l0Check
│   ├── l1Check
│   ├── l2Check
│   ├── l3Check
│   ├── passed（final）
│   └── failed（final）
│
├── aiFirstCheck（SP-2: compound state）← T6で詳細定義
│   ├── （内部はT6で定義）
│   ├── aiLead（final）
│   └── humanLead（final）
│
└── 実行フェーズ ← T6で詳細定義
```

### 2.2 状態遷移の全体フロー

```
[Bright Lines通過済み]
    │
    ▼
l0l3Check（SP-1）
    │
    ├── onDone [SP-1通過] → aiFirstCheck（SP-2）  ← INV-H3保証
    │
    └── onDone [SP-1不合格] → l0l3Adjust → l0l3Check（ループ）
                                                    ← INV-H2保証
```

**INV-H5の実現方法**: Bright Linesチェック（DT-0）はメインフローのGW-1で実施される（T6で定義）。T5のスコープでは「SP-1に到達する時点でBright Lines通過済みである」ことを前提条件として明記する。この前提は、T6でメインフローの`brightLinesCheck`状態としてガード付き遷移を定義することで構造的に保証される。

---

## 3. SP-1（L0-L3チェック）の詳細仕様

### 3.1 マシン定義

**マシンID**: `l0l3CheckMachine`
**種類**: compound state（SP-1はメインフローのネスト状態として組み込まれる）

### 3.2 コンテキスト定義

```typescript
type L0L3CheckContext = {
  /** 各レベルの評価結果を保持 */
  evaluationResults: {
    l0: LevelResult | null;  // DT-2の評価結果
    l1: LevelResult | null;  // DT-3の評価結果
    l2: LevelResult | null;  // DT-4の評価結果
    l3: LevelResult | null;  // DT-5の評価結果
  };
};

type LevelResult = {
  passed: boolean;
  /** 要対応項目のリスト（ヒットポリシーC: 収集） */
  issues: string[];
};
```

**設計根拠**: DT-2〜DT-5のヒットポリシーC（収集）を反映。各レベルで全ルールを評価し、`issues`に「要対応」をすべて収集する。`issues.length > 0`のとき`passed = false`（INV-SP1-5）。

### 3.3 状態定義

| 状態名 | 種別 | entry アクション | 対応BPMN要素 |
|--------|------|-----------------|-------------|
| `l0Check` | 通常状態（initial） | `evaluateL0` | SP1-T1 |
| `l1Check` | 通常状態 | `evaluateL1` | SP1-T2 |
| `l2Check` | 通常状態 | `evaluateL2` | SP1-T3 |
| `l3Check` | 通常状態 | `evaluateL3` | SP1-T4 |
| `passed` | 最終状態 | — | SP1-EE-OK |
| `failed` | 最終状態 | — | SP1-EE-NG |

**適用マッピングルール**: MR-1（SE → initial）、MR-2（EE → final）、MR-3（タスク → 状態 + entry）

### 3.4 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `L0_EVALUATION_COMPLETE` | L0チェック（DT-2）の評価が完了した | `evaluateL0`アクション |
| `L1_EVALUATION_COMPLETE` | L1チェック（DT-3）の評価が完了した | `evaluateL1`アクション |
| `L2_EVALUATION_COMPLETE` | L2チェック（DT-4）の評価が完了した | `evaluateL2`アクション |
| `L3_EVALUATION_COMPLETE` | L3チェック（DT-5）の評価が完了した | `evaluateL3`アクション |

### 3.5 ガード条件定義

| ガード名 | 条件 | 対応GW | 対応DT |
|---------|------|--------|--------|
| `isL0Pass` | `context.evaluationResults.l0.passed === true` | SP1-GW1 | DT-2: 全ルール通過 |
| `isL1Pass` | `context.evaluationResults.l1.passed === true` | SP1-GW2 | DT-3: 全ルール通過 |
| `isL2Pass` | `context.evaluationResults.l2.passed === true` | SP1-GW3 | DT-4: 全ルール通過 |
| `isL3Pass` | `context.evaluationResults.l3.passed === true` | SP1-GW4 | DT-5: 全ルール通過 |

**適用マッピングルール**: MR-5（排他GW → ガード付き遷移配列）

**DT-1のヒットポリシーF（最初一致）との対応**: ガード配列は上から順に評価され、最初にtrueになったものが採用される。各レベルの`EVALUATION_COMPLETE`イベントに対するガード配列がこれを実現する（T2 Step Aで確認済み）。

### 3.6 アクション定義

| アクション名 | 種別 | 処理内容 |
|-------------|------|---------|
| `evaluateL0` | entry | DT-2の6原則（G1-G3, R1-R3）を評価し、結果を`context.evaluationResults.l0`に格納 |
| `evaluateL1` | entry | DT-3の3原則（P1-P3）を評価し、結果を`context.evaluationResults.l1`に格納 |
| `evaluateL2` | entry | DT-4の3原則（H1-H3）を評価し、結果を`context.evaluationResults.l2`に格納 |
| `evaluateL3` | entry | DT-5の4原則（S1-S3 + YAGNI）を評価し、結果を`context.evaluationResults.l3`に格納 |
| `assignL0Result` | assign | `L0_EVALUATION_COMPLETE`時に評価結果をcontextに保存 |
| `assignL1Result` | assign | `L1_EVALUATION_COMPLETE`時に評価結果をcontextに保存 |
| `assignL2Result` | assign | `L2_EVALUATION_COMPLETE`時に評価結果をcontextに保存 |
| `assignL3Result` | assign | `L3_EVALUATION_COMPLETE`時に評価結果をcontextに保存 |

### 3.7 遷移表

| 現在の状態 | イベント | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|---------|--------|-----------|--------|------------|
| `l0Check` | `L0_EVALUATION_COMPLETE` | `isL0Pass` | `assignL0Result` | `l1Check` | INV-SP1-1, INV-SP1-2 |
| `l0Check` | `L0_EVALUATION_COMPLETE` | （フォールバック） | `assignL0Result` | `failed` | INV-SP1-3 |
| `l1Check` | `L1_EVALUATION_COMPLETE` | `isL1Pass` | `assignL1Result` | `l2Check` | INV-SP1-2 |
| `l1Check` | `L1_EVALUATION_COMPLETE` | （フォールバック） | `assignL1Result` | `failed` | INV-SP1-3 |
| `l2Check` | `L2_EVALUATION_COMPLETE` | `isL2Pass` | `assignL2Result` | `l3Check` | INV-SP1-2 |
| `l2Check` | `L2_EVALUATION_COMPLETE` | （フォールバック） | `assignL2Result` | `failed` | INV-SP1-3 |
| `l3Check` | `L3_EVALUATION_COMPLETE` | `isL3Pass` | `assignL3Result` | `passed` | INV-SP1-2 |
| `l3Check` | `L3_EVALUATION_COMPLETE` | （フォールバック） | `assignL3Result` | `failed` | INV-SP1-3 |

**適用パターン**: MR-5の「二分岐（Yes/No）」パターン。各レベルで同一構造を繰り返す。

### 3.8 XState v5 擬似コード

```typescript
import { setup, assign } from 'xstate';

const l0l3CheckMachine = setup({
  types: {
    context: {} as L0L3CheckContext,
    events: {} as
      | { type: 'L0_EVALUATION_COMPLETE'; result: LevelResult }
      | { type: 'L1_EVALUATION_COMPLETE'; result: LevelResult }
      | { type: 'L2_EVALUATION_COMPLETE'; result: LevelResult }
      | { type: 'L3_EVALUATION_COMPLETE'; result: LevelResult },
  },
  guards: {
    isL0Pass: ({ context }) => context.evaluationResults.l0?.passed === true,
    isL1Pass: ({ context }) => context.evaluationResults.l1?.passed === true,
    isL2Pass: ({ context }) => context.evaluationResults.l2?.passed === true,
    isL3Pass: ({ context }) => context.evaluationResults.l3?.passed === true,
  },
  actions: {
    evaluateL0: () => { /* DT-2 評価ロジック */ },
    evaluateL1: () => { /* DT-3 評価ロジック */ },
    evaluateL2: () => { /* DT-4 評価ロジック */ },
    evaluateL3: () => { /* DT-5 評価ロジック */ },
  },
}).createMachine({
  id: 'l0l3Check',
  initial: 'l0Check',
  context: {
    evaluationResults: { l0: null, l1: null, l2: null, l3: null },
  },
  states: {
    // --- L0 持続可能性チェック（DT-2: G1-G3, R1-R3） ---
    l0Check: {
      entry: [{ type: 'evaluateL0' }],
      on: {
        L0_EVALUATION_COMPLETE: [
          {
            guard: 'isL0Pass',
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l0: event.result,
              }),
            }),
            target: 'l1Check',
          },
          {
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l0: event.result,
              }),
            }),
            target: 'failed',
          },
        ],
      },
    },

    // --- L1 心理的安全性チェック（DT-3: P1-P3） ---
    l1Check: {
      entry: [{ type: 'evaluateL1' }],
      on: {
        L1_EVALUATION_COMPLETE: [
          {
            guard: 'isL1Pass',
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l1: event.result,
              }),
            }),
            target: 'l2Check',
          },
          {
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l1: event.result,
              }),
            }),
            target: 'failed',
          },
        ],
      },
    },

    // --- L2 急がば回れチェック（DT-4: H1-H3） ---
    l2Check: {
      entry: [{ type: 'evaluateL2' }],
      on: {
        L2_EVALUATION_COMPLETE: [
          {
            guard: 'isL2Pass',
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l2: event.result,
              }),
            }),
            target: 'l3Check',
          },
          {
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l2: event.result,
              }),
            }),
            target: 'failed',
          },
        ],
      },
    },

    // --- L3 シンプルさチェック（DT-5: S1-S3 + YAGNI） ---
    l3Check: {
      entry: [{ type: 'evaluateL3' }],
      on: {
        L3_EVALUATION_COMPLETE: [
          {
            guard: 'isL3Pass',
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l3: event.result,
              }),
            }),
            target: 'passed',
          },
          {
            actions: assign({
              evaluationResults: ({ context, event }) => ({
                ...context.evaluationResults,
                l3: event.result,
              }),
            }),
            target: 'failed',
          },
        ],
      },
    },

    // --- 最終状態 ---
    passed: { type: 'final' },  // SP1-EE-OK
    failed: { type: 'final' },  // SP1-EE-NG
  },

  // SP-1の結果をoutputで親に通知
  output: ({ context }) => ({
    result: context.evaluationResults,
    allPassed: Object.values(context.evaluationResults)
      .every(r => r?.passed === true),
  }),
});
```

---

## 4. SP-1と上位階層の接続仕様（外部インターフェース）

SP-1がメインフローにネスト状態として組み込まれた際の、親マシンとの接続を定義する。

### 4.1 親マシンからの呼び出し

```typescript
// メインフローの一部（T6で詳細定義）
l0l3Check: {
  // SP-1をネスト状態として配置
  ...l0l3CheckMachine,

  // SP-1完了時の分岐
  onDone: [
    {
      guard: 'isSP1Passed',   // output.allPassed === true
      target: 'aiFirstCheck', // → SP-2（L4）へ（INV-H3保証）
    },
    {
      target: 'l0l3Adjust',   // → 調整 → 再チェック（INV-MF3保証）
    },
  ],
},
```

### 4.2 onDone通知方法（T4申し送り#4の決定）

| # | 設計判断 | 選択 | 理由 |
|---|---------|------|------|
| 4 | SP-1のonDone通知方法 | **B: `output`プロパティ** | XState v5の`output`は最終状態到達時に値を返すAPI。SP-1は`passed`/`failed`の2つの最終状態を持つが、`output`でallPassedフラグを返すことで、親のガード条件がシンプルになる。最終状態IDの判定（方法A）は、XState v5で最終状態IDを取得するAPIが冗長なため不採用 |

### 4.3 L4（SP-2）への接続の前提条件

SP-2（`aiFirstCheck`）に到達するための必要条件:

1. **Bright Lines通過済み**（INV-H5）: メインフローのGW-1で保証（T6で定義）
2. **SP-1通過済み**（INV-H3）: `isSP1Passed`ガードで保証（本仕様）
3. **L0→L1→L2→L3の順序**（INV-H1）: SP-1内部の逐次遷移で保証（本仕様 §3）

これにより、SP-2に到達した時点でL0〜L3がすべて通過していることが構造的に保証される。

---

## 5. DT-1〜DT-5のガード条件マッピング

### 5.1 DT-1（メインフロー）のヒットポリシーF → 遷移ロジック

DT-1のヒットポリシーF（最初一致）は、SP-1の逐次状態遷移によって自然に実現される。

| DT-1ルール# | 評価レベル | 結果 | SP-1での対応 |
|-------------|----------|------|------------|
| 1 | L0 | No | `l0Check` → `failed`（最初のNo → 即座に不合格） |
| 2 | L0 | Yes | `l0Check` → `l1Check`（次レベルへ） |
| 3 | L1 | No | `l1Check` → `failed` |
| 4 | L1 | Yes | `l1Check` → `l2Check` |
| 5 | L2 | No | `l2Check` → `failed` |
| 6 | L2 | Yes | `l2Check` → `l3Check` |
| 7 | L3 | No | `l3Check` → `failed` |
| 8 | L3 | Yes | `l3Check` → `passed` |
| 9-10 | L4 | — | SP-2で処理（T6スコープ） |

**構造的保証**: 逐次状態遷移（l0→l1→l2→l3）の各段階で「No → failed」のフォールバック遷移を持つことで、ヒットポリシーFの「最初のNoで分岐」が状態構造として埋め込まれる。

### 5.2 DT-2〜DT-5（各レベル詳細）のヒットポリシーC → 評価ロジック

各レベルの`evaluate`アクション内で、ヒットポリシーC（収集）を実装する。

```typescript
// DT-2（L0詳細）の評価ロジック例
function evaluateL0(taskContext: TaskInfo): LevelResult {
  const issues: string[] = [];

  // G1: 属人化を避ける仕組みがあるか？
  if (!taskContext.hasKnowledgeSharingMechanism) {
    issues.push('G1: 属人化を避ける仕組みがない');
  }
  // G2: チームの成長に寄与するか？
  if (!taskContext.contributesToTeamGrowth) {
    issues.push('G2: チームの成長に寄与しない');
  }
  // G3: 技術的負債を増やさないか？
  if (taskContext.increasesTechnicalDebt) {
    issues.push('G3: 技術的負債を増やす');
  }
  // R1〜R3（回復性原則）も同様に評価...

  // ヒットポリシーC: 全ルール評価完了
  // 集約ルール: issues.length > 0 → 不合格（INV-SP1-5）
  return {
    passed: issues.length === 0,
    issues,
  };
}
```

**INV-SP1-5の保証**: `issues.length > 0`（1つでも「要対応」がある）のとき`passed = false`。これがDT-2〜DT-5の集約ルール「要対応が1つ以上 → 不合格」と一致する。

---

## 6. 不変条件の保証マッピング

T5の検証基準である10個の不変条件それぞれに対して、本仕様のどの設計要素で保証されるかを示す。

### 6.1 INV-H: 階層不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-H1** | L0〜L4の評価順序は常にL0→L1→L2→L3→L4であり、逆転しない | §3.7 遷移表: `l0Check`→`l1Check`→`l2Check`→`l3Check`の一方向遷移 | 状態間に逆方向の遷移が定義されていないことで構造的に保証。L4はSP-1通過後にのみ到達可能（§4.1） |
| **INV-H2** | 上位レベルでNoとなった場合、下位レベルの評価は行われない | §3.7 遷移表: 各レベルのフォールバック遷移が`failed`に直結 | `failed`は最終状態であり、そこから他の状態への遷移は不可能。Noの場合、下位レベルの状態に遷移する経路が存在しない |
| **INV-H3** | L4は常にL0〜L3のすべてを通過した後にのみ適用される | §4.1: `onDone`の`isSP1Passed`ガード + §4.3の前提条件 | SP-1の`passed`最終状態に到達するにはl0→l1→l2→l3の全4状態を通過する必要がある。`passed`到達時のみ`isSP1Passed`がtrueとなり、SP-2に遷移可能 |
| **INV-H4** | 競合発生時、上位レベルの判断が常に優先される | §5.1: DT-1ヒットポリシーFの逐次遷移による実現 | 逐次評価のため、上位レベルのNoが下位レベルの評価に先行する。競合以前に上位がNoならば下位は評価されない（DT-10はT6の統合仕様で参照） |
| **INV-H5** | Bright Linesは全レベルに先行する | §4.3: 前提条件として明記。構造的保証はT6で定義 | SP-1に到達する時点でBright Lines通過済みであることを前提条件とする。T6のメインフロー`brightLinesCheck`状態で構造的に保証 |

### 6.2 INV-SP1: L0-L3チェック不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-SP1-1** | L0チェックは常に最初に実行される | §3.3: `l0Check`がinitial状態 | `initial: 'l0Check'`により、SP-1開始時に必ず`l0Check`から始まる |
| **INV-SP1-2** | 各レベルのチェックは直前のレベルを通過した後にのみ実行される | §3.7: ガード`isLxPass`がtrueの場合のみ次レベルに遷移 | `l1Check`に遷移するには`isL0Pass`が必要。`l2Check`には`isL1Pass`が必要…という逐次依存が構造化されている |
| **INV-SP1-3** | いずれかのレベルでNoの場合、`failed`に遷移し他レベルの評価はスキップされる | §3.7: 各レベルのフォールバック遷移がすべて`failed`に直結 | ガードを通過できない場合（Noの場合）、フォールバックとして`failed`最終状態に遷移。`failed`からの遷移は定義されていない |
| **INV-SP1-4** | SP-1の結果は「全通過」または「不合格」の2つのみ | §3.3: 最終状態が`passed`と`failed`の2つのみ | 状態定義に2つの最終状態しか存在しない。すべての遷移パスはこの2状態のいずれかに到達して終了する |
| **INV-SP1-5** | 各レベルの詳細判断で「要対応」が1つでもあれば不合格 | §5.2: ヒットポリシーCの実装 + `issues.length > 0 → passed: false` | `evaluateLx`アクションが全ルールを収集評価し、1つでも`issues`があれば`passed = false`を返す。ガード`isLxPass`がfalseとなり`failed`に遷移 |

---

## 7. 出典

| ファイル | 参照セクション |
|---------|--------------|
| `phase1-t1-decision-conditions.md` | §1〜§3: L0-L4判断条件 |
| `phase1-t2-dmn-decision-tables.md` | DT-1〜DT-5: 各レベルの決定表定義 |
| `phase1-t7-invariants.md` | INV-H1〜H5, INV-SP1-1〜SP1-5 |
| `docs/bpmn-to-statechart-mapping.md` | MR-1〜MR-5: マッピングルール |
| `phase1-t3-bpmn-elements.md` | §2: SP-1フロー要素定義 |

---

## 8. T6への申し送り

| # | 項目 | 内容 |
|---|------|------|
| 1 | Bright Linesチェック（DT-0）のガード実装 | T5では前提条件として記述。T6でメインフローの`brightLinesCheck`状態としてガード付き遷移を定義すること |
| 2 | SP-2（L4: AIファーストチェック）の内部定義 | T5ではSP-1通過後の遷移先としてのみ定義。T6でSP-2の内部状態を詳細定義すること |
| 3 | `isSP1Passed`ガードの実装 | `output.allPassed`を参照。T6のメインフローで`onDone`のガード条件として使用すること |
| 4 | INV-H4（競合解決）の完全な実装 | DT-10の参照はT6の統合仕様で対応。T5では逐次評価による構造的な上位優先を保証 |
| 5 | 設計判断#4確定 | SP-1のonDone通知方法は`output`プロパティを採用（§4.2）。T6以降でも同方式を踏襲すること |
