# 付録D: ツール化対応表 — AI協働開発を支える判断支援ツール

> **この付録について**: ガイド本編で紹介したフローチャート・判断基準をプログラム化可能な形式で一覧化しています。各ツールには「Claudeへの依頼プロンプト」を添付しており、そのままコピーして利用できます。
>
> **設計思想**: 「プログラム化できる = 人間にも明確」— フローがツール化可能であることは、判断基準が客観的である証拠です。
>
> **対象読者**: AI協働開発に取り組むエンジニア、特にタスク設計や品質管理の効率化を目指す方。

---

## この付録の使い方

### 前提条件

この付録を活用するにあたり、以下の準備が整っていることを前提としています。

- **ガイド本編の理解**: [Part 0: 哲学と原則](./00-philosophy.md) と [Part 1: マインドセット](./01-mindset.md) を読んでいること
- **基本的な開発環境**: TypeScript + Node.js の実行環境があること(ツール実装時)
- **Claudeへのアクセス**: Claude Code、Claude.ai、または API 経由でClaudeと対話できること

### ツール選択の考え方

すべてのツールを一度に導入する必要はありません。まずは日常的に使用頻度が高いものから始めて、徐々に拡張していくことをお勧めします。

```
推奨される導入順序:

Phase 1(基礎)     Phase 2(実践)       Phase 3(最適化)
────────────────    ────────────────      ────────────────
1文1検証チェッカー   損切り判断            並列化判断
Type 1/2 分類       コーチング/           新会話開始判断
タスク分解支援        ティーチング判断      階層判断フロー
                   CLAUDE.md 関連
                   INVEST原則チェック
```

---

## ツール一覧

| # | ツール名 | 参照元 | 優先度 | 複雑度 | 主な用途 |
|---|---------|--------|--------|--------|----------|
| 1 | 1文1検証チェッカー | Part 0 | ★★★ | 低 | タスク設計の品質確認 |
| 2 | Type 1/2 分類判定 | Part 1 | ★★★ | 低 | 実行方法の決定 |
| 3 | タスク分解支援 | Part 1 | ★★★ | 中 | 5ステップの対話的支援 |
| 4 | 損切り判断 | Part 3 | ★★★ | 低 | 撤退タイミングの判定 |
| 5 | 損切り後復帰ナビ | Part 3 | ★★☆ | 低 | 復帰アプローチの選択 |
| 6 | 型設計パターン選択 | Part 2 | ★★★ | 低 | 適切な型パターンの判定 |
| 7 | コーチング/ティーチング判断 | Part 4 | ★★★ | 低 | 学習モードの切替支援 |
| 8 | CLAUDE.md テンプレート生成 | Part 3 | ★★☆ | 中 | Phase別テンプレート |
| 9 | CLAUDE.md Phase推定 | Part 3 | ★★☆ | 中 | 現在Phaseの逆算 |
| 10 | 習熟度レベル判定 | Part 4 | ★★☆ | 低 | L1/L2/L3の判定 |
| 11 | INVEST原則チェック | Part 1 | ★★☆ | 低 | タスク粒度の検証 |
| 12 | 並列化判断 | Part 1 | ★☆☆ | 低 | タスク並列実行の判定 |
| 13 | 新会話開始判断 | Part 2 | ★☆☆ | 低 | コンテキスト管理 |
| 14 | 階層判断フロー | Part 0 | ★☆☆ | 低 | 階層に基づく判断支援 |
| 15 | 可逆性チェック | Part 0 | ★★☆ | 低 | AI依存度の定期確認 |

**優先度の目安:**

- ★★★: 日常的に使用し、効果が高いツール
- ★★☆: 特定の場面で有効なツール
- ★☆☆: あると便利なツール

**複雑度の目安:**

- 低: 条件分岐が明確で、1ファイルで実装可能
- 中: 複数の判定ロジックや状態管理が必要
- 高: 外部連携や永続化が必要

---

## Phase 1: 基礎ツール — タスク設計の土台を固める

これから紹介する3つのツールは、AI協働開発の最も基本的な判断を支援します。タスクを「AIに渡せる形」に整えることが、成功への第一歩です。

---

## 1. 1文1検証チェッカー

### なぜ必要か

AI協働開発でよくある失敗パターンの一つは、「あれもこれも」と複数の作業を一度に依頼してしまうことです。結果として、Claudeは期待と異なる方向に進み、手戻りが発生します。

このツールは、タスク記述が「1文で説明でき、検証1回で完了確認できる」かを自動判定します。曖昧さや複合的な指示を検出し、分解すべきポイントを提案します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- タスクを渡す前に「これはAIに渡せる粒度か?」を客観的に判断できるようになります
- 「なんとなく大きい気がする」という曖昧な感覚が、具体的な分解ポイントとして可視化されます
- 結果として、1回の依頼で期待通りの成果が得られる確率が向上します

### 前提条件

- タスク記述が日本語で書かれていること
- タスクの目的が明確であること(何を達成したいかが分かっている状態)

### 使い方

1. タスク記述をテキストとして用意します
2. ツールにタスク記述を入力します
3. 判定結果と改善提案を確認します
4. 必要に応じてタスクを分解し、再度チェックします

### 期待される出力

**入力例:**
```
Userエンティティを作成してください
```

**出力例(合格):**
```json
{
  "canPassToAI": true,
  "checks": {
    "multipleActions": { "detected": false, "matches": [] },
    "conditional": { "detected": false, "matches": [] },
    "vague": { "detected": false, "matches": [] },
    "multipleDeliverables": { "detected": false, "matches": [] },
    "multipleVerification": { "detected": false, "matches": [] }
  },
  "auxiliaryIndicators": {
    "estimatedMinutes": 15,
    "estimatedFileCount": 1
  },
  "suggestions": [],
  "decompositionSuggestions": []
}
```

**入力例:**
```
エンティティを作成して、テストも書いて、ビルドが通ることも確認して
```

**出力例(要分解):**
```json
{
  "canPassToAI": false,
  "checks": {
    "multipleActions": { "detected": true, "matches": ["作成して、テストも書いて"] },
    "conditional": { "detected": false, "matches": [] },
    "vague": { "detected": false, "matches": [] },
    "multipleDeliverables": { "detected": true, "matches": ["エンティティ", "テスト"] },
    "multipleVerification": { "detected": true, "matches": ["テストも", "確認も"] }
  },
  "suggestions": [
    "タスクを3つに分解することを検討してください"
  ],
  "decompositionSuggestions": [
    "1. Userエンティティの型定義を作成する",
    "2. Userエンティティのテストを作成する",
    "3. ビルドが通ることを確認する"
  ]
}
```

### トラブルシューティング

<details>
<summary>正しい記述なのに「要分解」と判定される場合</summary>

**症状:** 技術的には1つのタスクなのに、複数アクションとして検出される

**原因:** パターンマッチが過剰に反応している可能性があります

**対処法:**
1. `strictMode: false` で再度チェックしてください
2. 日本語の表現を言い換えてみてください(「〜して」を「〜する」に変更するなど)
3. それでも問題が続く場合は、判定ロジックのパターンを調整してください
</details>

<details>
<summary>曖昧な表現が検出されない場合</summary>

**症状:** 「適切に」「うまく」などの表現が見逃される

**原因:** 検出パターンに含まれていない表現の可能性があります

**対処法:**
1. `strictMode: true` で再度チェックしてください
2. 検出パターンリストを確認し、必要に応じて追加してください
</details>

### Claudeへの依頼プロンプト

以下のプロンプトをClaudeに渡すことで、このツールを実装できます。

```markdown
以下の仕様でタスクチェッカーを実装してください。

## 機能概要

タスク記述テキストを受け取り、「1文1検証」原則を満たすか判定します。

## 背景

AI協働開発において、タスクの粒度は成功率を大きく左右します。
「1文で説明でき、検証1回で完了確認できる」タスクは、Claudeが正しく理解し、
期待通りの成果を出せる可能性が高くなります。

## 入力

- `taskDescription`: タスク記述(文字列)
- `strictMode`: オプション、デフォルト false(true の場合、警告も失敗として扱う)

## 判定ルール

以下のパターンを検出したら「分解が必要」と判定します。

### 1. 複数アクション(❌ 必ず失敗)
- パターン: 「〜して〜して」「〜し、〜する」「したら〜する」
- 例: 「エンティティを作成して、APIも実装して」

### 2. 条件分岐(❌ 必ず失敗)
- パターン: 「もし」「〜の場合」「〜なら」「〜ならば」
- 例: 「もしエラーなら再試行して」

### 3. 曖昧表現(⚠️ 警告、strictModeでは失敗)
- パターン: 「適切に」「いい感じに」「うまく」「よしなに」
- 例: 「適切にエラーハンドリングして」

### 4. 複数成果物(❌ 必ず失敗)
- パターン: 「〜と〜を作成」「〜および〜を実装」「〜も作って」
- 例: 「UserエンティティとAPIを作成して」

### 5. 複数検証(❌ 必ず失敗)
- パターン: 「テストも」「ビルドも」「確認も」「〜と〜を実行」
- 例: 「実装してテストも書いて」

## 出力スキーマ

```typescript
interface TaskCheckResult {
  canPassToAI: boolean;        // AIに渡せるか
  checks: {
    multipleActions: { detected: boolean; matches: string[] };
    conditional: { detected: boolean; matches: string[] };
    vague: { detected: boolean; matches: string[] };
    multipleDeliverables: { detected: boolean; matches: string[] };
    multipleVerification: { detected: boolean; matches: string[] };
  };
  auxiliaryIndicators: {
    estimatedMinutes: number;  // 推定所要時間(30分以内が目安)
    estimatedFileCount: number; // 推定変更ファイル数(1-3が目安)
  };
  suggestions: string[];       // 改善提案
  decompositionSuggestions: string[]; // 分解案
}
```

## 技術スタック

TypeScript + zod(入出力の型定義とバリデーション)

## テストケース

1. 「Userエンティティを作成してください」→ canPassToAI: true
2. 「エンティティを作成して、テストも書いて」→ canPassToAI: false
3. 「もしエラーならリトライして」→ canPassToAI: false
4. 「適切にバリデーションを追加」→ strictMode時のみ false

## 検証方法

実装後、以下のコマンドで型チェックを実行し、結果を報告してください。

```bash
npx tsc --noEmit
```
```

