# T7: 検証ループ＋損切りのステートチャート仕様

> **タスク**: SP-3（typecheck→lint→test＋協働原則監視）と損切り判断（LC: 4条件OR評価＋短絡評価）のステートチャートを定義する
> **検証方法**: INV-SP3-1〜SP3-5、INV-LC1〜LC5の10個の不変条件が設計上すべて保証されていること
> **自信度**: おそらく（並行監視の設計解釈と損切りのネスト構造に設計判断を含む）
> **入力**: `phase1-t3-bpmn-elements.md`（§4: SP-3フロー要素）、`phase1-t5-losscutrecovery-bpmn.md`（§1: LC）、`phase1-t7-invariants.md`（INV-SP3, INV-LC）、`docs/bpmn-to-statechart-mapping.md`（MR-1〜MR-9）、`docs/spec-l0l4-hierarchy.md`（T5成果物）、`docs/spec-mainflow-sp2.md`（T6成果物）

---

## 1. スコープと設計判断

### 1.1 T7のスコープ

| 対象 | 詳細度 | 理由 |
|------|--------|------|
| SP-3（検証フィードバックループ） | **内部詳細定義** | INV-SP3-1〜SP3-5の保証にSP-3内部構造が必要 |
| LC（損切り判断サブプロセス） | **内部詳細定義** | INV-LC1〜LC5の保証にLC内部構造が必要 |
| メインフローとの接続 | **T6外部IFに準拠** | T6 §5.1で定義済みの`SP3Output`に準拠 |

### 1.2 T6からの申し送り対応

| # | 申し送り | T7での対応 |
|---|---------|-----------|
| 1 | SP-3のoutputは`{ passed: boolean; lossCut: boolean }` | §3.2 SP3Output型で準拠 |
| 2 | `humanExecution`と`humanReview`の2箇所から合流 | §2.2 両パスとも同一の`initial: 'typecheck'`に遷移 |
| 3 | onDoneで`isVerificationPassed`ガードを評価 | §3.8 outputで`passed`フラグを返す |
| 4 | 損切り時は`lossCut: true`を設定 | §3.8 outputで`lossCut`フラグを返す |
| 5 | 命名規則の継続 | イベント: SCREAMING_SNAKE、状態: camelCase、ガード: camelCase |

### 1.3 設計判断の確定

#### 設計判断#1: LC-GW1〜GW4の変換方式

| 項目 | 内容 |
|------|------|
| **選択** | **A: 状態チェーン** |
| **理由** | INV-LC4（短絡評価）がMR-5の「逐次ゲートウェイ」パターンで構造的に保証される。各GWを独立状態にすることで、Yesの場合は即座に`lossCutConfirmed`に遷移し、Noの場合のみ次の条件に進む。この構造は短絡評価の意味を直接表現する |
| **不採用理由（B: 複合ガード）** | `or`ガードで4条件をまとめると、短絡評価の順序と可視性が失われる。INV-LC4の「いずれかでYesとなった時点で残りは評価されない」が構造的に保証されない |

#### 設計判断#2: SP-3の並行リージョン間通信

| 項目 | 内容 |
|------|------|
| **選択** | **C: entry/exitアクション** |
| **理由** | DT-8（協働原則）とDT-9（AI行動原則）は評価チェックであり、長時間稼働するバックグラウンド監視ではない。各検証ステップのentry時に原則チェックを実行することで「検証プロセスの各段階で常にチェックされる」を実現する。これはINV-SP3-3の実用的解釈として十分である |
| **不採用理由（A: parallel state）** | 2リージョン間の同期（片方がfinalに達した時の他方への通知）にXState v5での追加設計が必要。テスタビリティが低下し、SP-3の中核である検証チェーン＋損切り判断の構造が不明瞭になる |
| **不採用理由（B: invokeサービス）** | 非同期サービスとしての監視は実態に合わない。原則チェックは各検証ステップと同期的に実行される評価であり、独立サービスとして分離する必要性がない |
| **INV-SP3-3への対応** | 「並行して常に監視」= 「各検証ステップ（typecheck/lint/test）のentryで毎回DT-8/DT-9を評価」と解釈。SP-3がアクティブな間、すべての検証ステップで原則チェックが実行されるため、実質的な常時監視が保証される |
| **マッピングとの差異** | `bpmn-to-statechart-mapping.md` §2.4ではSP-3を`parallel state`として仮定義していたが、T7での詳細設計の結果、compound stateに変更。マッピングの仮定義は「SP-3に並行的な側面がある」ことを示したものであり、最終的な実装構造はT7で確定する |

---

## 2. SP-3（検証フィードバックループ）の全体構造

### 2.1 状態階層図

