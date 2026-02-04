# 付録A: ウォークスルー例 - ログイン機能の実装

> このウォークスルーでは、「ログイン機能」を最初から最後まで実装する過程を示します。
> タスク分解から検証完了まで、実際の開発フローを体験できます。

---

## 前提条件

このウォークスルーを始める前に、以下の準備が完了していることを確認してください。

| 項目 | 要件 | 確認方法 |
|------|------|---------|
| プロジェクト | 研修管理システムが作成済み | `package.json` が存在する |
| 技術スタック | NestJS + TypeScript + Prisma | `npm run typecheck` が実行できる |
| CLAUDE.md | Phase 3(製造中)まで設定済み | ファイルが存在し、Commandsセクションがある |
| Node.js | v18以上 | `node -v` で確認 |

---

## Step 1 — タスクの分解

ログイン機能を実装する前に、作業を検証可能な単位に分解します。これは人間が行う最重要タスクです。

### 分解結果

```
大タスク: ログイン機能の実装

分解結果:
1. Userエンティティの作成
   - 完了条件: typecheck通過
   - 所要時間目安: 15分

2. ログインAPIエンドポイントの実装
   - 完了条件: typecheck + lint通過
   - 所要時間目安: 30分

3. JWT認証の実装
   - 完了条件: typecheck + lint + test通過
   - 所要時間目安: 45分

4. 統合テストの作成
   - 完了条件: 全テスト通過
   - 所要時間目安: 30分

合計: 約2時間
```

### 検証:分解が適切か確認

以下の質問にすべて「はい」と答えられれば、分解は適切です。

- [ ] 各タスクは1つの明確なゴールを持っているか?
- [ ] 各タスクの完了を検証コマンドで確認できるか?
- [ ] 各タスクは30分以内に完了できそうか?

すべて確認できたら、Step 2に進みます。

---

## Step 2 — Userエンティティの作成

最初のタスクとして、認証の基盤となるUserエンティティを作成します。

### Claudeへの指示

```
「ログイン機能の実装を始めます。

まず、Userエンティティを作成してください。

場所: src/modules/auth/domain/entities/User.ts

要件:
- id: UserId(Branded Typeで定義)
- email: Email(zodでバリデーション)
- passwordHash: ハッシュ化されたパスワード
- name: ユーザー名
- role: 'admin' | 'user'(Union型)
- createdAt, updatedAt: 日時

型設計ルール:
- ID型はBranded Typeで区別(type UserId = string & { readonly __brand: 'UserId' })
- 外部入力のEmailはzodスキーマで検証
- roleはUnion型で明示

CLAUDE.md のコーディングルールに従ってください。
完了したら typecheck を実行して結果を教えてください。」
```

### 検証

Claudeがコードを生成したら、typecheckを実行して型エラーがないか確認します。

```bash
npm run typecheck
```

**期待される出力:**

```
✅ Done in 2.3s.
```

または

```
Found 0 errors.
```

エラー数が0であれば成功です。

<details>
<summary>エラーが出た場合の対処</summary>

**よくあるエラー:**

```
error TS2307: Cannot find module '@/shared/types'
```
→ インポートパスを確認してください。`tsconfig.json` のパス設定と一致しているか確認します。

```
error TS2564: Property 'id' has no initializer
```
→ クラスプロパティに初期値または `!` アサーションが必要です。Prismaを使用している場合は、エンティティの定義方法を確認してください。

</details>

typecheckが通ったら、次のステップに進みます。

---

## Step 3 — ログインAPIエンドポイントの実装

Userエンティティが完成したので、次はログインAPIを実装します。このステップでは、ユーザーの認証ロジックとコントローラーを作成します。

### Claudeへの指示

```
「次に、ログインAPIエンドポイントを実装してください。

場所: src/modules/auth/application/use-cases/Login.ts
      src/modules/auth/infrastructure/controllers/AuthController.ts

要件:
- POST /api/auth/login
- リクエスト: { email: string, password: string }
- レスポンス: { accessToken: string, user: { id, email, name, role } }
- パスワード検証にはbcryptを使用
- 認証失敗時は401エラー

先ほど作成したUserエンティティを使用してください。
完了したら typecheck と lint を実行して結果を教えてください。」
```

### 検証

コードが生成されたら、typecheckとlintを順番に実行します。

**Step 3-1: typecheckの実行**

```bash
npm run typecheck
```

**期待される出力:**

```
Found 0 errors.
```

**Step 3-2: lintの実行**

```bash
npm run lint
```

**期待される出力:**

```
✔ No ESLint warnings or errors
```

または警告のみで、エラーがない状態:

```
⚠ 2 warnings (unused variable など軽微なもの)
✔ 0 errors
```

<details>
<summary>typecheckでエラーが出た場合</summary>

**よくあるエラーと対処:**

```
error TS2322: Type 'string' is not assignable to type 'Email'
```
→ Email型のバリデーションが必要な場合は、カスタム型を使用するか、string型に変更してください。

```
error TS7006: Parameter 'user' implicitly has an 'any' type
```
→ 関数パラメータに明示的な型を追加してください。CLAUDE.mdで`any`禁止を指定している場合、Claudeに修正を依頼します:

```
「user パラメータに適切な型を付与してください。User型を使用してください。」
```

</details>

<details>
<summary>lintでエラーが出た場合</summary>

**よくあるエラーと対処:**

```
error  'bcrypt' is defined but never used  @typescript-eslint/no-unused-vars
```
→ 不要なインポートを削除するか、実際に使用するようにコードを修正してください。

