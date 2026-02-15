# T8: 復帰フロー＋エスカレーションのステートチャート仕様

> **タスク**: RF（復帰フロー: 分析→アプローチ選択→CLAUDE.md記録→メインフロー復帰）とES（エスカレーション判断）のステートチャートを定義する
> **検証方法**: INV-RF1〜RF6、INV-ES1〜ES3の9個の不変条件が設計上すべて保証されていること
> **自信度**: おそらく（ESの配置方式とGW1/GW2の2段階構造に設計判断を含む）
> **入力**: `phase1-t5-losscutrecovery-bpmn.md`（§2: RF、§3: ES）、`phase1-t6-losscut-recovery-mermaid.md`（§2: RF、§3: ES）、`phase1-t7-invariants.md`（INV-RF1〜RF6, INV-ES1〜ES3）、`docs/bpmn-to-statechart-mapping.md`（MR-1〜MR-9）、`docs/spec-l0l4-hierarchy.md`（T5: output方式）、`docs/spec-mainflow-sp2.md`（T6: 外部IF）、`docs/spec-verification-losscut.md`（T7: SP-3コンテキスト、output定義）

---

## 1. スコープと設計判断

### 1.1 T8のスコープ

| 対象 | 詳細度 | 理由 |
|------|--------|------|
| RF（復帰フロー） | **内部詳細定義** | INV-RF1〜RF6の保証にRF内部構造が必要 |
| ES（エスカレーション判断） | **内部詳細定義** | INV-ES1〜ES3の保証にES内部構造が必要 |
| メインフローとの接続 | **T6外部IFの拡張** | SP-3のoutput拡張によるエラー情報引き渡しを定義 |

### 1.2 T7からの申し送り対応

| # | 申し送り | T8での対応 |
|---|---------|-----------|
| 1 | SP-3のerrorHistory/lastErrorをRFに渡す方法 | §3.2 SP3OutputExtended型で拡張定義、メインフローコンテキスト経由 |
| 2 | DT-9のA4違反時のメインフロー復帰 | T8スコープ外（メインフロー側の経路定義）。§8に申し送り記載 |
| 3 | LC-T2からtypecheckへの再開 | T7で対応済み。T8では変更なし |
| 4 | 設計判断#2の将来見直し | T7で対応済み。T8では変更なし |
| 5 | 命名規則の継続 | イベント: SCREAMING_SNAKE、状態: camelCase、ガード: camelCase |
| 6 | LCのonDone output方式 | RF/ESともに`output`プロパティで踏襲 |

### 1.3 設計判断の確定

#### 設計判断#1: ESのステートチャート配置

| 項目 | 内容 |
|------|------|
| **選択** | **A: RF内のネスト状態（compound state）** |
| **理由** | ESは7要素と小規模であり、T7でLCをSP-3内のcompound stateとして定義した前例（MR-4準拠）と一致する。ESの結果（ESC/SELF）がRFの後続フロー（RF-T8 or RF-GW2）に直接影響するため、RF内に閉じた方がデータの受け渡しがシンプルである |
| **不採用理由（B: invoke呼び出し）** | ESを別マシンとして切り離すと、ESの結果によるRF内の分岐処理にマシン間通信が必要になる。ESの複雑度（7要素）に対してinvokeのオーバーヘッドが不釣り合いである |

#### 設計判断#2: RF-GW1とRF-GW2の関係

| 項目 | 内容 |
|------|------|
| **選択** | **GW1はStep 1完了後のファストパス、GW2は通常のアプローチ選択として2段階逐次評価** |
| **理由** | RF-T3（Step 1完了）→ `escalationCheck`状態でガード`needsImmediateEscalation`を評価。trueならES直行、falseなら`approachSelection`に進む。GW2でD選択の場合もES状態に遷移するが、ES-EE-SELFの場合は`approachSelection`に戻ってアプローチを再選択する |
| **ESの共有** | RF-GW1とRF-GW2の2箇所からの呼び出しは、同一の`escalationJudgment` compound stateへの遷移として表現する |

#### 設計判断#3: Step 2のアプローチ実行後の合流方法

| 項目 | 内容 |
|------|------|
| **選択** | **各アプローチの完了イベントでStep 3（`learningRecord`）に直接遷移** |
| **理由** | T7でtypecheck/lint/testの各失敗がすべて`lossCutJudgment`に遷移するフォールバック合流パターンと同一。各アプローチ状態からの完了イベントが、すべて`recordToClaudeMd`（RF-T9）に直接遷移することで、INV-RF2（全パスからの必須合流）を構造的に保証する |

#### 設計判断#4: SP-3のerrorHistory/lastErrorのRFへの引き渡し

| 項目 | 内容 |
|------|------|
| **選択** | **SP-3のoutputを拡張し、メインフローのコンテキスト経由で引き渡す** |
| **理由** | SP-3のoutputに`lastError`と`errorHistory`を追加し、メインフローのonDone処理でコンテキストに格納する。RF起動時にメインフローコンテキストから参照する |
| **T6外部IFへの影響** | T6 §5.1のSP3Output型を拡張する必要がある。変更箇所は§3.10に記載 |

---

## 2. RF（復帰フロー）の全体構造

### 2.1 状態階層図