```
verificationLoop（SP-3: compound state）
│
├── typecheck（initial）     ← SP3-T1: INV-SP3-1
│   entry: [runTypecheck, checkCollaborationPrinciples, checkAIPrinciples]
│
├── lint                      ← SP3-T2: INV-SP3-2
│   entry: [runLint, checkCollaborationPrinciples, checkAIPrinciples]
│
├── test                      ← SP3-T3: INV-SP3-2
│   entry: [runTest, checkCollaborationPrinciples, checkAIPrinciples]
│
├── lossCutJudgment（LC: compound state）← SP3-GW4 → LC詳細
│   ├── recordErrorState（initial） ← LC-T1: INV-LC1
│   ├── check3Times               ← LC-GW1: INV-LC4
│   ├── check30Min                ← LC-GW2: INV-LC4
│   ├── checkComplexity           ← LC-GW3: INV-LC4
│   ├── checkRecurrence           ← LC-GW4: INV-LC4
│   ├── continueFix（final）      ← LC-EE-CONT: INV-LC3
│   └── lossCutConfirmed（final） ← LC-EE-CUT: INV-LC2
│
├── issueFix                  ← LC-T2（LCのonDone continueFixから遷移）
│
├── verificationPassed（final）← SP3-EE-OK: INV-SP3-5
└── verificationFailed（final）← SP3-EE-NG
```

### 2.2 状態遷移の全体フロー

```
[humanExecution or humanReview から遷移]
    │
    ▼
typecheck（initial）
    │
    ├── CHECK_COMPLETE [pass] → lint                    ← INV-SP3-1, INV-SP3-2
    │
    └── CHECK_COMPLETE [fail] → lossCutJudgment         ← INV-SP3-4
                                    │
lint                                │
    │                               │
    ├── CHECK_COMPLETE [pass] → test                    ← INV-SP3-2
    │                               │
    └── CHECK_COMPLETE [fail] → lossCutJudgment         ← INV-SP3-4
                                    │
test                                │
    │                               │
    ├── CHECK_COMPLETE [pass] → verificationPassed      ← INV-SP3-5
    │                               │
    └── CHECK_COMPLETE [fail] → lossCutJudgment         ← INV-SP3-4
                                    │
                        lossCutJudgment（LC）
                            │
                            ├── onDone [continueFix] → issueFix → typecheck（ループ）
                            │                                      ← INV-LC3
                            └── onDone [lossCutConfirmed] → verificationFailed
                                                            ← INV-LC2

[30分タイマー] ──→ lossCutJudgment（どの検証状態からでも強制遷移） ← INV-LC5
```

---

## 3. SP-3の詳細仕様

### 3.1 マシン定義

**マシンID**: `verificationLoopMachine`
**種類**: compound state（メインフローの`verificationLoop`としてネスト）

### 3.2 コンテキスト定義

```typescript
type VerificationLoopContext = {
  /** 検証対象ステップの現在位置（typecheck / lint / test） */
  currentStep: 'typecheck' | 'lint' | 'test';
  /** 同一エラーへの往復回数 */
  errorCount: number;
  /** SP-3開始時刻（30分タイマー計算用） */
  startedAt: number;
  /** 直近のエラー内容（損切り判断の入力） */
  lastError: ErrorInfo | null;
  /** エラー修正履歴（複雑化・再発判定の入力） */
  errorHistory: ErrorRecord[];
  /** 協働原則チェック結果（DT-8） */
  collaborationCheckResult: PrincipleCheckResult | null;
  /** AI行動原則チェック結果（DT-9） */
  aiPrincipleCheckResult: PrincipleCheckResult | null;
};

type ErrorInfo = {
  step: 'typecheck' | 'lint' | 'test';
  message: string;
  timestamp: number;
};

type ErrorRecord = {
  error: ErrorInfo;
  fixAttempt: string;
  /** 修正後のコード複雑度（定性値） */
  complexityDelta: 'increased' | 'unchanged' | 'decreased';
};

type PrincipleCheckResult = {
  passed: boolean;
  violations: string[];
};
```

**設計根拠**: `errorCount`はINV-LC5（3回ルール）の判定に使用。`startedAt`はINV-LC5（30分ルール）のために`after`遅延遷移と併用。`errorHistory`はINV-LC2の複雑化・再発条件の評価に使用。

### 3.3 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `typecheck` | 通常状態（initial） | `runTypecheck`, `checkCollaborationPrinciples`, `checkAIPrinciples` | SP3-T1, SP3-T5, SP3-T6 |
| `lint` | 通常状態 | `runLint`, `checkCollaborationPrinciples`, `checkAIPrinciples` | SP3-T2, SP3-T5, SP3-T6 |
| `test` | 通常状態 | `runTest`, `checkCollaborationPrinciples`, `checkAIPrinciples` | SP3-T3, SP3-T5, SP3-T6 |
| `lossCutJudgment` | compound state | — | SP3-GW4 → LC（§4で詳細定義） |
| `issueFix` | 通常状態 | `issueFixInstruction` | LC-T2 |
| `verificationPassed` | 最終状態 | — | SP3-EE-OK |
| `verificationFailed` | 最終状態 | — | SP3-EE-NG |

**適用マッピングルール**: MR-1（SP3-SE → `initial: 'typecheck'`）、MR-2（SP3-EE-OK/NG → final）、MR-3（SP3-T1〜T6 → 状態 + entry）、MR-4（LC → ネスト状態）

### 3.4 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `TYPECHECK_COMPLETE` | typecheckの実行が完了した | `runTypecheck`アクション |
| `LINT_COMPLETE` | lintの実行が完了した | `runLint`アクション |
| `TEST_COMPLETE` | testの実行が完了した | `runTest`アクション |
| `FIX_ISSUED` | エラー修正指示が完了した | `issueFixInstruction`アクション |

