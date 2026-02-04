# 付録A-2: ウォークスルー例 - CSVデータ変換パイプライン

> **📖 ドキュメント種類**: Tutorial（学習志向・行動）
>
> **この付録について**: CSVファイルを読み込み、変換・検証・出力する
> パイプライン処理を題材に、Type 2タスク（既存コードの変更）と
> エラーハンドリングの実践を学びます。
>
> **読むタイミング**: 
> - 付録A-1を完了した後のステップアップとして
> - データ処理系のタスクに取り組む前の参考として
>
> **前提**: 
> - [付録A-1: ログイン機能](./appendix-a-walkthrough.md) を完了していること
> - Node.js + TypeScript の基礎知識
> - zod によるバリデーションの基礎
>
> **所要時間**: 約45-60分
>
> **難易度**: ⭐⭐ 中級（Type 2タスクの連鎖、Result型の活用）

### このチュートリアルで学べること

| # | 学習内容 | 関連する原則 |
|---|---------|-------------|
| 1 | Type 2タスク（既存コード変更）の進め方 | H1: 設計→実装→検証 |
| 2 | Result型によるエラーハンドリング | A3: 検証可能な出力 |
| 3 | パイプライン処理の段階的な実装 | S1: 1文1検証 |
| 4 | 損切り判断の実践 | P2: 損切りは判断 |

---

## シナリオ

あなたは既存の売上管理システムに、CSVインポート機能を追加することになりました。

**要件:**
- CSVファイルを読み込み、売上データとして登録する
- 不正なデータはスキップし、エラーログを出力する
- 処理結果のサマリーを返す

**既存コード:**
- `src/domain/entities/Sale.ts` - 売上エンティティ（既存）
- `src/infrastructure/repositories/SaleRepository.ts` - 売上リポジトリ（既存）

---

## Step 1 — タスクの分解

### 分解のポイント

この機能は**複数のType 2タスク（既存コードの変更）** を含みます。
Type 2タスクは既存コードとの整合性を確認しながら進める必要があるため、
より慎重な分解が求められます。

### 分解結果

```
大タスク: CSVインポート機能の実装

分解結果:
1. CSVパーサーの作成（Type 1: 新規）
   - 完了条件: typecheck + test通過
   - 所要時間目安: 15分

2. バリデーション層の作成（Type 1: 新規）
   - 完了条件: typecheck + test通過
   - 所要時間目安: 15分

3. インポートユースケースの実装（Type 2: 既存連携）
   - 完了条件: typecheck + lint + test通過
   - 所要時間目安: 20分

4. エラーハンドリングとサマリー出力（Type 2: 既存連携）
   - 完了条件: 全検証通過
   - 所要時間目安: 15分

合計: 約65分
```

### 分解の検証

| チェック項目 | 結果 |
|-------------|------|
| 各タスクは1つの明確なゴールを持っているか? | ✅ |
| 各タスクの完了を検証コマンドで確認できるか? | ✅ |
| Type 2タスクは既存コードへの影響が明確か? | ✅ |

---

## Step 2 — CSVパーサーの作成

最初に、CSVファイルを解析する純粋関数を作成します。
外部依存がないType 1タスクから始めることで、早期に検証可能な状態を作ります。

### Claudeへの指示

```
「CSVインポート機能を実装します。

まず、CSVパーサーを作成してください。

場所: src/modules/import/domain/CsvParser.ts
      src/modules/import/domain/CsvParser.test.ts

要件:
- CSV文字列を受け取り、行ごとの配列を返す
- ヘッダー行を解析してキー名として使用
- 空行はスキップ
- Result型で成功/失敗を返す

型定義:
type CsvParseResult = Result<CsvRow[], CsvParseError>
type CsvRow = Record<string, string>
type CsvParseError = { type: 'INVALID_FORMAT'; line: number; message: string }

テストケース:
- 正常系: 有効なCSVの解析
- 異常系: 不正なフォーマット（列数不一致）
- エッジ: 空のCSV、ヘッダーのみ

完了したら typecheck と test を実行して結果を教えてください。」
```

### 検証

**Step 2-1: typecheckの実行**

```bash
npm run typecheck
```