```
recoveryFlow（RF: compound state）
│
├── problemAnalysis（Step 1: compound state, initial）  ← INV-RF1
│   ├── verbalizeProblem（initial）           ← RF-T1
│   │   entry: [verbalizeProblem]
│   ├── analyzeCause                          ← RF-T2
│   │   entry: [analyzeCause]
│   └── identifyEssence                       ← RF-T3
│       entry: [identifyEssence]
│
├── escalationCheck                           ← RF-GW1: INV-RF5
│   （always遷移でガード評価）
│
├── escalationJudgment（ES: compound state）  ← RF-SP1: INV-ES1〜ES3
│   ├── checkImmediate（initial）             ← ES-GW1: INV-ES1
│   ├── executeImmediate                      ← ES-T1: INV-ES2
│   │   entry: [executeImmediateEscalation]
│   ├── check30Min                            ← ES-GW2
│   ├── consider30Min                         ← ES-T2
│   │   entry: [considerEscalation]
│   ├── escalationConfirmed（final）          ← ES-EE-ESC: INV-ES3
│   └── selfResolution（final）               ← ES-EE-SELF: INV-ES3
│
├── approachSelection                         ← RF-GW2: INV-RF6
│   （always遷移で4分岐ガード評価）
│
├── directResolution（A: compound state）     ← RF-T4, RF-T5
│   ├── humanDirectFix（initial）             ← RF-T4
│   │   entry: [humanDirectFix]
│   └── askAiExplanation                      ← RF-T5
│       entry: [askAiExplanation]
│
├── redecompose                               ← RF-T6（B）
│   entry: [redecomposeProblem]
│
├── resetContext                              ← RF-T7（C）
│   entry: [resetContext]
│
├── consultTeam                               ← RF-T8（D: ES後）
│   entry: [consultTeam]
│
├── recordToClaudeMd                          ← RF-T9: INV-RF2（必須合流点）
│   entry: [recordFailurePattern]
│
├── documentWorkaround                        ← RF-T10: INV-RF3
│   entry: [documentWorkaround]
│
├── teamShareDecision                         ← RF-GW3
│   （always遷移でガード評価）
│
├── shareWithTeam                             ← RF-T11
│   entry: [shareWithTeam]
│
└── recoveryComplete（final）                 ← RF-EE: INV-RF4
```

### 2.2 状態遷移の全体フロー

```
[メインフロー lossCutExit から遷移]
    │
    ▼
problemAnalysis（Step 1: initial）                       ← INV-RF1
    │
    verbalizeProblem → analyzeCause → identifyEssence
    │                                      │
    │                                  onDone
    ▼                                      ▼
                                escalationCheck（RF-GW1）  ← INV-RF5
                                    │
                    ┌───────────────┤
                    │               │
         [needsImmediate            [!needsImmediate
          Escalation]                Escalation]
                    │               │
                    ▼               ▼
        escalationJudgment    approachSelection（RF-GW2）  ← INV-RF6
                │                   │
        ┌───────┤           ┌───────┼───────┬───────┐
        │       │           │       │       │       │
   [ESC]    [SELF]      [A:直接] [B:再分解] [C:ﾘｾｯﾄ] [D:ｴｽｶﾚ]
        │       │           │       │       │       │
        ▼       ▼           ▼       ▼       ▼       ▼
   consultTeam  approachSelection  directRes redecomp resetCtx escalationJudgment
        │       （再選択）         │       │       │       │
        │                          │       │       │   ┌───┤
        │                          │       │       │ [ESC] [SELF]
        │                          │       │       │   │    │
        │                          │       │       │   ▼    ▼
        │                          │       │       │ consultTeam approachSelection
        │                          │       │       │   │
        └──────────────────────────┴───────┴───────┴───┘
                                    │
                                    ▼
                        recordToClaudeMd（RF-T9）          ← INV-RF2（必須）
                                    │
                                    ▼
                        documentWorkaround（RF-T10）       ← INV-RF3
                                    │
                                    ▼
                        teamShareDecision（RF-GW3）
                            │               │
                    [shouldShare]     [!shouldShare]
                            │               │
                            ▼               │
                    shareWithTeam           │
                            │               │
                            └───────┬───────┘
                                    │
                                    ▼
                        recoveryComplete（final）          ← INV-RF4
                                    │
                            output: { recovered: true }
                                    │
                                    ▼
                    [メインフロー SE-1 に復帰]
```

---

## 3. RFの詳細仕様

### 3.1 マシン定義

**マシンID**: `recoveryFlowMachine`
**種類**: compound state（メインフローの`recoveryFlow`としてネスト）

### 3.2 コンテキスト定義

```typescript
type RecoveryFlowContext = {
  /** SP-3から引き継いだ直近のエラー内容 */
  lastError: ErrorInfo | null;
  /** SP-3から引き継いだエラー修正履歴 */
  errorHistory: ErrorRecord[];
  /** Step 1（問題分析）の結果 */
  analysisResult: ProblemAnalysis | null;
  /** 選択されたアプローチ */
  selectedApproach: 'A' | 'B' | 'C' | 'D' | null;
  /** エスカレーション判断の結果 */
  escalationResult: 'escalate' | 'self' | null;
  /** CLAUDE.mdへの記録内容 */
  failureRecord: FailureRecord | null;
  /** チーム共有の判断結果 */
  shouldShareWithTeam: boolean;
};

type ProblemAnalysis = {
  /** 問題の言語化結果（RF-T1） */
  verbalization: string;
  /** 原因分析結果（RF-T2） */
  causeAnalysis: string;
  /** 本質の特定結果（RF-T3） */
  essenceIdentification: string;
  /** エスカレーション判断の入力となるフラグ */
  hasSecurityIssue: boolean;
  hasProductionImpact: boolean;
  hasDataLossRisk: boolean;
  retreatCount: number;
  isUnknownCause: boolean;
  isOutOfSkillScope: boolean;
};

type FailureRecord = {
  /** 失敗パターンの記述 */
  pattern: string;
  /** 回避策の記述 */
  workaround: string;
  /** 記録日時 */
  recordedAt: number;
};

/** T7で定義済みの型（再掲） */
type ErrorInfo = {
  step: 'typecheck' | 'lint' | 'test';
  message: string;
  timestamp: number;
};

type ErrorRecord = {
  error: ErrorInfo;
  fixAttempt: string;
  complexityDelta: 'increased' | 'unchanged' | 'decreased';
};
```

**設計根拠**: `lastError`と`errorHistory`はSP-3からメインフローのコンテキスト経由で引き継ぐ（設計判断#4）。`analysisResult`はStep 1の出力であり、RF-GW1/GW2のガード評価とES-GW1/GW2の判定条件の入力として使用する。