---

タスクが「AIに渡せる粒度」であることを確認したら、次はそのタスクをどのように実行するかを決める必要があります。Type 1(新規作成)とType 2(既存参照)では、Claudeへの渡し方が異なります。

---

## 2. Type 1/2 分類判定

### なぜ必要か

Claudeにタスクを依頼する際、「既存のコードを参照する必要があるか」によって、必要な準備が大きく異なります。

- **Type 1(新規作成)**: 既存コードを参照せずに実行できる。テンプレートやボイラープレートの生成など
- **Type 2(既存参照)**: プロジェクト固有のコンテキストが必要。既存APIの拡張など

この判断を誤ると、Type 2のタスクにコンテキストを渡さずに実行し、プロジェクトの規約に沿わないコードが生成されてしまいます。このツールは、タスク記述から自動的にType分類を判定し、必要なコンテキストを提示します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「このタスクには何を渡せばいいか」が明確になります
- Type 2タスクで必要なファイルやコンテキストの漏れを防げます
- 「迷ったらType 2」という原則に沿った安全側の判断ができます

### 前提条件

- タスク記述が用意されていること
- プロジェクトの基本的な構成を理解していること

### 使い方

1. タスク記述をテキストとして入力します
2. 判定結果(Type 1 または Type 2)を確認します
3. Type 2 の場合、提示された「必要なコンテキスト」を準備します
4. 準備が整ったら、Claudeにタスクを依頼します

### 期待される出力

**入力例:**
```
Userエンティティを新規作成してください
```

**出力例:**
```json
{
  "type": "Type1",
  "confidence": 0.85,
  "reason": "「新規作成」キーワードが検出され、既存コード参照の指標がありません",
  "indicators": {
    "type1": [{ "pattern": "新規作成", "weight": 0.8 }],
    "type2": []
  },
  "contextNeeded": []
}
```

**入力例:**
```
既存のauth.tsを参考にログインAPIを実装してください
```

**出力例:**
```json
{
  "type": "Type2",
  "confidence": 0.92,
  "reason": "「既存の」「参考に」キーワードと具体的なファイル参照が検出されました",
  "indicators": {
    "type1": [],
    "type2": [
      { "pattern": "既存の", "weight": 0.9 },
      { "pattern": "参考に", "weight": 0.8 },
      { "pattern": ".ts", "weight": 0.7 }
    ]
  },
  "contextNeeded": [
    "auth.ts の内容",
    "プロジェクトの認証アーキテクチャの概要",
    "API設計のコーディング規約"
  ]
}
```

### トラブルシューティング

<details>
<summary>Type 1 と判定されたが、実際は既存コードを参照すべきだった場合</summary>

**症状:** 生成されたコードがプロジェクトの規約に沿っていない

**原因:** タスク記述に既存コード参照の必要性が明示されていなかった可能性があります

**対処法:**
1. タスク記述に「既存の〇〇を参考に」「プロジェクトの〇〇に合わせて」などの文言を追加してください
2. 判定ルールの重みを調整し、より安全側(Type 2寄り)に倒すことを検討してください
3. 「迷ったらType 2」の原則に従い、手動でType 2として扱ってください
</details>

<details>
<summary>判定の確信度が低い場合</summary>

**症状:** confidence が 0.5 前後で、判断に迷う

**対処法:**
1. Type 2 として扱い、必要なコンテキストを準備してください(安全側に倒す)
2. タスク記述をより具体的に書き直してください
3. 不明な点があれば、Claudeに「このタスクを実行するのに何が必要ですか?」と質問してください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様でType分類判定ツールを実装してください。

## 機能概要

タスク記述から Type 1(新規作成)か Type 2(既存参照)かを判定します。

## 背景

- Type 1: 既存コードを参照せずに実行できるタスク
- Type 2: 既存コードやプロジェクト固有の情報が必要なタスク
- 「迷ったらType 2」が原則です(コンテキスト不足による失敗を防ぐため)

## 入力

- `taskDescription`: タスク記述(文字列)

## 判定ルール

### Type 1 の指標(重み付き)

| パターン | 重み |
|---------|------|
| 「新規作成」「新しく作成」「新規追加」 | 0.8 |
| 「ボイラープレート」「テンプレートから」「雛形」 | 0.7 |
| 「console.log削除」「未使用import削除」(機械的作業) | 0.9 |
| 「フォーマット」「整形」 | 0.6 |

### Type 2 の指標(重み付き)

| パターン | 重み |
|---------|------|
| 「既存の」「既存コード」 | 0.9 |
| 「参照」「参考に」「〜のように」「〜を見て」 | 0.8 |
| 「このファイル」「そのコード」「〜.ts」「〜.js」 | 0.7 |
| 「プロジェクトの」「プロジェクト固有」「現行の」 | 0.8 |

### 判定ロジック

1. 各指標の重みを合計します
2. Type 1 スコア > Type 2 スコア + 0.3 の場合 → Type 1
3. それ以外 → Type 2(安全側に倒す)

## 出力スキーマ

```typescript
interface TypeClassificationResult {
  type: 'Type1' | 'Type2';
  confidence: number;           // 0-1
  reason: string;               // 判定理由
  indicators: {
    type1: { pattern: string; weight: number }[];
    type2: { pattern: string; weight: number }[];
  };
  contextNeeded: string[];      // Type 2の場合、必要なコンテキスト
}
```

## 技術スタック

TypeScript + zod

## テストケース

1. 「Userエンティティを新規作成してください」→ Type 1
2. 「既存のauth.tsを参考にログインAPIを実装」→ Type 2
3. 「APIを実装してください」→ Type 2(曖昧なので安全側に)
4. 「console.logをすべて削除」→ Type 1(機械的作業)

## 検証方法