**期待される出力:**
```
Found 0 errors.
```

**Step 2-2: testの実行**

```bash
npm run test -- CsvParser
```

**期待される出力:**
```
 ✓ CsvParser > parse > should parse valid CSV
 ✓ CsvParser > parse > should return error for mismatched columns
 ✓ CsvParser > parse > should handle empty CSV
 ✓ CsvParser > parse > should handle header-only CSV

 Test Files  1 passed (1)
      Tests  4 passed (4)
```

<details>
<summary>テストが失敗した場合</summary>

**よくあるエラー:**

```
Expected: { ok: true, value: [...] }
Received: { ok: false, error: {...} }
```

→ CSVのフォーマット判定ロジックを確認してください。特に改行コード（`\n` vs `\r\n`）の処理に注意。

```
「テストが失敗しました。
改行コードが \r\n の場合も正しく処理できるように修正してください。」
```

</details>

---

## Step 3 — バリデーション層の作成

CSVの各行を売上データとして検証するバリデーション層を作成します。

### Claudeへの指示

```
「次に、CSVデータのバリデーション層を作成してください。

場所: src/modules/import/domain/SaleValidator.ts
      src/modules/import/domain/SaleValidator.test.ts

要件:
- CsvRowを受け取り、Saleエンティティに変換可能か検証
- zodスキーマで検証
- 複数のエラーがあればすべて収集

型定義:
const SaleRowSchema = z.object({
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
  productId: z.string().min(1),
  quantity: z.coerce.number().positive(),
  unitPrice: z.coerce.number().positive(),
})

type ValidationResult = Result<ValidatedSaleRow, ValidationError[]>
type ValidationError = { field: string; message: string }

テストケース:
- 正常系: 有効なデータ
- 異常系: 日付フォーマット不正、数値が負
- 異常系: 複数フィールドでエラー

既存のSaleエンティティ(src/domain/entities/Sale.ts)の型と整合性を取ってください。
完了したら typecheck と test を実行して結果を教えてください。」
```

### 検証

```bash
npm run typecheck && npm run test -- SaleValidator
```

**期待される出力:**
```
Found 0 errors.

 ✓ SaleValidator > validate > should accept valid sale row
 ✓ SaleValidator > validate > should reject invalid date format
 ✓ SaleValidator > validate > should reject negative quantity
 ✓ SaleValidator > validate > should collect multiple errors

 Test Files  1 passed (1)
      Tests  4 passed (4)
```

---

## Step 4 — インポートユースケースの実装

ここからが**Type 2タスク**です。既存のSaleRepositoryと連携します。

### 損切りポイントの設定

Type 2タスクは既存コードとの整合性問題が発生しやすいため、
**損切りラインを明確に設定**します。

```
┌─────────────────────────────────────────────────────────────────┐
│                    損切りライン設定                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  損切り条件:                                                    │
│  ・3往復以上の修正が必要になった場合                            │
│  ・30分以上経過しても typecheck が通らない場合                  │
│                                                                 │
│  損切り時のアクション:                                          │
│  ・既存コードの設計を見直す（リポジトリのインターフェース確認） │
│  ・タスクをさらに細かく分解する                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Claudeへの指示

```
「次に、インポートユースケースを実装してください。

場所: src/modules/import/application/ImportSalesUseCase.ts
      src/modules/import/application/ImportSalesUseCase.test.ts

要件:
- CSV文字列を受け取り、売上データをDBに保存
- CsvParser → SaleValidator → SaleRepository のパイプライン
- 各行を個別に処理し、エラーがあってもスキップして続行
- 処理結果のサマリーを返す

型定義:
type ImportResult = {
  total: number
  success: number
  failed: number
  errors: ImportError[]
}

type ImportError = {
  line: number
  errors: ValidationError[]
}

既存のSaleRepository(src/infrastructure/repositories/SaleRepository.ts)を使用してください。
完了したら typecheck, lint, test を実行して結果を教えてください。」
```

### 検証

```bash
npm run typecheck && npm run lint && npm run test -- ImportSalesUseCase
```

**期待される出力:**
```
Found 0 errors.
✔ No ESLint warnings or errors

 ✓ ImportSalesUseCase > execute > should import valid rows
 ✓ ImportSalesUseCase > execute > should skip invalid rows and continue
 ✓ ImportSalesUseCase > execute > should return correct summary

 Test Files  1 passed (1)
      Tests  3 passed (3)