### 3.3 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `problemAnalysis` | compound state（initial） | — | RF-T1〜T3の親状態 |
| `verbalizeProblem` | 通常状態（problemAnalysisのinitial） | `verbalizeProblem` | RF-T1 |
| `analyzeCause` | 通常状態 | `analyzeCause` | RF-T2 |
| `identifyEssence` | 通常状態 | `identifyEssence` | RF-T3 |
| `escalationCheck` | 判定状態 | — | RF-GW1 |
| `escalationJudgment` | compound state | — | RF-SP1（ES）→ §4で詳細定義 |
| `approachSelection` | 判定状態 | — | RF-GW2 |
| `directResolution` | compound state | — | RF-T4〜T5（A: 直接解決） |
| `humanDirectFix` | 通常状態（directResolutionのinitial） | `humanDirectFix` | RF-T4 |
| `askAiExplanation` | 通常状態 | `askAiExplanation` | RF-T5 |
| `redecompose` | 通常状態 | `redecomposeProblem` | RF-T6（B: 再分解） |
| `resetContext` | 通常状態 | `resetContext` | RF-T7（C: リセット） |
| `consultTeam` | 通常状態 | `consultTeam` | RF-T8（D: チーム相談） |
| `recordToClaudeMd` | 通常状態 | `recordFailurePattern` | RF-T9（必須） |
| `documentWorkaround` | 通常状態 | `documentWorkaround` | RF-T10 |
| `teamShareDecision` | 判定状態 | — | RF-GW3 |
| `shareWithTeam` | 通常状態 | `shareWithTeam` | RF-T11 |
| `recoveryComplete` | 最終状態 | — | RF-EE |

**適用マッピングルール**: MR-1（RF-SE → `initial: 'problemAnalysis'`）、MR-2（RF-EE → final）、MR-3（RF-T1〜T11 → 状態 + entry）、MR-4（RF-SP1(ES) → ネスト状態）、MR-5（RF-GW1〜GW3 → ガード付き遷移）

### 3.4 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `PROBLEM_VERBALIZED` | 問題の言語化が完了した | `verbalizeProblem`アクション |
| `CAUSE_ANALYZED` | 原因分析が完了した | `analyzeCause`アクション |
| `ESSENCE_IDENTIFIED` | 本質の特定が完了した | `identifyEssence`アクション |
| `ESCALATION_DECIDED` | エスカレーション判断のアクションが完了した | `executeImmediateEscalation`/`considerEscalation`アクション |
| `HUMAN_FIX_COMPLETE` | 人間による直接解決が完了した | `humanDirectFix`アクション |
| `AI_EXPLANATION_RECEIVED` | AIからの説明取得が完了した | `askAiExplanation`アクション |
| `REDECOMPOSE_COMPLETE` | 問題の再分解が完了した | `redecomposeProblem`アクション |
| `CONTEXT_RESET_COMPLETE` | コンテキストのリセットが完了した | `resetContext`アクション |
| `TEAM_CONSULTED` | チームへの相談が完了した | `consultTeam`アクション |
| `CLAUDE_MD_RECORDED` | CLAUDE.mdへの記録が完了した | `recordFailurePattern`アクション |
| `WORKAROUND_DOCUMENTED` | 回避策の文書化が完了した | `documentWorkaround`アクション |
| `TEAM_SHARED` | チームへの共有が完了した | `shareWithTeam`アクション |

### 3.5 ガード条件定義

| ガード名 | 条件 | 対応GW | 保証する不変条件 |
|---------|------|--------|----------------|
| `needsImmediateEscalation` | `context.analysisResult`のセキュリティ/本番影響/データ損失フラグのいずれかがtrue | RF-GW1 | INV-RF5（Step 1完了後にのみ評価） |
| `isApproachA` | `context.selectedApproach === 'A'` | RF-GW2 | INV-RF6 |
| `isApproachB` | `context.selectedApproach === 'B'` | RF-GW2 | INV-RF6 |
| `isApproachC` | `context.selectedApproach === 'C'` | RF-GW2 | INV-RF6 |
| `isApproachD` | `context.selectedApproach === 'D'`（フォールバック） | RF-GW2 | INV-RF6 |
| `shouldShareWithTeam` | `context.shouldShareWithTeam === true` | RF-GW3 | — |
| `isEscalationConfirmed` | ES内の最終状態が`escalationConfirmed` | ES onDone | INV-ES3 |

### 3.6 アクション定義

| アクション名 | 種別 | 処理内容 |
|-------------|------|---------|
| `verbalizeProblem` | entry | エラー情報（lastError, errorHistory）をもとに問題を言語化する |
| `analyzeCause` | entry | 言語化された問題の原因を分析する |
| `identifyEssence` | entry | 原因分析から問題の本質を特定し、`context.analysisResult`に格納する |
| `executeImmediateEscalation` | entry | 即座にエスカレーションを実施する（セキュリティ/本番/データ損失） |
| `considerEscalation` | entry | 30分以内にエスカレーションを検討する（3回撤退/原因不明/スキル範囲外） |
| `selectApproach` | assign | 問題分析結果に基づきアプローチ（A/B/C/D）を`context.selectedApproach`に設定する |
| `humanDirectFix` | entry | 人間が直接問題を解決する |
| `askAiExplanation` | entry | AIに問題の説明と解決策を求める |
| `redecomposeProblem` | entry | 問題をより小さな単位に再分解する |
| `resetContext` | entry | AIのコンテキストをリセットする（新しいセッション開始） |
| `consultTeam` | entry | チームメンバーに問題を相談する |
| `recordFailurePattern` | entry | 失敗パターンをCLAUDE.md（D-1）に記録する。`context.failureRecord`に記録内容を格納 |
| `documentWorkaround` | entry | 次回の回避策を失敗パターン記録（D-5）に文書化する |
| `shareWithTeam` | entry | チーム共有記録（D-8）を作成し、外部ナレッジベースに出力する |
| `setEscalationResult` | assign | ESの結果を`context.escalationResult`に格納する |

### 3.7 遷移表