**注記**: 3つの検証完了イベントは個別のイベント名とする。これにより、各検証ステップからのガード条件が明確に分離され、状態ごとに異なるイベントを処理する設計となる。

### 3.5 ガード条件定義

| ガード名 | 条件 | 対応GW | 保証する不変条件 |
|---------|------|--------|----------------|
| `isTypecheckPass` | `event.result.passed === true` | SP3-GW1 | INV-SP3-2 |
| `isLintPass` | `event.result.passed === true` | SP3-GW2 | INV-SP3-2 |
| `isTestPass` | `event.result.passed === true` | SP3-GW3 | INV-SP3-5 |
| `isLossCutContinue` | `event.output.decision === 'continue'` | — | INV-LC3 |

### 3.6 アクション定義

| アクション名 | 種別 | 処理内容 |
|-------------|------|---------|
| `runTypecheck` | entry | typecheckを実行し、結果イベントを発火 |
| `runLint` | entry | lintを実行し、結果イベントを発火 |
| `runTest` | entry | testを実行し、結果イベントを発火 |
| `checkCollaborationPrinciples` | entry | DT-8（C1-C4）を評価し、結果を`context.collaborationCheckResult`に格納 |
| `checkAIPrinciples` | entry | DT-9（A1-A4）を評価し、結果を`context.aiPrincipleCheckResult`に格納 |
| `issueFixInstruction` | entry | エラー内容に基づく修正指示を生成 |
| `incrementErrorCount` | assign | `context.errorCount`を+1する |
| `recordError` | assign | エラー情報を`context.lastError`に、履歴を`context.errorHistory`に追加 |
| `resetCurrentStep` | assign | `context.currentStep`を`'typecheck'`にリセット（再検証ループ用） |

### 3.7 遷移表

| 現在の状態 | イベント/条件 | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|-------------|--------|-----------|--------|------------|
| `typecheck` | `TYPECHECK_COMPLETE` | `isTypecheckPass` | — | `lint` | INV-SP3-1, INV-SP3-2 |
| `typecheck` | `TYPECHECK_COMPLETE` | （フォールバック） | `incrementErrorCount`, `recordError` | `lossCutJudgment` | INV-SP3-4 |
| `lint` | `LINT_COMPLETE` | `isLintPass` | — | `test` | INV-SP3-2 |
| `lint` | `LINT_COMPLETE` | （フォールバック） | `incrementErrorCount`, `recordError` | `lossCutJudgment` | INV-SP3-4 |
| `test` | `TEST_COMPLETE` | `isTestPass` | — | `verificationPassed` | INV-SP3-5 |
| `test` | `TEST_COMPLETE` | （フォールバック） | `incrementErrorCount`, `recordError` | `lossCutJudgment` | INV-SP3-4 |
| `lossCutJudgment` | onDone | `isLossCutContinue` | — | `issueFix` | INV-LC3 |
| `lossCutJudgment` | onDone | （フォールバック） | — | `verificationFailed` | INV-LC2 |
| `issueFix` | `FIX_ISSUED` | — | `resetCurrentStep` | `typecheck` | MR-9（ループ） |
| （SP-3全体） | `after: 1800000` | — | — | `lossCutJudgment` | INV-LC5（30分タイマー） |

**30分タイマー（SP3-IE1）の設計**: SP-3の最上位（`verificationLoop`）に`after: { 1800000: '.lossCutJudgment' }`を定義する。XState v5ではcompound stateの`after`は、その状態にいる累積時間で発火する。SP-3に入った時点からカウントが始まり、どの子状態（typecheck/lint/test/issueFix）にいても30分経過で強制的に`lossCutJudgment`に遷移する。

**3回ルール（SP3-IE2）の設計**: `context.errorCount`で管理し、LC内の`check3Times`状態のガード`isErrorCount3OrMore`で判定する。検証失敗のたびに`incrementErrorCount`アクションでカウントが増加するため、3回目の失敗でLCに入った時点で`errorCount >= 3`がtrueとなり、LC-GW1が即座に損切り確定に遷移する（INV-LC5）。

### 3.8 SP-3のoutput定義（T6外部IFとの接続）

```typescript
// T6 §5.1 で定義されたSP3Output型に準拠
output: ({ context }) => ({
  passed: false, // verificationPassedに到達した場合のみtrue
  lossCut: false, // verificationFailedに到達した場合のみtrue
}),
```

**実装方針**: XState v5では、どの最終状態に到達したかによってoutputの値を変える。`verificationPassed`到達時は`{ passed: true, lossCut: false }`、`verificationFailed`到達時は`{ passed: false, lossCut: true }`を返す。

```typescript
// 最終状態ごとのoutput
verificationPassed: {
  type: 'final',
  // SP-3のoutputは親状態のoutputで算出
},
verificationFailed: {
  type: 'final',
},

// SP-3レベルのoutput
output: ({ self }) => {
  const finalState = self.getSnapshot().value;
  return {
    passed: finalState === 'verificationPassed',
    lossCut: finalState === 'verificationFailed',
  };
},
```

---

## 4. LC（損切り判断）の詳細仕様

### 4.1 設計方針