```

<details>
<summary>Type 2タスクでよくある問題と対処</summary>

**既存の型と合わない場合:**

```
error TS2345: Argument of type 'ValidatedSaleRow' is not 
assignable to parameter of type 'CreateSaleInput'
```

→ まず既存の型定義を確認します:

```
「SaleRepository.create() の引数の型を教えてください。
現在のValidatedSaleRowと比較して、差分を明示してください。」
```

**リポジトリのインターフェースが想定と異なる場合:**

```
「既存のSaleRepositoryのインターフェースを確認しました。
create()ではなくsave()メソッドを使用するように修正してください。」
```

**損切りが必要な場合:**

3往復しても解決しない場合は、以下を試してください:

```
「一旦このタスクは保留します。
まず、SaleRepositoryのインターフェースを確認するタスクを先に実行してください。
- SaleRepository の public メソッド一覧
- 各メソッドの引数と戻り値の型
をリストアップしてください。」
```

</details>

---

## Step 5 — 最終確認と学びの記録

### 最終検証

```bash
npm run typecheck && npm run lint && npm run test
```

**期待される出力:**
```
Found 0 errors.
✔ No ESLint warnings or errors

 Test Files  4 passed (4)
      Tests  15 passed (15)
   Duration  2.34s
```

### 作成されたファイルの確認

```
src/modules/import/
├── domain/
│   ├── CsvParser.ts
│   ├── CsvParser.test.ts
│   ├── SaleValidator.ts
│   └── SaleValidator.test.ts
└── application/
    ├── ImportSalesUseCase.ts
    └── ImportSalesUseCase.test.ts
```

### 学びのCLAUDE.mdへの記録

```
「今回の実装で、CLAUDE.mdに追記すべき学びはありますか?」
```

**期待される提案:**

```markdown
## パターン（追記）
- CSVインポートはパイプライン構成で実装
  (Parser → Validator → Repository)

## 禁止事項（追記）
- CSVパース時は改行コード(\n, \r\n)の両方を考慮すること

## 学び
- Type 2タスクは損切りラインを先に決めておくと判断が早くなる
- Result型を使うとエラーの伝播が明示的になる
```

---

## ウォークスルーのまとめ

### Type 1 vs Type 2 の進め方の違い

| 観点 | Type 1（新規） | Type 2（既存連携） |
|------|---------------|-------------------|
| 開始前 | 要件の明確化 | 既存コードの把握 |
| 指示 | 新規作成を依頼 | 既存コードとの整合性を明示 |
| 検証 | 単体で完結 | 統合での動作確認が必要 |
| リスク | 低い | 高い（損切りライン設定） |

### パイプライン処理のポイント

```
CSV文字列
    │
    ↓ CsvParser（純粋関数）
CsvRow[]
    │
    ↓ SaleValidator（zodスキーマ）
ValidatedSaleRow | ValidationError[]
    │
    ↓ SaleRepository（副作用）
保存結果
    │
    ↓ 集計
ImportResult
```

**各段階で検証可能にする**ことで、問題発生時の切り分けが容易になります。

### Result型の活用

```typescript
// エラーを握りつぶさず、明示的に伝播
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E }

// 使用例
const parseResult = CsvParser.parse(csv)
if (!parseResult.ok) {
  return { ok: false, error: parseResult.error }
}
// parseResult.value が利用可能
```

---

## 付録A-1との比較

| 観点 | 付録A-1（ログイン） | 付録A-2（CSV変換） |
|------|-------------------|-------------------|
| タスクタイプ | Type 1中心 | Type 1 + Type 2 |
| エラー処理 | 例外ベース | Result型ベース |
| 検証の難易度 | 低 | 中（既存連携あり） |
| 損切り判断 | なし | あり（Step 4） |

---

次のドキュメント: [付録A-3: 既存コードのリファクタリング](./appendix-a-3-refactoring.md)