| 現在の状態 | イベント/条件 | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|-------------|--------|-----------|--------|------------|
| `verbalizeProblem` | `PROBLEM_VERBALIZED` | — | — | `analyzeCause` | INV-RF1 |
| `analyzeCause` | `CAUSE_ANALYZED` | — | — | `identifyEssence` | INV-RF1 |
| `identifyEssence` | `ESSENCE_IDENTIFIED` | — | — | `escalationCheck` | INV-RF1, INV-RF5 |
| `escalationCheck` | always | `needsImmediateEscalation` | — | `escalationJudgment` | INV-RF5 |
| `escalationCheck` | always | （フォールバック） | — | `approachSelection` | INV-RF5 |
| `escalationJudgment` | onDone | `isEscalationConfirmed` | `setEscalationResult` | `consultTeam` | INV-ES3 |
| `escalationJudgment` | onDone | （フォールバック） | `setEscalationResult` | `approachSelection` | INV-ES3, MR-9 |
| `approachSelection` | always | `isApproachA` | — | `directResolution` | INV-RF6 |
| `approachSelection` | always | `isApproachB` | — | `redecompose` | INV-RF6 |
| `approachSelection` | always | `isApproachC` | — | `resetContext` | INV-RF6 |
| `approachSelection` | always | （フォールバック: D） | — | `escalationJudgment` | INV-RF6 |
| `humanDirectFix` | `HUMAN_FIX_COMPLETE` | — | — | `askAiExplanation` | — |
| `askAiExplanation` | `AI_EXPLANATION_RECEIVED` | — | — | `recordToClaudeMd` | INV-RF2 |
| `redecompose` | `REDECOMPOSE_COMPLETE` | — | — | `recordToClaudeMd` | INV-RF2 |
| `resetContext` | `CONTEXT_RESET_COMPLETE` | — | — | `recordToClaudeMd` | INV-RF2 |
| `consultTeam` | `TEAM_CONSULTED` | — | — | `recordToClaudeMd` | INV-RF2 |
| `recordToClaudeMd` | `CLAUDE_MD_RECORDED` | — | — | `documentWorkaround` | INV-RF3 |
| `documentWorkaround` | `WORKAROUND_DOCUMENTED` | — | — | `teamShareDecision` | INV-RF3 |
| `teamShareDecision` | always | `shouldShareWithTeam` | — | `shareWithTeam` | — |
| `teamShareDecision` | always | （フォールバック） | — | `recoveryComplete` | INV-RF4 |
| `shareWithTeam` | `TEAM_SHARED` | — | — | `recoveryComplete` | INV-RF4 |

**全パスからRF-T9への合流の構造的保証**: `recordToClaudeMd`への遷移元を以下のとおり網羅している。

| パス | 遷移元 → recordToClaudeMd |
|------|--------------------------|
| A: 直接解決 | `askAiExplanation` → `recordToClaudeMd` |
| B: 再分解 | `redecompose` → `recordToClaudeMd` |
| C: リセット | `resetContext` → `recordToClaudeMd` |
| D: エスカレーション | `consultTeam` → `recordToClaudeMd` |
| GW1直行エスカレーション | `consultTeam` → `recordToClaudeMd` |

5つの合流パスすべてが`recordToClaudeMd`に直結しており、他の状態への遷移経路は定義されていない（INV-RF2保証）。

### 3.8 RFのoutput定義（メインフローとの接続）

```typescript
// RF完了時のoutput
output: () => ({
  recovered: true,
}),
```

メインフローはRFのonDoneで`recovered: true`を受け取り、SE-1（メインフロー開始状態）に遷移する（INV-RF4, INV-CF4）。

### 3.9 approachSelection状態の設計補足

`approachSelection`は`always`遷移で4分岐するが、その前にアプローチ選択の入力が必要である。この状態に入る前に`selectApproach`アクションが実行され、`context.selectedApproach`に選択結果が設定される。

具体的には、`approachSelection`状態のentry時に`selectApproach`アクションを実行する。このアクションは問題分析結果（`context.analysisResult`）に基づき、人間が選択したアプローチをコンテキストに反映する。

```typescript
approachSelection: {
  entry: [{ type: 'selectApproach' }],
  always: [
    { guard: 'isApproachA', target: 'directResolution' },
    { guard: 'isApproachB', target: 'redecompose' },
    { guard: 'isApproachC', target: 'resetContext' },
    { target: 'escalationJudgment' },  // フォールバック: D
  ],
},
```

### 3.10 T6外部IFの拡張（SP3Output → SP3OutputExtended）

T7のSP-3 outputを拡張し、エラー情報をRFに引き渡す。

```typescript
// T6 §5.1 の既存定義
type SP3Output = {
  passed: boolean;
  lossCut: boolean;
};

// T8で拡張
type SP3OutputExtended = SP3Output & {
  /** 直近のエラー内容（RF入力用） */
  lastError: ErrorInfo | null;
  /** エラー修正履歴（RF入力用） */
  errorHistory: ErrorRecord[];
};
```

メインフローのverificationLoop状態のonDone処理で、この拡張outputをメインフローのコンテキストに格納し、RF起動時にRFのコンテキストに注入する。

```typescript
// メインフロー側のonDone処理（概念）
verificationLoop: {
  onDone: [
    {
      guard: 'isVerificationPassed',
      target: 'taskComplete',
    },
    {
      // lossCut: true の場合
      actions: assign({
        lastErrorForRecovery: ({ event }) => event.output.lastError,
        errorHistoryForRecovery: ({ event }) => event.output.errorHistory,
      }),
      target: 'recoveryFlow',
    },
  ],
},
```

---

## 4. ES（エスカレーション判断）の詳細仕様

### 4.1 設計方針

ESはRF内のcompound state（`escalationJudgment`）として定義する（設計判断#1）。ESの内部構造はLC（損切り判断）と同様のパターンで、ゲートウェイを判定状態に変換し、`always`遷移でガード評価を行う。

ES-GW1（即座判定）→ ES-GW2（30分以内判定）の2段階評価は、LC-GW1〜GW4の状態チェーンパターンの簡易版である。

### 4.2 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `checkImmediate` | 判定状態（initial） | — | ES-GW1 |
| `executeImmediate` | 通常状態 | `executeImmediateEscalation` | ES-T1 |
| `check30Min` | 判定状態 | — | ES-GW2 |
| `consider30Min` | 通常状態 | `considerEscalation` | ES-T2 |
| `escalationConfirmed` | 最終状態 | — | ES-EE-ESC |
| `selfResolution` | 最終状態 | — | ES-EE-SELF |

**適用マッピングルール**: MR-1（ES-SE → `initial: 'checkImmediate'`）、MR-2（ES-EE-ESC/SELF → final）、MR-3（ES-T1/T2 → 状態 + entry）、MR-5逐次GWパターン（ES-GW1/GW2 → 状態チェーン + `always`遷移）