設計判断#1（A: 状態チェーン）に基づき、LC-GW1〜GW4をそれぞれ独立した状態として定義する。各状態は`always`（即時遷移）でガードを評価し、Yesなら`lossCutConfirmed`に直結、Noなら次の条件状態に遷移する。これによりINV-LC4（短絡評価）が構造的に保証される。

### 4.2 状態定義

| 状態名 | 種別 | entryアクション | 対応BPMN要素 |
|--------|------|----------------|-------------|
| `recordErrorState` | 通常状態（initial） | `recordCurrentErrorState` | LC-T1 |
| `check3Times` | 判定状態 | — | LC-GW1 |
| `check30Min` | 判定状態 | — | LC-GW2 |
| `checkComplexity` | 判定状態 | — | LC-GW3 |
| `checkRecurrence` | 判定状態 | — | LC-GW4 |
| `continueFix` | 最終状態 | — | LC-EE-CONT |
| `lossCutConfirmed` | 最終状態 | — | LC-EE-CUT |

**適用マッピングルール**: MR-1（LC-SE → `initial: 'recordErrorState'`）、MR-2（LC-EE-CONT/CUT → final）、MR-3（LC-T1 → 状態 + entry）、MR-5逐次GWパターン（LC-GW1〜GW4 → 状態チェーン + `always`遷移）

### 4.3 イベント定義

| イベント名 | 発火条件 | 発火元 |
|-----------|---------|--------|
| `ERROR_STATE_RECORDED` | エラー状態の記録が完了した | `recordCurrentErrorState`アクション |

**注記**: check3Times〜checkRecurrenceは`always`（即時遷移）を使用するため、イベントは不要。`recordErrorState`のみ外部イベントで完了を通知する。

### 4.4 ガード条件定義

| ガード名 | 条件 | 対応GW | 保証する不変条件 |
|---------|------|--------|----------------|
| `isErrorCount3OrMore` | `context.errorCount >= 3` | LC-GW1 | INV-LC2, INV-LC4, INV-LC5 |
| `isOver30Min` | `Date.now() - context.startedAt >= 1800000` | LC-GW2 | INV-LC2, INV-LC4, INV-LC5 |
| `isGrowingComplexity` | `context.errorHistory`の直近エントリで`complexityDelta === 'increased'` | LC-GW3 | INV-LC2, INV-LC4 |
| `isRecurringError` | `context.errorHistory`に同一エラーの過去修正済みレコードが存在する | LC-GW4 | INV-LC2, INV-LC4 |

**ガードの評価順序**: `check3Times` → `check30Min` → `checkComplexity` → `checkRecurrence`の順で状態が定義されるため、ガードの評価順序はINV-LC4の「3回→30分→複雑化→再発」と一致する。

### 4.5 アクション定義

| アクション名 | 種別 | 処理内容 |
|-------------|------|---------|
| `recordCurrentErrorState` | entry | 現在のエラー回数、経過時間、コード複雑度の変化を`context`のD-6相当フィールドに記録する |

### 4.6 遷移表

| 現在の状態 | イベント/条件 | ガード | アクション | 遷移先 | 対応不変条件 |
|-----------|-------------|--------|-----------|--------|------------|
| `recordErrorState` | `ERROR_STATE_RECORDED` | — | — | `check3Times` | INV-LC1 |
| `check3Times` | always | `isErrorCount3OrMore` | — | `lossCutConfirmed` | INV-LC2, INV-LC4 |
| `check3Times` | always | （フォールバック） | — | `check30Min` | INV-LC4 |
| `check30Min` | always | `isOver30Min` | — | `lossCutConfirmed` | INV-LC2, INV-LC4 |
| `check30Min` | always | （フォールバック） | — | `checkComplexity` | INV-LC4 |
| `checkComplexity` | always | `isGrowingComplexity` | — | `lossCutConfirmed` | INV-LC2, INV-LC4 |
| `checkComplexity` | always | （フォールバック） | — | `checkRecurrence` | INV-LC4 |
| `checkRecurrence` | always | `isRecurringError` | — | `lossCutConfirmed` | INV-LC2, INV-LC4 |
| `checkRecurrence` | always | （フォールバック） | — | `continueFix` | INV-LC3 |

**短絡評価の構造的保証**: 各判定状態の`always`遷移は上から順に評価される。`isErrorCount3OrMore`がtrueであれば即座に`lossCutConfirmed`に遷移し、`check30Min`以降は評価されない。これがINV-LC4の短絡評価を構造そのもので保証する。

### 4.7 LCのoutput定義

```typescript
output: ({ self }) => {
  const snapshot = self.getSnapshot();
  return {
    decision: snapshot.matches('lossCutConfirmed') ? 'cut' : 'continue',
  };
},
```

SP-3の`lossCutJudgment`状態のonDoneで、このoutputを参照して`issueFix`（continue）または`verificationFailed`（cut）に分岐する。

---

## 5. 中間イベント（IE1/IE2）の設計

### 5.1 SP3-IE1: 30分タイマー