```bash
npx tsc --noEmit
```
```

---

1文1検証チェックで「分解が必要」と判定された場合、具体的にどう分解すればよいでしょうか。次のツールは、その分解プロセスを5ステップで支援します。

---

## 3. タスク分解支援

### なぜ必要か

「ログイン機能を実装する」のような大きなタスクを、そのままClaudeに渡しても期待通りの結果は得られません。しかし、「どこまで細かく分解すればよいか」の判断は経験が必要です。

このツールは、大きなタスクを「1文1検証」を満たす粒度まで分解する5ステップを対話的に支援します。分解の過程で、各タスクのType分類や依存関係も整理されます。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「どこまで分解すればよいか」の判断基準が明確になります
- タスク間の依存関係が可視化され、実行順序を最適化できます
- 各タスクに適切なTypeラベルが付与され、必要なコンテキストが明示されます

### 前提条件

- 達成したいゴールが明確であること
- プロジェクトの技術スタックが決まっていること(オプションだが、あると精度が上がる)

### 使い方

このツールは5つのステップで対話的に進みます。

**Step 1: ゴールを1文で書く**
- 元のタスク記述を入力します
- ツールが曖昧さを検出し、明確化を促します

**Step 2: 成果物をリストアップ**
- ゴールから必要な成果物を抽出します
- 漏れがないか確認し、必要に応じて追加します

**Step 3: 依存関係を整理**
- 成果物間の依存関係を分析します
- 実行順序が提案されます

**Step 4: 「1文1検証」単位に調整**
- 各成果物を1文1検証チェッカーで検証します
- 失敗したものは再分解が提案されます

**Step 5: Type 1/2 を判定**
- 各タスクにType分類を適用します
- 必要なコンテキストが明示されます

### 期待される出力

**入力例:**
```json
{
  "originalTask": "ログイン機能を実装する",
  "projectContext": {
    "techStack": ["TypeScript", "Express", "PostgreSQL", "zod"],
    "existingFiles": ["src/auth/", "src/entities/"],
    "conventions": ["JWT認証を使用", "zodでバリデーション"]
  }
}
```

**出力例:**
```json
{
  "clarifiedGoal": "メール/パスワードでログインし、JWTトークンを発行する機能を実装する",
  "deliverables": [
    "Userエンティティの型定義",
    "ログインAPIエンドポイント",
    "JWT発行ロジック",
    "入力バリデーション",
    "テスト"
  ],
  "dependencyGraph": {
    "Userエンティティの型定義": [],
    "入力バリデーション": ["Userエンティティの型定義"],
    "JWT発行ロジック": [],
    "ログインAPIエンドポイント": ["Userエンティティの型定義", "入力バリデーション", "JWT発行ロジック"],
    "テスト": ["ログインAPIエンドポイント"]
  },
  "tasks": [
    {
      "id": "1",
      "description": "Userエンティティの型定義を作成する",
      "type": "Type1",
      "verification": "npx tsc --noEmit",
      "dependencies": [],
      "estimatedMinutes": 10
    },
    {
      "id": "2",
      "description": "既存のsrc/auth/を参考に、ログイン入力のzodスキーマを作成する",
      "type": "Type2",
      "verification": "npx tsc --noEmit",
      "dependencies": ["1"],
      "contextNeeded": ["src/auth/ 配下の既存バリデーションパターン"],
      "estimatedMinutes": 15
    },
    {
      "id": "3",
      "description": "JWT発行関数を作成する",
      "type": "Type1",
      "verification": "npx tsc --noEmit",
      "dependencies": [],
      "estimatedMinutes": 15
    },
    {
      "id": "4",
      "description": "既存のAPIパターンに合わせて、POST /login エンドポイントを作成する",
      "type": "Type2",
      "verification": "npx tsc --noEmit && curl テスト",
      "dependencies": ["1", "2", "3"],
      "contextNeeded": ["既存のAPIエンドポイント実装例", "エラーハンドリングの規約"],
      "estimatedMinutes": 20
    },
    {
      "id": "5",
      "description": "ログインAPIのテストを作成する",
      "type": "Type2",
      "verification": "npm test",
      "dependencies": ["4"],
      "contextNeeded": ["既存のテストパターン"],
      "estimatedMinutes": 20
    }
  ],
  "executionOrder": ["1", "3", "2", "4", "5"],
  "estimatedTotalMinutes": 80
}
```

### トラブルシューティング

<details>
<summary>分解後のタスクがまだ大きいと感じる場合</summary>

**症状:** 1つのタスクの見積もり時間が30分を超える

**対処法:**
1. 該当タスクを再度1文1検証チェッカーにかけてください
2. 成果物をさらに細かい単位(関数単位、ファイル単位)で分解してください
3. 「このタスクで変更するファイルは何個か?」を基準に判断してください(1-3ファイルが目安)
</details>

<details>
<summary>依存関係が複雑すぎる場合</summary>

**症状:** 循環依存や複雑な依存グラフが生成される

**対処法:**
1. 成果物の抽象度を見直してください(インターフェースと実装を分離するなど)
2. 依存関係を減らすために、モックやスタブを使った分離を検討してください
3. 並列実行可能なタスクを特定し、効率化を図ってください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様でタスク分解支援ツールを実装してください。

## 機能概要

大きなタスクを「1文1検証」を満たす粒度まで分解する5ステップを支援します。

## 5ステップの定義

### Step 1: ゴールを1文で書く
- 入力: 元のタスク記述
- 処理: 曖昧さを検出し、明確化を促す
- 出力: 明確化されたゴール文

### Step 2: 成果物をリストアップ
- 入力: ゴール文
- 処理: 必要な成果物(ファイル、機能、テスト等)を抽出
- 出力: 成果物リスト

### Step 3: 依存関係を整理
- 入力: 成果物リスト
- 処理: 依存関係を分析し、実行順序を決定
- 出力: 順序付き成果物リスト(依存関係グラフ)

### Step 4: 「1文1検証」単位に調整
- 入力: 順序付き成果物リスト
- 処理: 各成果物を「1文1検証チェッカー」で検証
- 出力: 分解されたタスクリスト

### Step 5: Type 1/2 を判定
- 入力: 分解されたタスクリスト
- 処理: 各タスクにType分類を適用
- 出力: Type付きタスクリスト

## 入力スキーマ

```typescript
interface TaskDecompositionInput {
  originalTask: string;
  projectContext?: {
    techStack?: string[];
    existingFiles?: string[];
    conventions?: string[];
  };
}
```

## 出力スキーマ

```typescript
interface TaskDecompositionResult {
  clarifiedGoal: string;
  deliverables: string[];
  dependencyGraph: Record<string, string[]>;
  tasks: {
    id: string;
    description: string;
    type: 'Type1' | 'Type2';
    verification: string;
    dependencies: string[];
    contextNeeded?: string[];
    estimatedMinutes: number;
  }[];
  executionOrder: string[];
  estimatedTotalMinutes: number;
}
```

## 技術スタック

TypeScript + zod

※ 内部で「1文1検証チェッカー」と「Type分類判定」を使用します

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## Phase 2: 実践ツール — 開発中の判断を支援する

タスク設計の基礎が固まったら、次は開発中に発生する判断を支援するツールを導入します。特に「いつ撤退するか」「どう学ぶか」の判断は、経験がないと難しいものです。

---

## 4. 損切り判断

### なぜ必要か

エラー対応を続けていると、「もう少しで解決できそう」という感覚に陥りがちです。しかし、同じアプローチで3回以上試行しても解決しない場合、そのアプローチ自体に問題がある可能性が高いです。

このツールは、エラー対応の継続/撤退を客観的に判定し、撤退時の最適なアプローチを提案します。「撤退は敗北ではない」という考え方を、具体的なルールとして実装します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「もう少しで...」という感覚に流されず、客観的な判断ができます
- 撤退を「失敗」ではなく「戦略的な選択」として捉えられるようになります
- 撤退後の具体的な行動指針が得られ、次のステップに迷いません

### 前提条件

- 現在エラー対応中であること
- 往復回数や経過時間を把握していること

### 使い方

1. 現在の状況(往復回数、経過時間、複雑化傾向)を入力します
2. 判定結果(継続/検討/推奨)を確認します
3. 「推奨」または「検討」の場合、提案されたアプローチを検討します
4. 選択したアプローチに従って行動します

### 期待される出力

**入力例(継続可能):**
```json
{
  "roundTrips": 2,
  "elapsedMinutes": 15,
  "isGettingMoreComplex": false
}
```

**出力例:**
```json
{
  "decision": "continue",
  "triggeredRules": [],
  "reasoning": "まだ損切りラインに達していません。もう少し試行できます。",
  "recommendedApproach": null,
  "approachDescription": "",
  "nextSteps": [
    "現在のアプローチを続けてください",
    "次の試行でも解決しない場合、別のアプローチを検討してください"
  ]
}
```

**入力例(損切り推奨):**
```json
{
  "roundTrips": 4,
  "elapsedMinutes": 45,
  "isGettingMoreComplex": true,
  "sameErrorRecurred": true
}
```

**出力例:**
```json
{
  "decision": "recommend",
  "triggeredRules": [
    "3回ルール(往復4回)",
    "時間ルール(45分経過)",
    "複雑化ルール",
    "再発ルール"
  ],
  "reasoning": "複数の損切りラインに達しています。現在のアプローチでは解決が難しい可能性が高いです。",
  "recommendedApproach": "B",
  "approachDescription": "問題を再分解 — タスクがまだ大きすぎた可能性があります。より小さなタスクに分割し、最小タスクから再挑戦します。",
  "nextSteps": [
    "現在のセッションで得た情報を整理してください",
    "タスクを再分解し、最小の検証可能な単位を特定してください",
    "CLAUDE.mdに今回の学びを記録してください",
    "新しいセッションで最小タスクから再開してください"
  ]
}
```

### トラブルシューティング

<details>
<summary>「継続」と判定されたが、行き詰まりを感じる場合</summary>

**症状:** ルール上は継続可能だが、進展がない

**対処法:**
1. 「複雑化傾向」を再評価してください(コードが複雑になっていないか?)
2. 同じエラーメッセージが繰り返し出ていないか確認してください
3. 判定ルールに関わらず、直感的に「限界」を感じたら撤退を検討してください
</details>

<details>
<summary>提案されたアプローチが適切でないと感じる場合</summary>

**症状:** アプローチBが提案されたが、Cの方が適切に思える

**対処法:**
1. アプローチの選択基準を確認してください
2. 複数のルールに該当する場合、より優先度の高いアプローチを選択してください
3. 最終的な判断は人間が行います。ツールの提案は参考情報として活用してください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様で損切り判断ツールを実装してください。

## 機能概要

エラー対応を継続すべきか撤退すべきかを判定し、撤退時のアプローチを提案します。

## 背景

「撤退は敗北ではない」— AIの限界を知り、効率的にゴールに向かうためのスキルです。

## 入力スキーマ

```typescript
interface CutLossInput {
  roundTrips: number;           // 往復回数(同じエラーへの対応回数)
  elapsedMinutes: number;       // 経過時間(分)
  isGettingMoreComplex: boolean; // 修正のたびにコードが複雑になっているか
  sameErrorRecurred?: boolean;  // 同じエラーが再発したか
}
```

## 判定ルール

### 🚨 3回ルール(最優先)
- 条件: roundTrips >= 3
- 判定: 損切り「推奨」
- 理由: 同じアプローチでは解決しない可能性が高い

### 🚨 時間ルール
- 条件: elapsedMinutes >= 30
- 判定: 損切り「検討」
- 理由: 1つのエラーに時間をかけすぎている

### 🚨 複雑化ルール
- 条件: isGettingMoreComplex === true
- 判定: 損切り「推奨」
- 理由: 問題の分解が足りていないサイン

### 🚨 再発ルール
- 条件: sameErrorRecurred === true
- 判定: 損切り「推奨」
- 理由: 根本原因が特定できていない

## アプローチの定義

| ID | アプローチ | 説明 | 選択基準 |
|----|-----------|------|---------|
| A | 人間が直接解決 | 自分でコードを書く → 動いたらClaudeに解説させる | 自分で解法がわかる |
| B | 問題を再分解 | より小さなタスクに分割 → 最小タスクから再挑戦 | タスクが大きすぎた |
| C | コンテキストをリセット | 新しいセッションを開始 → CLAUDE.mdに学びを追記 | コンテキストが汚染されている |
| D | エスカレーション | チームメンバーに相談 → ペアプログラミング | 自分では判断できない |

### アプローチ選択の優先順位

複数の損切りルールに該当した場合、以下の優先順位でアプローチを選択します。

```
┌─────────────────────────────────────────────────────────────────┐
│                アプローチ選択フローチャート                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  複数ルールに該当                                               │
│       │                                                         │
│       ▼                                                         │
│  ┌────────────────┐                                             │
│  │ 複雑化ルール    │──Yes──→ アプローチB(問題を再分解)         │
│  │ に該当?        │         タスクの分解が足りていない          │
│  └───────┬────────┘                                             │
│         No                                                      │
│          ▼                                                      │
│  ┌────────────────┐                                             │
│  │ 自分で解法が   │──Yes──→ アプローチA(人間が直接解決)        │
│  │ わかる?        │         Claudeに頼らず自力で解決            │
│  └───────┬────────┘                                             │
│         No                                                      │
│          ▼                                                      │
│  ┌────────────────┐                                             │
│  │ 会話が長い     │──Yes──→ アプローチC(コンテキストリセット)  │
│  │ or 矛盾発生?   │         新しいセッションで再開              │
│  └───────┬────────┘                                             │
│         No                                                      │
│          ▼                                                      │
│  アプローチD(エスカレーション)                                 │
│  チームメンバーに相談                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| 状況 | 推奨アプローチ | 理由 |
|------|--------------|------|
| 複雑化 + 再発 | B | タスク分解の問題が根本原因 |
| 時間超過のみ | C | コンテキスト汚染の可能性 |
| 3回 + 解法わかる | A | 人間の知識で解決可能 |
| 判断に迷う | D | 第三者の視点が必要 |