### 4.3 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `ESCALATION_DECIDED` | エスカレーションの実施/検討アクションが完了した | `executeImmediateEscalation`/`considerEscalation`アクション |

**注記**: `checkImmediate`と`check30Min`は`always`（即時遷移）を使用するため、イベントは不要。

### 4.4 ガード条件定義

| ガード名 | 条件 | 対応GW | 保証する不変条件 |
|---------|------|--------|----------------|
| `isSecurityOrProductionOrDataLoss` | `context.analysisResult.hasSecurityIssue \|\| context.analysisResult.hasProductionImpact \|\| context.analysisResult.hasDataLossRisk` | ES-GW1 | INV-ES1, INV-ES2 |
| `isRetreat3TimesOrUnknownOrOutOfScope` | `context.analysisResult.retreatCount >= 3 \|\| context.analysisResult.isUnknownCause \|\| context.analysisResult.isOutOfSkillScope` | ES-GW2 | INV-ES1 |

### 4.5 アクション定義

| アクション名 | 種別 | 処理内容 |
|-------------|------|---------|
| `executeImmediateEscalation` | entry | セキュリティ/本番影響/データ損失に対する即座のエスカレーションを実施する |
| `considerEscalation` | entry | 3回撤退/原因不明/スキル範囲外に対する30分以内のエスカレーション検討を実施する |

### 4.6 遷移表

| 現在の状態 | イベント/条件 | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|-------------|--------|-----------|--------|------------|
| `checkImmediate` | always | `isSecurityOrProductionOrDataLoss` | — | `executeImmediate` | INV-ES1, INV-ES2 |
| `checkImmediate` | always | （フォールバック） | — | `check30Min` | INV-ES1 |
| `executeImmediate` | `ESCALATION_DECIDED` | — | — | `escalationConfirmed` | INV-ES2, INV-ES3 |
| `check30Min` | always | `isRetreat3TimesOrUnknownOrOutOfScope` | — | `consider30Min` | INV-ES1 |
| `check30Min` | always | （フォールバック） | — | `selfResolution` | INV-ES3 |
| `consider30Min` | `ESCALATION_DECIDED` | — | — | `escalationConfirmed` | INV-ES3 |

**2段階評価の構造的保証**: `checkImmediate`（ES-GW1）がinitial状態であるため、ESに入ると必ず即座判定が最初に実行される。`check30Min`（ES-GW2）は`checkImmediate`のフォールバックからのみ到達可能であり、GW1→GW2の順序がINV-ES1を構造的に保証する。

### 4.7 ESのoutput定義

```typescript
output: ({ self }) => {
  const snapshot = self.getSnapshot();
  return {
    result: snapshot.matches('escalationJudgment.escalationConfirmed')
      ? 'escalate'
      : 'self',
  };
},
```

RF側の`escalationJudgment`のonDoneで、このoutputを参照して`consultTeam`（escalate）または`approachSelection`（self）に分岐する。

---

## 5. データオブジェクトとの対応

| データID | 名称 | T8での操作 | 対応アクション |
|---------|------|-----------|--------------|
| **D-1** | CLAUDE.md | RF-T9で更新（失敗パターン追記） | `recordFailurePattern` |
| **D-5** | 失敗パターン記録 | RF-T10で更新（回避策追記） | `documentWorkaround` |
| **D-6** | エラー状態記録 | SP-3から引き継ぎ（RF入力） | メインフローコンテキスト経由 |
| **D-7** | 問題分析結果 | RF-T1〜T3で作成 | `identifyEssence`で`context.analysisResult`に格納 |
| **D-8** | チーム共有記録 | RF-T11で作成 | `shareWithTeam` |

---

## 6. フロー間接続の設計

### 6.1 RFへの入口（メインフロー → RF）

```typescript
// メインフローのverificationLoop onDone（lossCut時）
{
  actions: assign({
    lastErrorForRecovery: ({ event }) => event.output.lastError,
    errorHistoryForRecovery: ({ event }) => event.output.errorHistory,
  }),
  target: 'recoveryFlow',
}
```

RFの初期コンテキストは、メインフローのコンテキストから注入する。

```typescript
recoveryFlow: {
  entry: assign({
    lastError: ({ context }) => context.lastErrorForRecovery,
    errorHistory: ({ context }) => context.errorHistoryForRecovery,
  }),
  initial: 'problemAnalysis',
  // ...
}
```

### 6.2 RFからの出口（RF → メインフロー SE-1）

```typescript
// メインフローのrecoveryFlow onDone
recoveryFlow: {
  onDone: {
    target: 'brightLinesCheck',  // SE-1相当（メインフロー開始状態）
  },
}
```

INV-RF4: `recoveryComplete`（final）への遷移元は`teamShareDecision`（共有しない場合）と`shareWithTeam`（共有した場合）のみ。`recoveryComplete`からの唯一の接続先はメインフローのSE-1であり、他のサブプロセスへの直接遷移パスは存在しない。

INV-CF3: RF内にLCへの遷移パスは定義されていない（一方向性保証）。
INV-CF4: RF-EEからの唯一の遷移先がSE-1（`brightLinesCheck`）に限定されている。

### 6.3 ES呼び出しパス

ESは以下の2箇所からRF内の同一`escalationJudgment`状態に遷移する。

| 呼び出し元 | 条件 | 遷移 |
|-----------|------|------|
| `escalationCheck`（RF-GW1） | `needsImmediateEscalation` = true | → `escalationJudgment` |
| `approachSelection`（RF-GW2） | フォールバック（D選択） | → `escalationJudgment` |

ESの結果による後続遷移は§3.7遷移表のとおり。

---

## 7. 不変条件の保証マッピング