| 項目 | 内容 |
|------|------|
| **BPMN要素** | SP3-IE1（中間タイマーイベント: 30分経過） |
| **適用ルール** | MR-6（中間タイマー → `after`遅延遷移） |
| **XState実装** | `verificationLoop`状態に`after: { 1800000: '.lossCutJudgment' }` |
| **発火タイミング** | SP-3に入ってから累計30分経過した時点（子状態の遷移に関わらず） |
| **遷移先** | `lossCutJudgment`（LC）。LC内で`check30Min`の`isOver30Min`ガードがtrueとなり、損切り確定に至る |
| **INV-LC5保証** | 30分経過で強制的にLCが起動されることを構造的に保証 |

### 5.2 SP3-IE2: 同一エラー3回目

| 項目 | 内容 |
|------|------|
| **BPMN要素** | SP3-IE2（中間エラーイベント: 同一エラー3回目） |
| **適用ルール** | MR-7（中間エラー → `context`条件 + ガード） |
| **XState実装** | `context.errorCount`で管理。各検証失敗時に`incrementErrorCount`アクションで加算 |
| **発火メカニズム** | 3回目の検証失敗でLCに遷移した際、LC内の`check3Times`で`isErrorCount3OrMore`がtrueとなる |
| **INV-LC5保証** | 3回目のエラーでLCが起動され、LC内で即座に損切り確定となることを保証 |

### 5.3 IE1とIE2の相互関係

- IE1（30分タイマー）とIE2（3回エラー）は**独立して動作**する
- IE1はSP-3レベルの`after`で管理され、IE2は`context.errorCount`で管理される
- いずれか一方が先に条件を満たせば、LCが起動される（INV-LC5: OR条件）
- 両方が同時に条件を満たした場合、`after`遅延遷移とイベント遷移の優先度はXState v5の仕様に従う（イベント遷移が優先）。いずれの場合もLCが起動されるため、結果は同一

---

## 6. DT-8/DT-9のステートチャートへの統合

### 6.1 協働原則チェック（DT-8: C1-C4）のentryアクション

```typescript
// 各検証状態（typecheck, lint, test）のentryアクション
function checkCollaborationPrinciples(context: VerificationLoopContext): PrincipleCheckResult {
  const violations: string[] = [];

  // C1: 人間がレビュー可能な形式か？
  if (!context.isHumanReviewable) {
    violations.push('C1: 人間がレビュー可能な形式でない');
  }
  // C2: 作業ログが残る仕組みがあるか？
  if (!context.hasWorkLog) {
    violations.push('C2: 作業ログが残らない');
  }
  // C3: 学びの記録が予定されているか？
  if (!context.hasLearningRecord) {
    violations.push('C3: 学びの記録がない');
  }
  // C4: チームで共有できる状態か？
  if (!context.isShareable) {
    violations.push('C4: チームで共有できない');
  }

  return {
    passed: violations.length === 0,
    violations,
  };
}
```

### 6.2 AI行動原則チェック（DT-9: A1-A4）のentryアクション

```typescript
function checkAIPrinciples(context: VerificationLoopContext): PrincipleCheckResult {
  const violations: string[] = [];

  // A1: タスクを1文で説明できるか？
  if (!context.isTaskExplainableInOneSentence) {
    violations.push('A1: タスクが1文で説明できない');
  }
  // A2: 完了条件が明確か？
  if (!context.hasClearCompletionCriteria) {
    violations.push('A2: 完了条件が不明確');
  }
  // A3: 検証方法と自信度が明示されているか？
  if (!context.hasVerificationMethod || !context.hasConfidenceLevel) {
    violations.push('A3: 検証方法または自信度が未明示');
  }
  // A4: Bright Lines違反がないか？
  if (context.hasBrightLinesViolation) {
    violations.push('A4: Bright Lines違反検出 → DT-0に戻る');
  }

  return {
    passed: violations.length === 0,
    violations,
  };
}
```

### 6.3 原則違反時の動作

| 原則違反 | 動作 | 理由 |
|---------|------|------|
| DT-8違反（C1-C4） | `context.collaborationCheckResult`に記録。検証プロセスは継続（即座に中断はしない） | 協働原則は「是正アクション」が必要だが、検証チェーンを中断する性質ではない。INV-CA1に従い、記録された違反は次の検証ループで是正される |
| DT-9のA4違反 | 損切り判断とは別に、Bright Lines違反としてメインフローのGW-1に戻る経路を取る | INV-CA2に従い、A4違反はDT-0に戻る。ただしSP-3内部ではこの遷移は直接表現せず、SP-3のoutputで通知し、メインフローで処理する（T8統合仕様で対応） |

---

## 7. 不変条件の保証マッピング