```
error  Missing return type on function  @typescript-eslint/explicit-function-return-type
```
→ 関数に戻り値の型を明示的に追加してください。

</details>

両方の検証が通ったら、次のステップに進みます。

---

## Step 4 — JWT認証の実装

ログインAPIができたので、次はJWTトークンの発行と検証を実装します。このステップではテストも一緒に作成します。

### Claudeへの指示

```
「次に、JWT認証を実装してください。

場所: src/modules/auth/infrastructure/services/JwtService.ts
      src/modules/auth/infrastructure/services/JwtService.test.ts

要件:
- JWTの発行(有効期限: 1時間)
- JWTの検証
- リフレッシュトークン(有効期限: 7日)

テストも一緒に作成してください:
- 正常系: トークン発行・検証成功
- 異常系: 無効なトークン、期限切れトークン

完了したら typecheck, lint, test を実行して結果を教えてください。」
```

### 検証

このステップでは3つの検証を順番に実行します。

**Step 4-1: typecheckの実行**

```bash
npm run typecheck
```

**期待される出力:**

```
Found 0 errors.
```

**Step 4-2: lintの実行**

```bash
npm run lint
```

**期待される出力:**

```
✔ No ESLint warnings or errors
```

**Step 4-3: testの実行**

```bash
npm run test
```

**期待される出力:**

```
 ✓ JwtService > generateToken > should generate valid JWT token
 ✓ JwtService > verifyToken > should verify valid token
 ✓ JwtService > verifyToken > should throw error for invalid token
 ✓ JwtService > verifyToken > should throw error for expired token
 ✓ JwtService > refreshToken > should generate new token pair

 Test Files  1 passed (1)
      Tests  5 passed (5)
   Duration  1.23s
```

すべてのテストが `passed` になっていれば成功です。

<details>
<summary>テストが失敗した場合</summary>

**よくある失敗パターン:**

```
❌ JwtService > verifyToken > should throw error for expired token

Expected: TokenExpiredError
Received: JsonWebTokenError
```

→ JWT検証時のエラーハンドリングが不適切です。期限切れトークンは `TokenExpiredError` として処理する必要があります。Claudeに以下のように指示してください:

```
「テストが失敗しました。
期待値: TokenExpiredError
実際値: JsonWebTokenError

期限切れトークンの場合、TokenExpiredErrorを返すように
エラーハンドリングを修正してください。」
```

**テストがタイムアウトする場合:**

```
Timeout - Async callback was not invoked within 5000ms
```

→ 非同期処理が正しく完了していません。`await` の漏れがないか確認してください。

</details>

すべての検証が通ったら、最終ステップに進みます。

---

## Step 5 — 最終確認と学びの記録

すべてのタスクが完了しました。最後に全体の検証を行い、今回の学びをCLAUDE.mdに記録します。

### 最終検証

すべての検証コマンドをまとめて実行します。

```bash
npm run typecheck && npm run lint && npm run test
```

**期待される出力:**

```
# typecheck
Found 0 errors.

# lint
✔ No ESLint warnings or errors

# test
 Test Files  2 passed (2)
      Tests  15 passed (15)
   Duration  3.45s
```

すべて成功であれば、ログイン機能の実装は完了です。

### 作成されたファイルの確認

以下のファイルが作成されていることを確認します。

```bash
ls -la src/modules/auth/
```

**期待される出力:**

```
domain/
  entities/
    User.ts
application/
  use-cases/
    Login.ts
infrastructure/
  controllers/
    AuthController.ts
  services/
    JwtService.ts
    JwtService.test.ts
```

### 学びのCLAUDE.mdへの記録

Claudeに今回の実装で得た学びを聞きます。

```
「CLAUDE.mdに追記すべき学びはありますか?」
```

**期待される提案:**

```
「以下の追記を提案します:

## 禁止事項(追記)
- JWT検証エラーは種類別に処理する
  (TokenExpiredError と JsonWebTokenError を区別)

## 学び
- bcryptのラウンド数はCONFIGから読み取る(ハードコード禁止)」
```

この提案を確認し、適切であればCLAUDE.mdに追記します。

---

## ウォークスルーのまとめ

### 実行した検証の一覧

| Step | 検証コマンド | 確認内容 |
|------|-------------|---------|
| Step 2 | `npm run typecheck` | Userエンティティの型定義 |
| Step 3 | `npm run typecheck` + `npm run lint` | ログインAPIの実装 |
| Step 4 | `npm run typecheck` + `npm run lint` + `npm run test` | JWT認証とテスト |
| Step 5 | 全検証 | 最終確認 |

### ポイント

| # | ポイント | 内容 |
|---|---------|------|
| 1 | タスク分解は人間が行う | 検証可能な粒度に分解 |
| 2 | 各タスクで検証を実行 | エラーを早期発見 |
| 3 | エラー発生時は具体的に指示 | 「修正して」ではなく、どう修正するか示唆 |
| 4 | 検証が通ってから次へ | エラーの蓄積を防ぐ |
| 5 | 完了後に学びをCLAUDE.mdに追記 | 次回以降に活かす |

### 使用したPrompt Pattern

| Step | Pattern | 目的 |
|------|---------|------|
| Step 2, 3, 4 | Step-by-Step | 段階的な実装 |
| エラー発生時 | エラーリカバリー | エラーの修正 |
| Step 5 | Review Mode | 学びの抽出 |

---

次のドキュメント: [付録B: トラブルシューティング](./appendix-b-troubleshooting.md)