### 7.1 INV-RF: 復帰フロー不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-RF1** | 問題の分析（RF-T1→T2→T3）は復帰フローの最初のステップとして必ず実行される（5-10分） | §2.1: `problemAnalysis`がinitial状態。§3.7: `verbalizeProblem`→`analyzeCause`→`identifyEssence`の一方向遷移 | `initial: 'problemAnalysis'`によりRF開始時に必ずStep 1から始まる。`problemAnalysis`はcompound stateであり、内部の3状態は逐次遷移のみが定義されている。Step 1を経由せずにStep 2以降に進む経路は存在しない |
| **INV-RF2** | CLAUDE.mdへの失敗パターン記録（RF-T9）は必須ステップであり、いかなるアプローチを選択しても省略できない | §3.7: 5つの合流パス（A: `askAiExplanation`、B: `redecompose`、C: `resetContext`、D: `consultTeam`、GW1直行: `consultTeam`）がすべて`recordToClaudeMd`に直結 | `recordToClaudeMd`へ至らない経路は定義されていない。各アプローチの完了イベント処理の遷移先がすべて`recordToClaudeMd`であり、他の状態への分岐やスキップパスは存在しない |
| **INV-RF3** | 次回の回避策の明記（RF-T10）はRF-T9の後に必ず実行される | §3.7: `recordToClaudeMd`の`CLAUDE_MD_RECORDED`イベントの遷移先が`documentWorkaround`のみ | `recordToClaudeMd`から`documentWorkaround`への直接遷移が唯一の経路であり、RF-T9をスキップしてRF-T10に到達するパスは存在しない。逆にRF-T10をスキップするパスも存在しない |
| **INV-RF4** | 復帰フローは常にメインフローSE-1に戻る | §2.1: `recoveryComplete`が唯一のfinal状態。§3.8: output `{ recovered: true }`。§6.2: onDoneで`brightLinesCheck`（SE-1）に遷移 | `recoveryComplete`以外にfinal状態は存在しない。メインフローのrecoveryFlow onDoneの遷移先は`brightLinesCheck`（SE-1相当）のみであり、他のサブプロセスへの直接遷移パスは定義されていない |
| **INV-RF5** | アプローチ選択（RF-GW2）は問題の分析（Step 1）完了後にのみ実行される | §3.7: `identifyEssence`（Step 1最終）→ `escalationCheck` → `approachSelection`の逐次遷移。`approachSelection`への遷移元は`escalationCheck`のフォールバックと`escalationJudgment`のonDone（SELF）のみ | `approachSelection`に到達するには、必ず`escalationCheck`を経由する。`escalationCheck`に到達するには`identifyEssence`（Step 1完了）を通過する必要がある。Step 1を経由せずに`approachSelection`に到達する経路は存在しない |
| **INV-RF6** | アプローチは4パターン（A/B/C/D）のいずれか1つが選択される | §3.7: `approachSelection`の`always`遷移が4つのガード付き分岐（`isApproachA`/B/C + フォールバックD）を定義 | `always`遷移のガード配列は上から順に評価され、最初にtrueとなったガードの遷移先のみが実行される（排他的選択）。フォールバック（D）により必ず1つのパスが選択される |

### 7.2 INV-ES: エスカレーション判断不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-ES1** | 「即座にエスカレーション必要」の判定（ES-GW1）は「30分以内」の判定（ES-GW2）に先行する | §4.2: `checkImmediate`がinitial状態。§4.6: `checkImmediate`のフォールバックが`check30Min`に遷移 | `initial: 'checkImmediate'`によりESに入ると必ずGW1判定が最初に実行される。`check30Min`は`checkImmediate`のフォールバックからのみ到達可能であり、GW2がGW1に先行する経路は存在しない |
| **INV-ES2** | セキュリティ問題・本番影響・データ損失リスクのいずれかに該当する場合、即座にエスカレーションが実施される（遅延不可） | §4.4: `isSecurityOrProductionOrDataLoss`ガードが3条件のOR評価。§4.6: `checkImmediate`→`executeImmediate`→`escalationConfirmed`の直結パス | `isSecurityOrProductionOrDataLoss`がtrueの場合、`always`遷移で即座に`executeImmediate`に遷移し、`ESCALATION_DECIDED`イベントで`escalationConfirmed`に直結する。30分以内検討（`check30Min`）を経由する経路は存在しない |
| **INV-ES3** | エスカレーション判断の結果は「エスカレーション実施（ES-EE-ESC）」または「自力で対応（ES-EE-SELF）」の2つのみ | §4.2: final状態が`escalationConfirmed`と`selfResolution`の2つのみ。§4.7: outputは`'escalate'`または`'self'`の2値 | ES内のfinal状態は2つのみであり、他の終了パスは定義されていない。`escalationConfirmed`への遷移元は`executeImmediate`と`consider30Min`、`selfResolution`への遷移元は`check30Min`のフォールバックのみ |

### 7.3 関連する不変条件（INV-CF）の対応

| ID | 不変条件 | T8での対応 |
|----|---------|-----------|
| INV-CF3 | LC→RFへの遷移は一方向（RF→LCへの逆遷移なし） | RF内にLCへの遷移パスを定義していない（§3.7遷移表に`lossCutJudgment`への遷移なし） |
| INV-CF4 | RF完了後はSE-1に戻る（他のSPに直接遷移しない） | §6.2: onDoneの遷移先を`brightLinesCheck`（SE-1相当）に限定 |

---

## 8. XState v5 擬似コード

### 8.1 RF（recoveryFlow）