### 7.1 INV-SP3: 検証フィードバックループ不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-SP3-1** | 検証の実行順序はtypecheck→lint→testであり、変更できない | §3.3: `typecheck`がinitial状態。§3.7: `typecheck`→`lint`→`test`の一方向遷移 | `initial: 'typecheck'`により必ずtypecheckから開始。各状態からの遷移先は次の検証ステップのみであり、順序の逆転や飛び越しの経路は定義されていない |
| **INV-SP3-2** | 前段の検証が成功しない限り、次段の検証には進めない | §3.7: 各検証状態のガード`isXxxPass`がtrueの場合のみ次状態に遷移 | `typecheck`の`TYPECHECK_COMPLETE`イベントで`isTypecheckPass`がtrueの場合のみ`lint`に遷移。falseの場合は`lossCutJudgment`に遷移し、`lint`には到達不可。`lint`→`test`も同様 |
| **INV-SP3-3** | 協働原則チェック（SP3-T5 / DT-8）とAI行動原則チェック（SP3-T6 / DT-9）は検証プロセスと並行して常に監視される | §3.3: 全検証状態（typecheck, lint, test）のentryに`checkCollaborationPrinciples`と`checkAIPrinciples`を配置 | SP-3がアクティブな間、各検証ステップに入るたびにDT-8/DT-9が評価される。検証ループ（issueFix→typecheck）でも再度チェックされるため、SP-3の全期間にわたり原則監視が継続する |
| **INV-SP3-4** | いずれかの検証が失敗した場合、損切り判断（LC）が起動される | §3.7: 各検証状態のフォールバック遷移がすべて`lossCutJudgment`に直結 | typecheck/lint/testのいずれで失敗しても、フォールバック遷移で`lossCutJudgment`（LC）に遷移する。失敗時に他の状態に遷移する経路は定義されていない |
| **INV-SP3-5** | SP-3の正常終了は全3段階の検証がすべて成功した場合のみ | §3.7: `verificationPassed`への遷移は`test`状態の`isTestPass`ガードがtrueの場合のみ | `verificationPassed`（最終状態）への唯一の遷移元は`test`であり、`isTestPass`ガードが必須。`test`に到達するには`lint`を、`lint`に到達するには`typecheck`を通過する必要がある。よって3段階すべての成功が構造的に要求される |

### 7.2 INV-LC: 損切り判断不変条件群

| ID | 不変条件 | 保証する設計要素 | 保証方法 |
|----|---------|----------------|---------|
| **INV-LC1** | エラー状態の記録（LC-T1）は損切り判定の前に必ず実行される | §4.2: `recordErrorState`がinitial状態。§4.6: `ERROR_STATE_RECORDED`イベント後に`check3Times`に遷移 | `initial: 'recordErrorState'`によりLC開始時に必ず`recordErrorState`から始まる。`check3Times`に遷移するには`ERROR_STATE_RECORDED`イベントが必要であり、記録処理を経由せずに判定に進む経路は存在しない |
| **INV-LC2** | 損切り4条件（3回/30分/複雑化/再発）はOR条件で評価される | §4.6: 4つの判定状態すべてのYesガードが`lossCutConfirmed`に直結 | `check3Times`〜`checkRecurrence`の各`always`遷移で、ガードがtrueの場合はすべて`lossCutConfirmed`に遷移する。4条件のいずれか1つでも該当すれば損切り確定（OR条件） |
| **INV-LC3** | 4条件すべてに該当しない場合のみ修正継続 | §4.6: `checkRecurrence`のフォールバック遷移のみが`continueFix`に到達 | `continueFix`への唯一の遷移元は`checkRecurrence`のフォールバック（`isRecurringError`がfalse）。4条件すべてのNoを通過した場合にのみ到達可能 |
| **INV-LC4** | 4条件の評価順序は3回→30分→複雑化→再発、短絡評価 | §4.6: 状態チェーン`check3Times`→`check30Min`→`checkComplexity`→`checkRecurrence`の逐次遷移 | 各判定状態は`always`のYesガードで`lossCutConfirmed`に直結。Yesになった時点で後続の判定状態は到達不可（短絡評価）。状態の定義順序が評価順序を構造的に決定する |
| **INV-LC5** | 30分経過（SP3-IE1）または同一エラー3回目（SP3-IE2）は損切り判断を強制起動 | §5.1: `after: { 1800000: '.lossCutJudgment' }`。§5.2: `context.errorCount`による管理 | SP3-IE1: SP-3レベルの`after`遅延遷移により、30分経過でどの子状態からも強制的にLCに遷移。SP3-IE2: 3回目の失敗時に通常フローでLCに遷移し、LC内の`check3Times`で`isErrorCount3OrMore`がtrueとなり即座に損切り確定 |

---

## 8. XState v5 擬似コード

### 8.1 SP-3（verificationLoop）