> 💡 **原則**: 迷ったらアプローチD(エスカレーション)。
> 一人で抱え込まず、チームの力を借りることも重要なスキルです。

## 出力スキーマ

```typescript
interface CutLossResult {
  decision: 'continue' | 'consider' | 'recommend';
  triggeredRules: string[];
  reasoning: string;
  recommendedApproach: 'A' | 'B' | 'C' | 'D' | null;
  approachDescription: string;
  nextSteps: string[];
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 5. 損切り後復帰ナビ

### なぜ必要か

損切りを決断した後、「具体的に何をすればよいか」で迷うことがあります。特に、撤退後のアプローチ(A/B/C/D)を選んだ後の具体的な手順は、状況によって異なります。

このツールは、選択したアプローチに応じた具体的な復帰手順を提示します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 撤退後の「次に何をするか」が明確になります
- 失敗から学びを抽出し、CLAUDE.mdに記録する流れが自然にできます
- 復帰後の成功確率が向上します

### 前提条件

- 損切り判断ツールでアプローチを選択済みであること
- エラーの内容や試行した内容を把握していること

### 使い方

1. 選択したアプローチ(A/B/C/D)を入力します
2. 現在の状況(エラー内容、試行した内容)を入力します
3. 提示された復帰手順に従って行動します

### 期待される出力

**入力例:**
```json
{
  "selectedApproach": "B",
  "currentContext": {
    "errorMessage": "Type 'string' is not assignable to type 'number'",
    "attemptedSolutions": [
      "型キャストを追加",
      "zodスキーマを修正",
      "入力値の検証を追加"
    ],
    "taskDescription": "既存のAPIレスポンスを新しい型定義に合わせて修正する"
  }
}
```

**出力例:**
```json
{
  "approachDetails": {
    "name": "問題を再分解",
    "principle": "現在のタスクがまだ大きすぎた可能性があります。より小さな単位で検証しながら進めます。"
  },
  "recoverySteps": [
    {
      "step": 1,
      "action": "現在のタスクを分析する",
      "details": "「既存のAPIレスポンスを新しい型定義に合わせて修正する」というタスクを、以下の観点で分析してください:\n- 変更が必要なファイルは何個か?\n- 型の不一致は何箇所あるか?\n- 依存関係はどうなっているか?"
    },
    {
      "step": 2,
      "action": "最小の検証可能単位を特定する",
      "details": "型の不一致が発生している最も上流の箇所を特定してください。そこから順に修正していきます。"
    },
    {
      "step": 3,
      "action": "CLAUDE.mdに学びを記録する",
      "details": "以下の内容を記録してください:\n- 今回のタスクで発生した問題\n- 試行した解決策とその結果\n- 次回同様のタスクで注意すべきこと"
    },
    {
      "step": 4,
      "action": "最小タスクから再開する",
      "details": "新しいセッションで、特定した最小タスクから再開してください。"
    }
  ],
  "claudeMdTemplate": "## 学び: APIレスポンス型修正タスク (YYYY-MM-DD)\n\n### 発生した問題\n- 型の不一致: 'string' vs 'number'\n\n### 試行した解決策\n1. 型キャストを追加 → 根本解決にならず\n2. zodスキーマを修正 → 別の箇所でエラー\n3. 入力値の検証を追加 → 同上\n\n### 学び\n- このタスクは「1文1検証」を満たしていなかった\n- 型の不一致は上流から順に修正すべき\n- 次回は先に型の依存関係を整理する"
}
```

### Claudeへの依頼プロンプト

```markdown
以下の仕様で損切り後復帰ナビを実装してください。

## 機能概要

損切り後に選択したアプローチに応じた具体的な復帰手順を提示します。

## 入力スキーマ

```typescript
interface RecoveryNavInput {
  selectedApproach: 'A' | 'B' | 'C' | 'D';
  currentContext: {
    errorMessage?: string;
    attemptedSolutions?: string[];
    taskDescription?: string;
  };
}
```

## 出力スキーマ

```typescript
interface RecoveryNavResult {
  approachDetails: {
    name: string;
    principle: string;
  };
  recoverySteps: {
    step: number;
    action: string;
    details: string;
  }[];
  claudeMdTemplate: string;  // CLAUDE.mdに追記するテンプレート
}
```

## アプローチ別の復帰手順

### アプローチA: 人間が直接解決
1. 問題の本質を整理する
2. 自分でコードを書いて解決する
3. 動いたらClaudeに解説させる
4. CLAUDE.mdに学びを記録する

### アプローチB: 問題を再分解
1. 現在のタスクを分析する
2. 最小の検証可能単位を特定する
3. CLAUDE.mdに学びを記録する
4. 最小タスクから再開する

### アプローチC: コンテキストをリセット
1. 現在のセッションの情報を整理する
2. CLAUDE.mdに学びを記録する
3. 新しいセッションを開始する
4. CLAUDE.mdを参照しながら再開する

### アプローチD: エスカレーション
1. 問題を整理して説明できるようにする
2. チームメンバーに相談する
3. ペアプログラミングで解決する
4. CLAUDE.mdに学びを記録する

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

損切りと復帰の判断ができるようになったら、次は「Claudeとの対話スタイル」を最適化しましょう。知識を教えてもらうのか、考えを整理するのかで、適切な依頼の仕方が変わります。

---

## 6. 型設計パターン選択

### なぜ必要か

TypeScriptでの型設計には、状況に応じた複数のパターンがあります。「Branded Types」「Discriminated Unions」「Conditional Types」など、それぞれに適した場面があります。

このツールは、解決したい問題に対して最適な型設計パターンを提案し、その理由と実装例を提示します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「この場面ではどの型パターンを使うべきか」の判断が速くなります
- 選んだパターンの「なぜ」が理解でき、応用が利くようになります
- Claudeに型設計を依頼する際の的確な指示ができるようになります

### 前提条件

- TypeScriptを使用していること
- 解決したい型の問題が明確であること

### 使い方

1. 解決したい問題(「IDの取り違えを防ぎたい」など)を入力します
2. 推奨されるパターンと代替パターンを確認します
3. 各パターンの実装例を参考に、プロジェクトに適用します

### 期待される出力

**入力例:**
```json
{
  "problem": "UserIdとPostIdを取り違えるバグを防ぎたい",
  "context": {
    "currentApproach": "両方ともstringで定義している",
    "techStack": ["TypeScript", "zod"]
  }
}
```

**出力例:**
```json
{
  "recommendedPattern": {
    "name": "Branded Types",
    "reason": "同じプリミティブ型(string)を区別するために最適です。コンパイル時に型の取り違えを検出できます。",
    "implementation": "type UserId = string & { readonly brand: unique symbol };\ntype PostId = string & { readonly brand: unique symbol };\n\n// zodと組み合わせる場合\nconst UserIdSchema = z.string().brand<'UserId'>();\nconst PostIdSchema = z.string().brand<'PostId'>();",
    "tradeoffs": [
      "実行時のオーバーヘッドはゼロ",
      "型の作成に若干のボイラープレートが必要",
      "デバッグ時に実際の値が見えにくくなることがある"
    ]
  },
  "alternativePatterns": [
    {
      "name": "Newtype Pattern (クラスラッパー)",
      "reason": "実行時にも区別が必要な場合に有効です。",
      "tradeoffs": ["実行時オーバーヘッドあり", "シリアライズ/デシリアライズが複雑になる"]
    }
  ],
  "claudePromptSuggestion": "UserIdとPostIdをBranded Typesで実装してください。zodのbrand機能を使用し、型の取り違えをコンパイル時に検出できるようにしてください。"
}
```

### Claudeへの依頼プロンプト