```typescript
import { setup, assign } from 'xstate';

const recoveryFlowMachine = setup({
  types: {
    context: {} as RecoveryFlowContext,
    events: {} as
      | { type: 'PROBLEM_VERBALIZED' }
      | { type: 'CAUSE_ANALYZED' }
      | { type: 'ESSENCE_IDENTIFIED'; analysisResult: ProblemAnalysis }
      | { type: 'ESCALATION_DECIDED' }
      | { type: 'HUMAN_FIX_COMPLETE' }
      | { type: 'AI_EXPLANATION_RECEIVED' }
      | { type: 'REDECOMPOSE_COMPLETE' }
      | { type: 'CONTEXT_RESET_COMPLETE' }
      | { type: 'TEAM_CONSULTED' }
      | { type: 'CLAUDE_MD_RECORDED' }
      | { type: 'WORKAROUND_DOCUMENTED' }
      | { type: 'TEAM_SHARED' },
  },
  guards: {
    needsImmediateEscalation: ({ context }) => {
      const a = context.analysisResult;
      return a !== null && (
        a.hasSecurityIssue ||
        a.hasProductionImpact ||
        a.hasDataLossRisk
      );
    },
    isApproachA: ({ context }) => context.selectedApproach === 'A',
    isApproachB: ({ context }) => context.selectedApproach === 'B',
    isApproachC: ({ context }) => context.selectedApproach === 'C',
    // isApproachD はフォールバックのため明示的ガード不要
    shouldShareWithTeam: ({ context }) => context.shouldShareWithTeam === true,
    isEscalationConfirmed: (_, params) =>
      params.output.result === 'escalate',
    // ES内部のガード
    isSecurityOrProductionOrDataLoss: ({ context }) => {
      const a = context.analysisResult;
      return a !== null && (
        a.hasSecurityIssue ||
        a.hasProductionImpact ||
        a.hasDataLossRisk
      );
    },
    isRetreat3TimesOrUnknownOrOutOfScope: ({ context }) => {
      const a = context.analysisResult;
      return a !== null && (
        a.retreatCount >= 3 ||
        a.isUnknownCause ||
        a.isOutOfSkillScope
      );
    },
  },
  actions: {
    verbalizeProblem: () => { /* 問題言語化ロジック */ },
    analyzeCause: () => { /* 原因分析ロジック */ },
    identifyEssence: () => { /* 本質特定ロジック */ },
    selectApproach: () => { /* アプローチ選択ロジック */ },
    executeImmediateEscalation: () => { /* 即座エスカレーション */ },
    considerEscalation: () => { /* 30分以内エスカレーション検討 */ },
    humanDirectFix: () => { /* 人間直接解決 */ },
    askAiExplanation: () => { /* AI説明取得 */ },
    redecomposeProblem: () => { /* 問題再分解 */ },
    resetContext: () => { /* コンテキストリセット */ },
    consultTeam: () => { /* チーム相談 */ },
    recordFailurePattern: () => { /* CLAUDE.md記録 */ },
    documentWorkaround: () => { /* 回避策文書化 */ },
    shareWithTeam: () => { /* チーム共有 */ },
    setEscalationResult: assign({
      escalationResult: (_, params) =>
        params.output.result === 'escalate' ? 'escalate' : 'self',
    }),
  },
}).createMachine({
  id: 'recoveryFlow',
  initial: 'problemAnalysis',
  context: {
    lastError: null,
    errorHistory: [],
    analysisResult: null,
    selectedApproach: null,
    escalationResult: null,
    failureRecord: null,
    shouldShareWithTeam: false,
  },

  states: {
    // ========================================
    // Step 1: 問題分析（compound state）
    // ========================================
    problemAnalysis: {
      initial: 'verbalizeProblem',
      states: {
        // RF-T1: 問題を言語化する（INV-RF1: 最初のステップ）
        verbalizeProblem: {
          entry: [{ type: 'verbalizeProblem' }],
          on: {
            PROBLEM_VERBALIZED: 'analyzeCause',       // INV-RF1
          },
        },

        // RF-T2: 原因を分析する
        analyzeCause: {
          entry: [{ type: 'analyzeCause' }],
          on: {
            CAUSE_ANALYZED: 'identifyEssence',         // INV-RF1
          },
        },

        // RF-T3: 本質を特定する
        identifyEssence: {
          entry: [{ type: 'identifyEssence' }],
          on: {
            ESSENCE_IDENTIFIED: {
              actions: assign({
                analysisResult: ({ event }) => event.analysisResult,
              }),
              target: '#recoveryFlow.escalationCheck',  // INV-RF5
            },
          },
        },
      },
    },

    // ========================================
    // RF-GW1: エスカレーション要否判定
    // ========================================
    escalationCheck: {
      always: [
        {
          guard: 'needsImmediateEscalation',
          target: 'escalationJudgment',                // INV-RF5
        },
        {
          target: 'approachSelection',                 // INV-RF5
        },
      ],
    },

    // ========================================
    // ES: エスカレーション判断（compound state）
    // ========================================
    escalationJudgment: {
      initial: 'checkImmediate',
      states: {
        // ES-GW1: 即座にエスカレーション必要？（INV-ES1: 最初の判定）
        checkImmediate: {
          always: [
            {
              guard: 'isSecurityOrProductionOrDataLoss',
              target: 'executeImmediate',              // INV-ES2
            },
            {
              target: 'check30Min',                    // INV-ES1
            },
          ],
        },

        // ES-T1: 即座にエスカレーションを実施（INV-ES2: 遅延不可）
        executeImmediate: {
          entry: [{ type: 'executeImmediateEscalation' }],
          on: {
            ESCALATION_DECIDED: 'escalationConfirmed', // INV-ES3
          },
        },

        // ES-GW2: 30分以内にエスカレーション検討すべき？
        check30Min: {
          always: [
            {
              guard: 'isRetreat3TimesOrUnknownOrOutOfScope',
              target: 'consider30Min',
            },
            {
              target: 'selfResolution',                // INV-ES3
            },
          ],
        },

        // ES-T2: 30分以内にエスカレーションを検討する
        consider30Min: {
          entry: [{ type: 'considerEscalation' }],
          on: {
            ESCALATION_DECIDED: 'escalationConfirmed', // INV-ES3
          },
        },

        // ES-EE-ESC: エスカレーション実施
        escalationConfirmed: { type: 'final' },

        // ES-EE-SELF: 自力で対応
        selfResolution: { type: 'final' },
      },

      // ESのoutput
      output: ({ self }) => ({
        result: self.getSnapshot().matches(
          'escalationJudgment.escalationConfirmed'
        )
          ? 'escalate' as const
          : 'self' as const,
      }),

      // ES完了時の分岐
      onDone: [
        {
          guard: 'isEscalationConfirmed',
          actions: [{ type: 'setEscalationResult' }],
          target: 'consultTeam',                       // ESC → チーム相談
        },
        {
          actions: [{ type: 'setEscalationResult' }],
          target: 'approachSelection',                 // SELF → アプローチ再選択
        },
      ],
    },

    // ========================================
    // RF-GW2: アプローチ選択（INV-RF6: 4パターン排他）
    // ========================================
    approachSelection: {
      entry: [{ type: 'selectApproach' }],
      always: [
        {
          guard: 'isApproachA',
          target: 'directResolution',                  // INV-RF6: A
        },
        {
          guard: 'isApproachB',
          target: 'redecompose',                       // INV-RF6: B
        },
        {
          guard: 'isApproachC',
          target: 'resetContext',                       // INV-RF6: C
        },
        {
          target: 'escalationJudgment',                // INV-RF6: D（フォールバック）
        },
      ],
    },

    // ========================================
    // Step 2: アプローチ実行
    // ========================================

    // --- A: 直接解決（RF-T4 + RF-T5） ---
    directResolution: {
      initial: 'humanDirectFix',
      states: {
        humanDirectFix: {
          entry: [{ type: 'humanDirectFix' }],
          on: {
            HUMAN_FIX_COMPLETE: 'askAiExplanation',
          },
        },
        askAiExplanation: {
          entry: [{ type: 'askAiExplanation' }],
          on: {
            AI_EXPLANATION_RECEIVED: {
              target: '#recoveryFlow.recordToClaudeMd', // INV-RF2: 合流
            },
          },
        },
      },
    },

    // --- B: 再分解（RF-T6） ---
    redecompose: {
      entry: [{ type: 'redecomposeProblem' }],
      on: {
        REDECOMPOSE_COMPLETE: 'recordToClaudeMd',      // INV-RF2: 合流
      },
    },

    // --- C: リセット（RF-T7） ---
    resetContext: {
      entry: [{ type: 'resetContext' }],
      on: {
        CONTEXT_RESET_COMPLETE: 'recordToClaudeMd',    // INV-RF2: 合流
      },
    },

    // --- D後のチーム相談（RF-T8） ---
    consultTeam: {
      entry: [{ type: 'consultTeam' }],
      on: {
        TEAM_CONSULTED: 'recordToClaudeMd',            // INV-RF2: 合流
      },
    },

    // ========================================
    // Step 3: 学習記録（必須）
    // ========================================

    // --- RF-T9: CLAUDE.mdに追記する（INV-RF2: 全パスの必須合流点） ---
    recordToClaudeMd: {
      entry: [{ type: 'recordFailurePattern' }],
      on: {
        CLAUDE_MD_RECORDED: 'documentWorkaround',      // INV-RF3
      },
    },

    // --- RF-T10: 回避策を文書化する（INV-RF3: RF-T9の後に必ず実行） ---
    documentWorkaround: {
      entry: [{ type: 'documentWorkaround' }],
      on: {
        WORKAROUND_DOCUMENTED: 'teamShareDecision',
      },
    },

    // --- RF-GW3: チーム共有判断 ---
    teamShareDecision: {
      always: [
        {
          guard: 'shouldShareWithTeam',
          target: 'shareWithTeam',
        },
        {
          target: 'recoveryComplete',                  // INV-RF4
        },
      ],
    },

    // --- RF-T11: チームに共有する ---
    shareWithTeam: {
      entry: [{ type: 'shareWithTeam' }],
      on: {
        TEAM_SHARED: 'recoveryComplete',               // INV-RF4
      },
    },

    // ========================================
    // RF-EE: 復帰完了（INV-RF4: SE-1に復帰）
    // ========================================
    recoveryComplete: { type: 'final' },
  },

  // RFのoutput（メインフローとの接続）
  output: () => ({
    recovered: true,
  }),
});
```