```typescript
import { setup, assign } from 'xstate';

const verificationLoopMachine = setup({
  types: {
    context: {} as VerificationLoopContext,
    events: {} as
      | { type: 'TYPECHECK_COMPLETE'; result: { passed: boolean; error?: ErrorInfo } }
      | { type: 'LINT_COMPLETE'; result: { passed: boolean; error?: ErrorInfo } }
      | { type: 'TEST_COMPLETE'; result: { passed: boolean; error?: ErrorInfo } }
      | { type: 'ERROR_STATE_RECORDED' }
      | { type: 'FIX_ISSUED' },
  },
  guards: {
    isTypecheckPass: ({ event }) =>
      event.type === 'TYPECHECK_COMPLETE' && event.result.passed === true,
    isLintPass: ({ event }) =>
      event.type === 'LINT_COMPLETE' && event.result.passed === true,
    isTestPass: ({ event }) =>
      event.type === 'TEST_COMPLETE' && event.result.passed === true,
    isLossCutContinue: (_, params) =>
      params.output.decision === 'continue',
    // LC内部のガード
    isErrorCount3OrMore: ({ context }) => context.errorCount >= 3,
    isOver30Min: ({ context }) => Date.now() - context.startedAt >= 1800000,
    isGrowingComplexity: ({ context }) => {
      const latest = context.errorHistory[context.errorHistory.length - 1];
      return latest?.complexityDelta === 'increased';
    },
    isRecurringError: ({ context }) => {
      if (!context.lastError) return false;
      return context.errorHistory.some(
        (record) =>
          record.error.message === context.lastError!.message &&
          record.error.step === context.lastError!.step
      );
    },
  },
  actions: {
    runTypecheck: () => { /* typecheck 実行ロジック */ },
    runLint: () => { /* lint 実行ロジック */ },
    runTest: () => { /* test 実行ロジック */ },
    checkCollaborationPrinciples: () => { /* DT-8 評価ロジック */ },
    checkAIPrinciples: () => { /* DT-9 評価ロジック */ },
    recordCurrentErrorState: () => { /* エラー状態記録ロジック */ },
    issueFixInstruction: () => { /* 修正指示生成ロジック */ },
  },
}).createMachine({
  id: 'verificationLoop',
  initial: 'typecheck',
  context: {
    currentStep: 'typecheck',
    errorCount: 0,
    startedAt: Date.now(),
    lastError: null,
    errorHistory: [],
    collaborationCheckResult: null,
    aiPrincipleCheckResult: null,
  },

  // --- SP3-IE1: 30分タイマー（SP-3全体に適用） ---
  after: {
    1800000: '.lossCutJudgment',  // INV-LC5: 30分経過で強制LC起動
  },

  states: {
    // --- typecheck（SP3-T1 + SP3-T5 + SP3-T6） ---
    typecheck: {
      entry: [
        { type: 'runTypecheck' },
        { type: 'checkCollaborationPrinciples' },  // INV-SP3-3
        { type: 'checkAIPrinciples' },              // INV-SP3-3
      ],
      on: {
        TYPECHECK_COMPLETE: [
          {
            guard: 'isTypecheckPass',
            target: 'lint',                         // INV-SP3-1, INV-SP3-2
          },
          {
            actions: [
              assign({
                errorCount: ({ context }) => context.errorCount + 1,
                lastError: ({ event }) => event.result.error ?? null,
                errorHistory: ({ context, event }) => [
                  ...context.errorHistory,
                  {
                    error: event.result.error!,
                    fixAttempt: '',
                    complexityDelta: 'unchanged' as const,
                  },
                ],
              }),
            ],
            target: 'lossCutJudgment',              // INV-SP3-4
          },
        ],
      },
    },

    // --- lint（SP3-T2 + SP3-T5 + SP3-T6） ---
    lint: {
      entry: [
        { type: 'runLint' },
        { type: 'checkCollaborationPrinciples' },   // INV-SP3-3
        { type: 'checkAIPrinciples' },               // INV-SP3-3
      ],
      on: {
        LINT_COMPLETE: [
          {
            guard: 'isLintPass',
            target: 'test',                          // INV-SP3-2
          },
          {
            actions: [
              assign({
                errorCount: ({ context }) => context.errorCount + 1,
                lastError: ({ event }) => event.result.error ?? null,
                errorHistory: ({ context, event }) => [
                  ...context.errorHistory,
                  {
                    error: event.result.error!,
                    fixAttempt: '',
                    complexityDelta: 'unchanged' as const,
                  },
                ],
              }),
            ],
            target: 'lossCutJudgment',               // INV-SP3-4
          },
        ],
      },
    },

    // --- test（SP3-T3 + SP3-T5 + SP3-T6） ---
    test: {
      entry: [
        { type: 'runTest' },
        { type: 'checkCollaborationPrinciples' },   // INV-SP3-3
        { type: 'checkAIPrinciples' },               // INV-SP3-3
      ],
      on: {
        TEST_COMPLETE: [
          {
            guard: 'isTestPass',
            target: 'verificationPassed',            // INV-SP3-5
          },
          {
            actions: [
              assign({
                errorCount: ({ context }) => context.errorCount + 1,
                lastError: ({ event }) => event.result.error ?? null,
                errorHistory: ({ context, event }) => [
                  ...context.errorHistory,
                  {
                    error: event.result.error!,
                    fixAttempt: '',
                    complexityDelta: 'unchanged' as const,
                  },
                ],
              }),
            ],
            target: 'lossCutJudgment',               // INV-SP3-4
          },
        ],
      },
    },

    // --- 損切り判断（LC: compound state） ---
    lossCutJudgment: {
      initial: 'recordErrorState',
      states: {
        // LC-T1: エラー状態の記録（INV-LC1: 損切り判定前に必ず実行）
        recordErrorState: {
          entry: [{ type: 'recordCurrentErrorState' }],
          on: {
            ERROR_STATE_RECORDED: 'check3Times',
          },
        },

        // LC-GW1: 3回ルール判定（INV-LC4: 短絡評価の第1条件）
        check3Times: {
          always: [
            {
              guard: 'isErrorCount3OrMore',
              target: 'lossCutConfirmed',            // INV-LC2: OR条件
            },
            { target: 'check30Min' },                // INV-LC4: 次条件へ
          ],
        },

        // LC-GW2: 時間ルール判定（INV-LC4: 短絡評価の第2条件）
        check30Min: {
          always: [
            {
              guard: 'isOver30Min',
              target: 'lossCutConfirmed',            // INV-LC2: OR条件
            },
            { target: 'checkComplexity' },           // INV-LC4: 次条件へ
          ],
        },

        // LC-GW3: 複雑化ルール判定（INV-LC4: 短絡評価の第3条件）
        checkComplexity: {
          always: [
            {
              guard: 'isGrowingComplexity',
              target: 'lossCutConfirmed',            // INV-LC2: OR条件
            },
            { target: 'checkRecurrence' },           // INV-LC4: 次条件へ
          ],
        },

        // LC-GW4: 再発ルール判定（INV-LC4: 短絡評価の第4条件）
        checkRecurrence: {
          always: [
            {
              guard: 'isRecurringError',
              target: 'lossCutConfirmed',            // INV-LC2: OR条件
            },
            { target: 'continueFix' },               // INV-LC3: 全条件Noのみ
          ],
        },

        // LC-EE-CONT: 修正継続
        continueFix: { type: 'final' },

        // LC-EE-CUT: 損切り確定
        lossCutConfirmed: { type: 'final' },
      },

      // LCのoutput
      output: ({ context, self }) => ({
        decision: self.getSnapshot().matches('lossCutJudgment.lossCutConfirmed')
          ? 'cut'
          : 'continue',
      }),

      // LC完了時の分岐
      onDone: [
        {
          guard: 'isLossCutContinue',
          target: 'issueFix',                        // INV-LC3: 修正継続
        },
        {
          target: 'verificationFailed',              // INV-LC2: 損切り確定
        },
      ],
    },

    // --- 修正指示（LC-T2） ---
    issueFix: {
      entry: [{ type: 'issueFixInstruction' }],
      on: {
        FIX_ISSUED: {
          actions: assign({
            currentStep: () => 'typecheck' as const, // 検証先頭に戻る
          }),
          target: 'typecheck',                       // MR-9: ループ
        },
      },
    },

    // --- 最終状態 ---
    verificationPassed: { type: 'final' },           // SP3-EE-OK
    verificationFailed: { type: 'final' },           // SP3-EE-NG
  },

  // SP-3のoutput（T6 §5.1 SP3Output型に準拠）
  output: ({ self }) => {
    const snapshot = self.getSnapshot();
    return {
      passed: snapshot.matches('verificationPassed'),
      lossCut: snapshot.matches('verificationFailed'),
    };
  },
});
```