```markdown
以下の仕様で型設計パターン選択ツールを実装してください。

## 機能概要

解決したい問題に対して最適な型設計パターンを提案します。

## 対応するパターン

| パターン | 主な用途 |
|---------|---------|
| Branded Types | 同じプリミティブ型の区別 |
| Discriminated Unions | 状態の網羅的なハンドリング |
| Conditional Types | 型レベルでの条件分岐 |
| Template Literal Types | 文字列パターンの型定義 |
| Mapped Types | 既存型からの派生型生成 |
| Utility Types | 既存型の変換(Partial, Required等) |

## 入力スキーマ

```typescript
interface TypePatternInput {
  problem: string;
  context?: {
    currentApproach?: string;
    techStack?: string[];
  };
}
```

## 出力スキーマ

```typescript
interface TypePatternResult {
  recommendedPattern: {
    name: string;
    reason: string;
    implementation: string;
    tradeoffs: string[];
  };
  alternativePatterns: {
    name: string;
    reason: string;
    tradeoffs: string[];
  }[];
  claudePromptSuggestion: string;
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 7. コーチング/ティーチング判断

### なぜ必要か

Claudeに質問する際、「知識を教えてほしい」のか「考えを整理するのを手伝ってほしい」のかで、効果的な依頼の仕方が異なります。

- **ティーチングモード**: Claudeから知識を学ぶ(Claude → 人間)
- **コーチングモード**: 対話で自分の考えを整理する(人間 ⇄ Claude)

このツールは、あなたの発言から適切なモードを判定し、そのモードに最適化されたプロンプトテンプレートを提案します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 自分が「何を求めているか」が明確になります
- Claudeとの対話がより効率的になります
- 学習効果が向上します(適切なモードで適切な質問ができる)

### 前提条件

- Claudeに質問や相談をしようとしていること
- 自分の状況を言語化できること

### 使い方

1. 自分の発言(「〇〇がわからない」「〇〇で迷っている」など)を入力します
2. 推奨されるモード(ティーチング/コーチング)を確認します
3. 提案されたプロンプトテンプレートをカスタマイズしてClaudeに送信します

### 期待される出力

**入力例(ティーチング向き):**
```json
{
  "userStatement": "TypeScriptのジェネリクスがわからない"
}
```

**出力例:**
```json
{
  "recommendedMode": "teaching",
  "confidence": 0.9,
  "detectedTriggers": {
    "teaching": [{ "keyword": "わからない", "weight": 0.9 }],
    "coaching": []
  },
  "samplePrompt": "TypeScriptのジェネリクスについて教えてください。\n\n以下の形式でお願いします:\n1. 一言で説明(初心者向け)\n2. なぜ必要なのか(動機)\n3. 具体的なコード例\n4. よくある間違い\n5. 私の理解度を確認する質問を3つ",
  "wrongModeWarning": null
}
```

**入力例(コーチング向き):**
```json
{
  "userStatement": "マイクロサービスにすべきかモノリスで続けるか迷っている"
}
```

**出力例:**
```json
{
  "recommendedMode": "coaching",
  "confidence": 0.85,
  "detectedTriggers": {
    "teaching": [],
    "coaching": [
      { "keyword": "迷っている", "weight": 0.9 },
      { "keyword": "すべきか", "weight": 0.7 }
    ]
  },
  "samplePrompt": "アーキテクチャの選択について壁打ちさせてください。\n\n現状の考え:\n・マイクロサービスとモノリスのどちらにすべきか決められない\n\n迷っている点:\n・(ここに具体的な迷いを追記してください)\n\nお願い:\n・私の考えを整理する質問をしてください\n・選択肢とトレードオフを提示してください\n・判断は私が行います",
  "wrongModeWarning": null
}
```

### トラブルシューティング

<details>
<summary>両方のモードが同程度にマッチする場合</summary>

**症状:** confidence が低く、両方のトリガーが検出される

**対処法:**
1. 自分が「何を求めているか」を再度考えてみてください
2. 「答えがほしい」→ ティーチング、「整理したい」→ コーチング
3. 迷う場合は、コーチングモードで始めて、必要に応じてティーチングに切り替えてください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様でコーチング/ティーチング判断ツールを実装してください。

## 機能概要

ユーザーの発言から適切な学習モードを判定し、サンプルプロンプトを提案します。

## 背景

- ティーチングモード: Claudeから知識を学ぶ(Claude → 人間)
- コーチングモード: 対話で自分の考えを整理する(人間 ⇄ Claude)

## 入力スキーマ

```typescript
interface ModeDetectionInput {
  userStatement: string;
}
```

## 判定ルール

### ティーチングモード トリガー

| カテゴリ | キーワード | 重み |
|---------|-----------|------|
| 知識不足系 | 「わからない」「知らない」「初めて」「未経験」「教えて」「説明して」「解説して」 | 0.9 |
| スキル不足系 | 「方法は」「やり方」「どうやって」「書き方」「使い方」「ベストプラクティス」「パターン」 | 0.8 |
| 確認系 | 「合ってる?」「正しい?」「これでいい?」「間違ってない?」 | 0.6 |

### コーチングモード トリガー

| カテゴリ | キーワード | 重み |
|---------|-----------|------|
| 判断・決定系 | 「迷っている」「どちらがいい」「選べない」「判断できない」「決められない」「優先度」 | 0.9 |
| 思考整理系 | 「壁打ち」「整理したい」「まとまらない」「考えを聞いて」「相談」「話を聞いて」 | 0.8 |
| 言語化系 | 「説明できない」「言語化したい」「理由を整理」「根拠が曖昧」 | 0.7 |

## 出力スキーマ

```typescript
interface ModeDetectionResult {
  recommendedMode: 'teaching' | 'coaching';
  confidence: number;
  detectedTriggers: {
    teaching: { keyword: string; weight: number }[];
    coaching: { keyword: string; weight: number }[];
  };
  samplePrompt: string;
  wrongModeWarning?: string;
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 8. CLAUDE.md テンプレート生成

### なぜ必要か

CLAUDE.mdはプロジェクトの成熟度に応じて「育てる」ものです。しかし、各フェーズで「何を書くべきか」は明確ではありません。

このツールは、プロジェクトのPhase(0〜4)に応じた最適なCLAUDE.mdテンプレートを生成します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「今のフェーズで何を書くべきか」が明確になります
- 過不足のないCLAUDE.mdを作成できます
- 次のフェーズへの移行指針も得られます

### 前提条件

- プロジェクトの現在のフェーズを把握していること
- 基本的なプロジェクト情報(名前、目的など)があること

### Phase定義

| Phase | 名称 | 文字数目安 | 主な内容 |
|-------|------|-----------|---------|
| 0 | 初期 | 300-500 | プロジェクト名・目的・技術スタック候補 |
| 1 | 設計中 | 500-1,000 | 機能一覧・用語集・ドキュメント構成 |
| 2 | 開発開始 | 1,000-1,500 | 技術スタック確定・アーキテクチャ・ディレクトリ |
| 3 | 開発中 | 1,500-2,500 | コマンド・コーディングルール・型ルール・禁止事項 |
| 4 | 運用中 | 2,000-3,000 | 運用知見・トラブルシューティング |

### Claudeへの依頼プロンプト

```markdown
以下の仕様でCLAUDE.mdテンプレート生成ツールを実装してください。

## 機能概要

プロジェクトのPhaseに応じたCLAUDE.mdテンプレートを生成します。

## 入力スキーマ

```typescript
interface ClaudeMdTemplateInput {
  phase: 0 | 1 | 2 | 3 | 4;
  projectInfo: {
    name: string;
    description?: string;
    purpose?: string;
    techStack?: string[];
    features?: string[];
    glossary?: { term: string; definition: string }[];
    architecture?: string;
    commands?: { name: string; command: string; description: string }[];
    codingRules?: string[];
    prohibitions?: string[];
  };
}
```

## 出力スキーマ

```typescript
interface ClaudeMdTemplateResult {
  content: string;
  estimatedCharCount: number;
  sections: string[];
  nextPhaseHints: string[];
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 9. CLAUDE.md Phase推定

### なぜ必要か

既存のCLAUDE.mdがある場合、「今どのPhaseにいるか」を把握することで、次に何を追記すべきかが明確になります。

このツールは、CLAUDE.mdの内容を分析し、現在のPhaseを推定します。

### Claudeへの依頼プロンプト

```markdown
以下の仕様でCLAUDE.md Phase推定ツールを実装してください。

## 機能概要

既存のCLAUDE.mdの内容を分析し、現在のPhaseを推定します。

## 判定基準

- 文字数
- 含まれるセクション(Commands, Tech Stack, Coding Rules等)
- 情報の詳細度

## 入力スキーマ

```typescript
interface ClaudeMdAnalysisInput {
  content: string;
}
```

## 出力スキーマ

```typescript
interface ClaudeMdAnalysisResult {
  estimatedPhase: 0 | 1 | 2 | 3 | 4;
  confidence: number;
  detectedSections: string[];
  missingForNextPhase: string[];
  recommendations: string[];
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 10. 習熟度レベル判定

### なぜ必要か

AI協働開発のスキルは段階的に身につきます。自分が今どのレベルにいるかを把握することで、次に学ぶべきことが明確になります。

### レベル定義

| レベル | 名称 | 主なスキル |
|-------|------|-----------|
| L1 | 基礎 | CLAUDE.mdを作成できる、1文1検証でタスクを渡せる、検証ループを回せる |
| L2 | 実践 | Type分類を使い分けられる、損切り判断ができる、失敗をCLAUDE.mdに記録 |
| L3 | 最適化 | 並列作業ができる、チームにガイドを展開できる、効果測定ができる |

### Claudeへの依頼プロンプト

```markdown
以下の仕様で習熟度レベル判定ツールを実装してください。

## 機能概要

チェックリストへの回答からAI協働開発の習熟度レベルを判定します。

## 入力スキーマ

```typescript
interface ProficiencyCheckInput {
  answers: {
    // Level 1 チェック項目
    canCreateClaudeMd: boolean;
    canDecomposeTask: boolean;
    canRunVerificationLoop: boolean;
    understandsTypeClassification: boolean;
    
    // Level 2 チェック項目
    canClassifyType1Or2: boolean;
    canMakeCutLossDecision: boolean;
    recordsFailuresToClaudeMd: boolean;
    usesCoachingAndTeaching: boolean;
    
    // Level 3 チェック項目
    canParallelizeTasks: boolean;
    canGuideTeamMembers: boolean;
    canMeasureEffectiveness: boolean;
    contributesToClaudeMdAsTeam: boolean;
  };
}
```

## 出力スキーマ

```typescript
interface ProficiencyCheckResult {
  currentLevel: 1 | 2 | 3;
  levelName: string;
  score: { level1: number; level2: number; level3: number };
  nextLevelRequirements: string[];
  recommendedPractice: string[];
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 11. INVEST原則チェック

### なぜ必要か

INVEST原則は、ユーザーストーリーやタスクの品質を評価するためのフレームワークです。AI協働開発では、特に「Small」と「Testable」が重要になります。

### INVEST原則

| 原則 | 意味 | AI協働での重要度 |
|-----|------|-----------------|
| I: Independent | 他タスクに依存せず単独で完了できる | ★★☆ |
| N: Negotiable | 詳細は実装時に調整できる | ★☆☆ |
| V: Valuable | 完了すると何らかの価値を提供する | ★★☆ |
| E: Estimable | 完了までの時間を概算できる | ★★☆ |
| S: Small | 1セッションで処理可能 | ★★★ |
| T: Testable | 受け入れ基準を明文化できる | ★★★ |

### Claudeへの依頼プロンプト

```markdown
以下の仕様でINVEST原則チェックツールを実装してください。

## 機能概要

タスク記述がINVEST原則を満たすかチェックします(特にSmall, Testableを重視)。

## 入力スキーマ

```typescript
interface InvestCheckInput {
  taskDescription: string;
  estimatedHours?: number;
  acceptanceCriteria?: string[];
}
```

## 出力スキーマ

```typescript
interface InvestCheckResult {
  overall: 'pass' | 'warning' | 'fail';
  checks: {
    independent: { pass: boolean; reason: string };
    negotiable: { pass: boolean; reason: string };
    valuable: { pass: boolean; reason: string };
    estimable: { pass: boolean; reason: string };
    small: { pass: boolean; reason: string };
    testable: { pass: boolean; reason: string };
  };
  suggestions: string[];
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## Phase 3: 最適化ツール — 効率を高める

基礎と実践のツールを使いこなせるようになったら、さらなる効率化を目指します。並列作業やコンテキスト管理を最適化することで、開発速度を向上させます。

---

## 12. 並列化判断

### なぜ必要か

複数のタスクを同時に進められれば、開発速度は大幅に向上します。ただし、並列化できるタスクとできないタスクがあります。

このツールは、複数のタスク間の依存関係を分析し、並列実行可能なグループを特定します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 並列実行可能なタスクを客観的に判断できます
- 依存関係によるデッドロックを事前に防げます
- Claude Codeの複数インスタンス活用が効率化されます

### 前提条件

- 複数のタスクがリストアップされていること
- 各タスクの影響範囲(変更ファイル)が把握されていること

### 使い方

1. 複数のタスク情報(影響ファイル、入出力、推定時間)を入力します
2. 並列実行可能なグループを確認します
3. 順次実行が必要な場合は、提示された順序に従います

### 判定基準

| 条件 | 並列可否 | 理由 |
|------|---------|------|
| 同じファイルを編集 | ❌ 不可 | マージコンフリクトのリスク |
| 片方の出力が他方の入力 | ❌ 不可 | 依存関係あり |
| どちらも10分以上 | ✅ 推奨 | 待ち時間を有効活用 |
| 影響範囲が独立 | ✅ 可能 | 相互干渉なし |

### 期待される出力

**入力例:**
```json
{
  "tasks": [
    {
      "id": "task-1",
      "description": "Userエンティティを作成",
      "affectedFiles": ["src/entities/User.ts"],
      "inputs": [],
      "outputs": ["User型"],
      "estimatedMinutes": 15
    },
    {
      "id": "task-2",
      "description": "Postエンティティを作成",
      "affectedFiles": ["src/entities/Post.ts"],
      "inputs": ["User型"],
      "outputs": ["Post型"],
      "estimatedMinutes": 15
    },
    {
      "id": "task-3",
      "description": "Commentエンティティを作成",
      "affectedFiles": ["src/entities/Comment.ts"],
      "inputs": [],
      "outputs": ["Comment型"],
      "estimatedMinutes": 10
    }
  ]
}
```

**出力例:**
```json
{
  "canParallelize": true,
  "reason": "task-1とtask-3は並列実行可能です。task-2はtask-1の完了後に実行してください。",
  "parallelGroups": [
    ["task-1", "task-3"]
  ],
  "sequentialOrder": ["task-1", "task-3", "task-2"],
  "efficiency": {
    "sequentialMinutes": 40,
    "parallelMinutes": 25,
    "timeSavedMinutes": 15
  }
}
```

### トラブルシューティング

<details>
<summary>すべてのタスクが順次実行と判定される場合</summary>

**症状:** 並列グループが空で、すべてが順次実行

**原因:** タスク間の依存関係が過剰に設定されている可能性があります

**対処法:**
1. 各タスクの `inputs` と `outputs` を見直してください
2. 本当に必要な依存関係のみを記載してください
3. インターフェースを先に定義し、実装を分離することで並列化できる場合があります
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様で並列化判断ツールを実装してください。

## 機能概要

複数のタスク間の依存関係を分析し、並列実行可能なグループを特定します。

## 入力スキーマ

```typescript
interface ParallelizationInput {
  tasks: {
    id: string;
    description: string;
    affectedFiles: string[];
    inputs: string[];
    outputs: string[];
    estimatedMinutes: number;
  }[];
}
```

## 出力スキーマ

```typescript
interface ParallelizationResult {
  canParallelize: boolean;
  reason: string;
  parallelGroups: string[][];  // 並列実行可能なタスクIDのグループ
  sequentialOrder: string[];   // 順次実行が必要な場合の順序
  efficiency?: {
    sequentialMinutes: number;
    parallelMinutes: number;
    timeSavedMinutes: number;
  };
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 13. 新会話開始判断

### なぜ必要か

Claudeとの会話が長くなると、コンテキストが汚染され、パフォーマンスが低下することがあります。適切なタイミングで新しい会話を開始することで、品質を維持できます。

このツールは、現在の会話状態から「継続」「検討」「新規開始」を判定し、新規開始時の事前準備を提示します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「会話を続けるべきか新規にすべきか」の判断が客観的にできます
- コンテキスト汚染による品質低下を防げます
- 不要な会話リセットによる情報ロスを避けられます

### 前提条件

- 現在の会話状態(メッセージ数、扱っているモジュール等)を把握していること

### 使い方

1. 現在の会話状態を入力します
2. 判定結果(継続/検討/新規開始)を確認します
3. 「新規開始」の場合、提示された事前準備を行います

### 判定基準

| 条件 | 判定 | 理由 |
|-----|------|------|
| 別のモジュールの作業に移る | 新規会話 | コンテキストの混乱を防ぐ |
| パフォーマンス低下を感じる | 新規会話 | コンテキスト長の影響 |
| Claudeが矛盾する回答をする | 新規会話 | コンテキスト汚染のサイン |
| 同じ機能の実装を継続中 | 継続 | コンテキストを活用 |
| エラー修正のイテレーション中 | 継続 | 直前の文脈が必要 |

### 期待される出力

**入力例:**
```json
{
  "currentContext": {
    "messageCount": 45,
    "currentModule": "src/auth/",
    "previousModule": "src/payment/",
    "isInErrorLoop": false,
    "hasContradictoryResponses": true,
    "perceivedPerformance": "degraded"
  }
}
```

**出力例:**
```json
{
  "recommendation": "start_new",
  "reason": "モジュール変更と矛盾する回答が検出されました。コンテキストをリセットすることを推奨します。",
  "beforeStartingNew": [
    "現在の会話で得た情報をCLAUDE.mdに記録してください",
    "未完了のタスクをリストアップしてください",
    "新しい会話でCLAUDE.mdを参照させてください"
  ],
  "indicators": {
    "moduleChange": true,
    "contradictions": true,
    "performanceDegraded": true,
    "messageCountHigh": true
  }
}
```

### トラブルシューティング

<details>
<summary>頻繁に「新規開始」と判定される場合</summary>

**症状:** 短い会話でも新規開始が推奨される

**原因:** タスクの粒度が大きすぎるか、モジュールを頻繁に跨いでいる可能性

**対処法:**
1. タスクを「1文1検証」粒度に分解してください
2. 1つの会話では1つのモジュールに集中してください
3. 複数モジュールにまたがる場合は、先に計画を立ててから各モジュールを別会話で対応してください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様で新会話開始判断ツールを実装してください。

## 機能概要

現在の会話状態から「継続」「検討」「新規開始」を判定し、新規開始時の事前準備を提示します。

## 入力スキーマ

```typescript
interface ConversationCheckInput {
  currentContext: {
    messageCount: number;
    currentModule: string;
    previousModule?: string;
    isInErrorLoop: boolean;
    hasContradictoryResponses: boolean;
    perceivedPerformance: 'good' | 'degraded' | 'poor';
  };
}
```

## 出力スキーマ

```typescript
interface ConversationCheckResult {
  recommendation: 'continue' | 'consider_new' | 'start_new';
  reason: string;
  beforeStartingNew: string[];  // 新規開始前にすべきこと
  indicators?: {
    moduleChange: boolean;
    contradictions: boolean;
    performanceDegraded: boolean;
    messageCountHigh: boolean;
  };
}
```

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 14. 階層判断フロー

### なぜ必要か

フレームワークの4つの哲学(持続可能性、心理的安全性、急がば回れ、シンプルさ)は階層構造を持ち、上位Levelが常に優先されます。このツールは、判断が必要な場面で階層に沿った一貫した判断を支援します。

### 期待される効果

このツールを使うことで、以下の変化が期待できます。

- 「どちらを優先するか」という競合判断が不要になります
- 判断が階層に沿って一意に決まり、チームで一貫した基準を共有できます
- 判断の根拠が明確になり、後から振り返りができます

### 前提条件

- 判断が必要な状況を把握していること
- 哲学の階層構造(L0〜L3)を理解していること

### 使い方

1. 判断が必要な状況を入力します
2. L0からL3まで順にチェックを行います
3. 各Levelの判断結果と推奨アクションを確認します

### 階層の4つのチェック

| Level | 哲学 | 問いかけ | Noの場合 |
|-------|------|---------|----------|
| L0 | 持続可能性 | 1年後のチームにプラスか? | その選択肢は却下 |
| L1 | 心理的安全性 | 誰かが萎縮しないか? | 萎縮しない方法を検討 |
| L2 | 急がば回れ | 丁寧さを保てるか? | 丁寧にできる範囲に縮小 |
| L3 | シンプルさ | 1文1検証を満たせるか? | よりシンプルな方法を選択 |

### 期待される出力

**入力例:**
```json
{
  "situation": "新機能開発中に、ドキュメントを詳細に書くか最小限にするか迷っている",
  "context": "production"
}
```

**出力例:**
```json
{
  "levelChecks": {
    "L0_sustainability": {
      "question": "ドキュメントは1年後のチームにプラスか?",
      "answer": true,
      "reasoning": "将来のメンテナンス・引き継ぎに必要"
    },
    "L1_psychologicalSafety": {
      "question": "ドキュメント作成で誰かが萎縮しないか?",
      "answer": true,
      "reasoning": "ドキュメント作成は特に萎縮を招かない"
    },
    "L2_goSlowToGoFast": {
      "question": "丁寧にドキュメントを書く余裕があるか?",
      "answer": true,
      "reasoning": "本番開発では丁寧さを優先すべき"
    },
    "L3_simplicity": {
      "question": "シンプルに書けるか?",
      "answer": true,
      "reasoning": "要点のみを簡潔に記載する"
    }
  },
  "recommendation": "ドキュメントを書く。ただし要点のみを簡潔に記載し、詳細は必要に応じて追記する。",
  "reasoning": "L0(持続可能性)を満たすためにドキュメントは必要。L3(シンプルさ)の制約内で、最小限かつ要点を押さえた内容にする。"
}
```

### トラブルシューティング

<details>
<summary>L0で「No」になった場合</summary>

**症状:** 1年後のチームにプラスにならない選択肢を検討している

**対処法:**
1. その選択肢は却下してください
2. 別の選択肢を検討してください
3. 「なぜこの選択肢を検討していたか」を振り返り、根本的な問題を再定義してください
</details>

<details>
<summary>L1で「No」になった場合</summary>

**症状:** 誰かが萎縮する恐れがある判断をしようとしている

**対処法:**
1. 萎縮を招かない形にアプローチを修正してください
2. 「撤退」を「戦略的再構成」としてフレーミングするなど、伝え方を工夫してください
3. 必要に応じてチームメンバーと相談してください
</details>

<details>
<summary>L2で「No」になった場合</summary>

**症状:** 丁寧さを保てない状況にある

**対処法:**
1. スコープを縮小し、丁寧にできる範囲に絞ってください
2. 緊急度を再評価し、本当に急ぐ必要があるか確認してください
3. 必要なら後で補完することを前提に、最小限の対応にしてください
</details>

### Claudeへの依頼プロンプト

```markdown
以下の仕様で階層判断フローツールを実装してください。

## 機能概要

哲学の階層構造(L0〜L3)に基づいて、判断が必要な場面での意思決定を支援します。

## 背景

哲学の階層構造:
- L0 持続可能性: 1年後のチームにプラスか?
- L1 心理的安全性: 誰かが萎縮しないか?
- L2 急がば回れ: 丁寧さを保てるか?
- L3 シンプルさ: 1文1検証を満たせるか?

上位Levelが常に優先され、下位Levelは上位の制約内で最適化されます。

## 入力スキーマ

```typescript
interface HierarchyCheckInput {
  situation: string;
  context: 'learning' | 'production' | 'teamOnboarding' | 'deadline';
}
```

## 出力スキーマ

```typescript
interface LevelCheckResult {
  question: string;
  answer: boolean;
  reasoning: string;
}

interface HierarchyCheckResult {
  levelChecks: {
    L0_sustainability: LevelCheckResult;
    L1_psychologicalSafety: LevelCheckResult;
    L2_goSlowToGoFast: LevelCheckResult;
    L3_simplicity: LevelCheckResult;
  };
  recommendation: string;
  reasoning: string;
}
```

## 判断ロジック

1. L0からL3まで順にチェックする
2. いずれかのLevelで「No」になった場合、そのLevelの対処法を提示する
3. すべて「Yes」の場合、実行可能と判断する

## 技術スタック

TypeScript + zod

## 検証方法

```bash
npx tsc --noEmit
```
```

---

## 統合ツールの提案

個別ツールを組み合わせた統合フローも構築できます。

### タスク実行支援フロー

```
入力: タスク記述

1. 1文1検証チェッカー
   → ❌ → タスク分解支援へ
   → ✅ ↓

2. Type 1/2 分類判定
   → Type 1: そのまま実行
   → Type 2: 必要なコンテキストを提示

3. 実行中...

4. エラー発生時: 損切り判断
   → 継続: エラー対応
   → 撤退: 損切り後復帰ナビへ

5. 完了時: CLAUDE.mdへの記録提案
```

---

## ツール実装の技術スタック推奨

| 用途 | 推奨スタック | 理由 |
|------|-------------|------|
| CLI | TypeScript + Commander + Inquirer | 対話的な入力が可能 |
| ライブラリ | TypeScript + zod + tsup | 型安全 + バンドル |
| MCP Server | TypeScript + @modelcontextprotocol/sdk | Claude連携 |
| VSCode拡張 | TypeScript + vscode API | IDE統合 |
| Web UI | React + TypeScript | アーティファクト対応 |

---

## 15. 可逆性チェック

### なぜ必要か

AI協働開発が進むと、気づかないうちにAIへの依存度が高まることがあります。「AIなしでは何もできない」状態に陥ると、AIの出力を適切に評価できなくなり、問題発生時に対処できなくなります。

このツールは、定期的に可逆性(R1-R3原則)を確認し、健全な協働関係を維持するために使用します。

### 期待される効果

- AIへの依存度を客観的に把握できる
- 「気づいたら依存していた」を防げる
- チーム全体の健全性を定期的に確認できる

### 前提条件

- Part 0「可逆性の3つの原則(R1-R3)」を理解していること
- 直近1ヶ月程度のAI協働開発の経験があること

### 判断フロー

```
┌─────────────────────────────────────────────────────────────────┐
│                    可逆性チェックフロー                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  R1: スキルの可逆性                                             │
│  ─────────────────                                              │
│  Q1: 直近1ヶ月で、AIなしでコードを書いたか?                    │
│      Yes → Q2へ                                                 │
│      No  → ⚠️ 月1回は手動実装を推奨                             │
│                                                                 │
│  Q2: AIが生成したコードを「なぜそうなるか」説明できるか?       │
│      Yes → R2へ                                                 │
│      No  → ⚠️ ティーチングモードで理解を深める                  │
│                                                                 │
│  R2: プロセスの可逆性                                           │
│  ─────────────────                                              │
│  Q3: CLAUDE.mdがなくても開発を継続できるか?                    │
│      Yes → Q4へ                                                 │
│      No  → ⚠️ フォールバック手順を整備                          │
│                                                                 │
│  Q4: チームの全員がAIなしで作業できるか?                       │
│      Yes → R3へ                                                 │
│      No  → ⚠️ 習熟度の低いメンバーをサポート                    │
│                                                                 │
│  R3: 判断の可逆性                                               │
│  ─────────────────                                              │
│  Q5: 直近1週間で、AIの提案を却下したことがあるか?              │
│      Yes → Q6へ                                                 │
│      No  → ⚠️ 批判的評価の習慣を確認                            │
│                                                                 │
│  Q6: 却下の理由を明確に説明できるか?                           │
│      Yes → ✅ 可逆性は健全                                      │
│      No  → ⚠️ 判断基準を言語化する                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Claudeへの依頼プロンプト

```
以下のチェックリストで、AI協働開発の可逆性を確認してください。

【R1: スキルの可逆性】
□ 直近1ヶ月で、AIなしでコードを書いた
□ AIが生成したコードの「なぜ」を説明できる
□ 新しい技術を学ぶとき、まず基礎を理解してからAIを使う

【R2: プロセスの可逆性】
□ CLAUDE.mdがなくても開発手順は成立する
□ チームの全員がAIなしで作業できる
□ 緊急時のフォールバック手順がある

【R3: 判断の可逆性】
□ AIの提案を「選択肢の一つ」として評価している
□ 却下する際の理由を明確に持っている
□ 「AIが言ったから」を最終判断理由にしていない

各項目について、現状と改善点を教えてください。
```

### 期待される出力

```
【可逆性チェック結果】

■ R1: スキルの可逆性
  状態: ⚠️ 要注意
  - 直近1ヶ月でAIなしのコードは書いていない
  - コードの説明は概ねできるが、深い部分は曖昧
  推奨アクション:
  - 週1回、30分程度の手動コーディング時間を設ける
  - ティーチングモードで「なぜこの実装か」を確認する習慣

■ R2: プロセスの可逆性
  状態: ✅ 健全
  - CLAUDE.mdなしでも開発手順は整備済み
  - チームメンバー全員がAIなしで基本作業可能
  推奨アクション:
  - 現状維持

■ R3: 判断の可逆性
  状態: ✅ 健全
  - 今週3回、AIの提案を却下(理由: 既存パターンとの不整合)
  - 判断基準はCLAUDE.mdに記録済み
  推奨アクション:
  - 現状維持

【総合評価】
可逆性スコア: 2/3(R1に改善の余地あり)
次回チェック推奨: 1ヶ月後
```

### 使用タイミング

| タイミング | 目的 |
|-----------|------|
| **月次振り返り時** | Layer 3の振り返りに組み込む |
| **新メンバー参加時** | チームの可逆性を確認 |
| **依存度の懸念を感じたとき** | 随時実施 |
| **AI活用が急増したとき** | 依存度の急激な変化を検知 |

### トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| すべての項目が⚠️ | AI依存が進行している | 意図的な「AIなし期間」を設ける |
| R1だけ⚠️ | スキル維持の機会が少ない | 手動実装の時間を確保 |
| R3だけ⚠️ | 批判的評価の習慣がない | AIの提案を「仮説」として扱う習慣 |

---

## 次のステップ

この付録で紹介したツールは、すべてを一度に導入する必要はありません。まずは日常的に使用頻度が高い以下の3つから始めることをお勧めします。

1. **1文1検証チェッカー** — タスクを渡す前のセルフチェックに
2. **Type 1/2 分類判定** — 必要なコンテキストの判断に
3. **損切り判断** — 行き詰まりからの脱出に

これらのツールを使いこなせるようになったら、[Part 4: 成長とチーム展開](./04-growth-adoption.md) を参考に、チームへの展開を検討してください。

---

## 付録D-1: 技術スタック別の対応表

このフレームワークの例示はTypeScript/NestJS/zodに偏っていますが、**原則は言語・フレームワークに依存しません**。
以下の対応表を参考に、自分の技術スタックに読み替えてください。

### 検証コマンドの対応表

| 概念 | TypeScript | Python | C/C++ | Rust | Go |
|------|-----------|--------|-------|------|-----|
| **型チェック** | `npx tsc --noEmit` | `mypy .` | `make` (コンパイル) | `cargo check` | `go build` |
| **リント** | `npx eslint .` | `ruff check .` | `clang-tidy` | `cargo clippy` | `golangci-lint` |
| **テスト** | `npx vitest run` | `pytest` | `ctest` / `make test` | `cargo test` | `go test ./...` |
| **フォーマット** | `npx prettier --check .` | `black --check .` | `clang-format` | `cargo fmt --check` | `gofmt -l .` |

### 型定義の対応表

| 概念 | TypeScript | Python | C/C++ | Rust |
|------|-----------|--------|-------|------|
| **エンティティ定義** | `interface User` | `@dataclass User` | `struct User` | `struct User` |
| **バリデーション** | zod スキーマ | Pydantic モデル | 手動チェック / 専用ライブラリ | serde + validator |
| **Branded Type** | `type UserId = string & { __brand: 'UserId' }` | `NewType('UserId', str)` | `typedef` + コメント | newtype パターン |
| **Union型** | `type Role = 'admin' \| 'user'` | `Literal['admin', 'user']` | `enum class Role` | `enum Role` |

### タスク記述の言語別例

**元の例(TypeScript):**
```
「zodでUserスキーマを作成してください」
```

**Python版:**
```
「PydanticでUserモデルを作成してください」
```

**C/C++版:**
```
「User構造体のヘッダーファイルを作成してください」
```

**Rust版:**
```
「User構造体をderiveマクロ付きで作成してください」
```

### 1文1検証の言語別適用

**原則は同じ**:「1文で説明でき、検証1回で完了確認できる」

| 言語 | 良い例 | 悪い例 |
|------|--------|--------|
| **TypeScript** | 「Userインターフェースを作成して、tscを実行」 | 「User作成、API実装、テストも」 |
| **Python** | 「Userデータクラスを作成して、mypyを実行」 | 「User作成、エンドポイント実装、pytest」 |
| **C/C++** | 「User.hを作成して、コンパイルを確認」 | 「ヘッダー作成、実装、Makefileも」 |
| **Rust** | 「User構造体を作成して、cargo checkを実行」 | 「構造体作成、impl追加、テストも」 |

> 💡 **重要**: 言語が変わっても、**「小さく、検証可能に、明確に」** という原則は不変です。
> 検証コマンドが異なるだけで、考え方は同じです。

---

## 付録D-2: 他のAIツールへの適用 — GitHub Copilotとの併用

このフレームワークの原則は**Claude専用ではありません**。
GitHub Copilot、Cursor、その他のAIコーディングツールにも適用できます。

### ツール特性の比較

```
┌─────────────────────────────────────────────────────────────────┐
│                AIコーディングツールの特性比較                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  観点           GitHub Copilot      Claude Code                │
│  ────           ──────────────      ───────────                │
│  得意領域       行単位の補完         対話型タスク               │
│                 リアルタイム提案     複数ファイル操作           │
│                                                                 │
│  コンテキスト   現在のファイル中心   会話履歴 + CLAUDE.md       │
│                 周辺ファイル参照     明示的なコンテキスト渡し   │
│                                                                 │
│  インタラクション Tab/Enter で確定   対話で詳細を詰める         │
│                   即座のフィードバック 検証ループを回す         │
│                                                                 │
│  適したタスク   小さな補完           設計を含む実装             │
│                 ボイラープレート     リファクタリング           │
│                 テストケース追加     ドキュメント生成           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 本フレームワークの原則の適用範囲

| 原則 | GitHub Copilot | Claude Code | 備考 |
|------|---------------|-------------|------|
| **1文1検証** | ✅ 適用可 | ✅ 適用可 | 補完を受け入れる単位を小さく |
| **Type 1/2 分類** | △ 部分適用 | ✅ 適用可 | Copilotは暗黙的にコンテキスト参照 |
| **損切り判断** | ✅ 適用可 | ✅ 適用可 | 補完が的外れなら別アプローチ |
| **CLAUDE.md** | △ 代替手段 | ✅ 適用可 | Copilotは `.github/copilot-instructions.md` |
| **検証ループ** | ✅ 適用可 | ✅ 適用可 | 補完後も必ず検証 |

### 併用のベストプラクティス

```
┌─────────────────────────────────────────────────────────────────┐
│                    併用のパターン                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  【パターン1: フェーズで使い分け】                               │
│  ─────────────────────────────────                              │
│  設計・要件整理      → Claude(コーチングモード)              │
│  実装・コーディング  → Copilot(リアルタイム補完)             │
│  レビュー・リファクタ → Claude(対話で詳細を詰める)           │
│                                                                 │
│  【パターン2: タスクサイズで使い分け】                           │
│  ─────────────────────────────────────                          │
│  1行〜数行の補完     → Copilot                                  │
│  関数・クラス単位    → Claude                                   │
│  複数ファイル変更    → Claude                                   │
│                                                                 │
│  【パターン3: 確信度で使い分け】                                 │
│  ───────────────────────────────                                │
│  何を書くか明確      → Copilot(手が止まらない)               │
│  何を書くか不明確    → Claude(対話で整理)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 共通の注意点

**どのツールを使っても守るべき原則**:

1. **検証は必須** — AIの出力は必ず検証する(ツールに依存しない)
2. **小さく渡す** — 大きなタスクは分解してから渡す
3. **明確に伝える** — 曖昧な指示は避ける
4. **記録する** — うまくいったパターン、失敗パターンを記録する
5. **判断は人間** — 最終判断の責任は常に人間

> 💡 **要点**: ツールは変わっても、**「引き出す責任は人間にある」** という原則は不変です。
> このフレームワークで学んだ考え方は、どのAIツールにも応用できます。

---

## 変更履歴

- v3.2 (2026-02-02): 技術スタック非依存の対応表追加
  - 付録D-1「技術スタック別の対応表」を新規追加
  - 付録D-2「他のAIツールへの適用」を新規追加
- v3.1 (2026-02-01): HITL調査に基づく強化
  - ツール#15「可逆性チェック」を新規追加(R1-R3原則の定期確認用)
- v3.0 (2026-02-01): 哲学構造の階層化対応
  - ツール#14「哲学競合時の判断」を「階層判断フロー」に全面書き換え
  - 複雑度を「中→低」に変更(競合判断が不要になったため)
- v2.1 (2026-01-31): レビュー指摘対応
  - #005: ツール#4「損切り判断」にアプローチ選択ロジック(フローチャート + 優先順位表)を追加
  - #011: ツール#12-#14の構造を統一(期待される効果、期待される出力、トラブルシューティングを追加)
- v2.0 (2026-01-31): DigitalOcean + DevOpsハイブリッド形式で全面改訂
- v1.0 (2026-01-31): 初版作成