---

## 9. 出典

| ファイル | 参照セクション |
|---------|--------------|
| `phase1-t5-losscutrecovery-bpmn.md` | §2: RF（17要素）、§3: ES（7要素）、§4: データオブジェクト、§5: フロー間接続 |
| `phase1-t6-losscut-recovery-mermaid.md` | §2: RF Mermaid図、§3: ES Mermaid図、§4: 統合フロー図 |
| `phase1-t7-invariants.md` | §7: INV-RF（6条件）、§8: INV-ES（3条件）、§12: INV-CF（5条件） |
| `phase1-t3-bpmn-elements.md` | §5: 復帰フロー概要（T5で詳細化済み） |
| `docs/bpmn-to-statechart-mapping.md` | MR-1〜MR-9マッピングルール |
| `docs/spec-l0l4-hierarchy.md` | T5: output定義方式 |
| `docs/spec-mainflow-sp2.md` | T6: §5.1 SP3Outputインターフェース |
| `docs/spec-verification-losscut.md` | T7: SP-3コンテキスト定義、output定義、申し送り |

---

## 10. T9以降への申し送り

| # | 項目 | 内容 |
|---|------|------|
| 1 | T6外部IFの拡張 | SP3OutputをSP3OutputExtendedに拡張（§3.10）。T6の`docs/spec-mainflow-sp2.md` §5.1を更新する必要がある。メインフロー側の`verificationLoop` onDone処理にlastError/errorHistoryの受け渡しアクションを追加すること |
| 2 | DT-9のA4違反時のメインフロー復帰 | T7 §6.3の申し送り（A4違反→DT-0に戻る経路）はRFのスコープ外。メインフロー側でSP-3のoutputにA4違反フラグを追加し、GW-1に戻る経路を定義すること。Step C（XState実装）のT14（統合）で対応を推奨 |
| 3 | `needsImmediateEscalation`と`isSecurityOrProductionOrDataLoss`の重複 | RF-GW1のガード（`needsImmediateEscalation`）とES-GW1のガード（`isSecurityOrProductionOrDataLoss`）は同一条件である。実装時に共通化またはRF-GW1の省略を検討すること。ただし、RF-GW1はES起動前のファストパスとして設計上の意味がある（ESに入らずに直接ESに遷移する経路を明示） |
| 4 | `selectApproach`の実装 | `approachSelection`状態のentry時に実行する`selectApproach`アクションは、人間の判断入力を受け取るインタラクティブなアクションである。XState実装時に、人間の入力待ち（外部イベントによるアプローチ指定）とalways遷移の組み合わせ方を検討すること。`always`遷移はentry完了後に評価されるため、entryで`selectedApproach`が設定済みであることが前提となる |
| 5 | 命名規則の継続 | イベント: SCREAMING_SNAKE、状態: camelCase、ガード: camelCase（T5→T6→T7→T8と同一） |
| 6 | ES/RFのoutput方式 | T5/T6/T7と同様に`output`プロパティを使用。Step C以降でも同方式を踏襲すること |