---

## 9. 出典

| ファイル | 参照セクション |
|---------|--------------|
| `phase1-t3-bpmn-elements.md` | §4: SP-3フロー要素定義、§4.2: フロー接続、§4.3: 損切り判定入力条件 |
| `phase1-t5-losscutrecovery-bpmn.md` | §1: LC（損切り判断サブプロセス）、§1.3: フロー接続 |
| `phase1-t7-invariants.md` | §5: INV-SP3（5条件）、§6: INV-LC（5条件） |
| `phase1-t4-mermaid-flowcharts.md` | §4: SP-3のMermaid図 |
| `phase1-t6-losscut-recovery-mermaid.md` | §1: LC Mermaid図、§5: 統合フロー図 |
| `phase1-t2-dmn-decision-tables.md` | DT-8（協働原則）、DT-9（AI行動原則） |
| `docs/bpmn-to-statechart-mapping.md` | MR-1〜MR-9マッピングルール |
| `docs/spec-l0l4-hierarchy.md` | T5: output定義方式 |
| `docs/spec-mainflow-sp2.md` | T6: §5.1 SP3Outputインターフェース、§3.7 verificationLoop外部IF |

---

## 10. T8への申し送り

| # | 項目 | 内容 |
|---|------|------|
| 1 | SP-3からの復帰フロー接続 | `verificationFailed`到達時のoutput `{ passed: false, lossCut: true }` をメインフローで受け取り、`lossCutExit`経由でRFに接続する設計。RFの入力として、SP-3の`context.errorHistory`（エラー履歴）と`context.lastError`（直近エラー）を渡す方法をT8で定義すること |
| 2 | DT-8/DT-9違反時のメインフロー復帰 | §6.3で記載のとおり、DT-9のA4違反（Bright Lines違反）はSP-3のoutputで通知し、メインフローのGW-1に戻る経路をT8の統合仕様で定義すること |
| 3 | LC-T2（issueFix）からtypecheckへの再開 | 修正後は常にtypecheckから再開する設計（INV-SP3-1に準拠）。lint/testから直接再開する最適化はT7スコープ外の将来拡張候補 |
| 4 | 設計判断#2の将来見直し | 並行監視をOption C（entry/exitアクション）で実現したが、実装段階でDT-8/DT-9の評価頻度やタイミングに課題が発見された場合、Option A（parallel state）への変更を検討すること |
| 5 | 命名規則の継続 | イベント: SCREAMING_SNAKE、状態: camelCase、ガード: camelCase（T5→T6→T7と同一） |
| 6 | LCのonDone output方式 | T5/T6と同様に`output`プロパティを使用。T8でもRF/ESに対して同方式を踏襲すること |
