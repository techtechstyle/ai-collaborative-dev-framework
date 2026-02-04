# 付録B: トラブルシューティング集

> よくある問題と解決策をまとめています。
> 問題発生時に参照してください。

---

## Q1: Claudeが存在しないAPIを提案してきた

**症状:**
Claudeが自信満々に「このAPIを使えば解決できます」と言うが、そのAPIは実際には存在しない。

**原因:**
幻覚(Hallucination)- Claudeが学習データに基づいてもっともらしいが存在しないAPIを生成。

**対処法:**

1. 「そのAPIの公式ドキュメントのURLを教えて」と確認
2. typecheckで検証(存在しないAPIはエラーになる)
3. 「このAPIが本当に存在するか、公式ドキュメントで確認して」

### 検証:解決できたか確認

typecheckを実行して、存在しないAPIの使用がエラーになるか確認します。

```bash
npm run typecheck
```

**期待される出力(問題が特定できた場合):**

```
error TS2339: Property 'nonExistentMethod' does not exist on type 'SomeLibrary'
```

このエラーが出れば、存在しないAPIを特定できています。該当箇所を正しいAPIに置き換えてください。

**期待される出力(修正後):**

```
Found 0 errors.
```

<details>
<summary>それでも解決しない場合</summary>

- 公式ドキュメントで正しいAPIを確認してください
- Claudeに「〇〇の公式ドキュメント(URL)を参照して、正しいAPIを使って」と指示してください
- ライブラリのバージョンを確認してください(古いバージョンではAPIが異なる場合があります)

</details>

**予防策:**
- 新しいライブラリを使う時は公式ドキュメントのURLを提供
- 「〇〇のv△△を使用」とバージョンを明示

---

## Q2: 長い会話で以前の指示を忘れる

**症状:**
会話が長くなると、最初に決めた命名規則や設計方針を無視し始める。

**原因:**
コンテキストウィンドウの限界。長い会話では前半の情報が薄れる。

**対処法:**

1. CLAUDE.mdに重要な決定を記録
2. 定期的に「CLAUDE.mdを確認して」とリマインド
3. 50ターン超えたら新しい会話を開始

### 検証:解決できたか確認

CLAUDE.mdに重要な決定が記録されているか確認します。

```bash
grep -E "命名規則|Naming|DON'T|禁止" CLAUDE.md
```

**期待される出力:**

```
## Coding Rules
// ❌ DON'T
- camelCase以外の命名禁止
```

記録されていれば、次回以降Claudeは自動的にこのルールを参照します。

<details>
<summary>それでも忘れる場合</summary>

- 会話の冒頭で「CLAUDE.mdを読んでから作業を開始してください」と明示的に指示
- 50ターンを超えたら新しい会話を開始
- 重要なルールは CLAUDE.md の上部に配置(上部ほど優先される傾向)

</details>

**予防策:**
- 重要な決定は即座にCLAUDE.mdに追記
- 10-15ターンごとに「命名規則を確認して」とリマインド

---

## Q3: typecheckエラーが大量に出る

**症状:**
typecheckを実行すると20件以上のエラーが表示される。

**原因:**
- 根本的な型定義の問題が連鎖的にエラーを発生
- 複数の問題が同時に存在

**対処法:**

Claudeへの指示:
```
「typecheckで20件のエラーが出ています。
 まず、根本原因となっているエラーを特定してください。
 その後、根本原因を修正し、再度typecheckを実行してください。」
```

### 検証:解決できたか確認

修正後、typecheckを実行してエラー数が減っているか確認します。

```bash
npm run typecheck 2>&1 | grep -c "error TS"
```

**期待される出力(改善中):**

```
5
```

最初の20件から5件に減っていれば、根本原因の修正が効いています。残りのエラーも同様に対処してください。

**期待される出力(完全解決):**

```
0
```

<details>
<summary>エラーが減らない場合</summary>

- エラーメッセージを1つずつ確認し、最も多く参照されているファイルから修正
- 型定義ファイル(*.d.ts)に問題がないか確認
- `node_modules` を削除して `npm install` を再実行

```bash
rm -rf node_modules
npm install
npm run typecheck
```

</details>

**予防策:**
- 小さな単位で検証を挟む
- 1文1検証

---

## Q4: テストが不安定(時々失敗する)

**症状:**
同じテストが時々成功し、時々失敗する。

**原因:**
- 非同期処理の競合
- テスト間の状態の干渉
- 時間依存のテスト

**対処法:**

Claudeへの指示:
```
「このテストが不安定な原因を分析してください。
 以下の観点で確認してください:
 - await の漏れ
 - beforeEach/afterEach でのリセット漏れ
 - 時間依存のロジック」
```

### 検証:解決できたか確認

テストを複数回実行して、安定して成功するか確認します。

```bash
for i in {1..5}; do npm run test -- --run; done
```

**期待される出力(安定した場合):**

```
 Test Files  5 passed (5)
 Test Files  5 passed (5)
 Test Files  5 passed (5)
 Test Files  5 passed (5)
 Test Files  5 passed (5)
```

5回連続で成功すれば、テストは安定しています。

<details>
<summary>まだ不安定な場合</summary>

**よくある原因と対処:**

| 原因 | 対処 |
|------|------|
| await漏れ | 非同期処理にawaitを追加 |
| 状態の干渉 | beforeEach/afterEachでリセット |
| 時間依存 | `vi.useFakeTimers()` で時間をモック化 |
| 外部依存 | API呼び出しをモック化 |

特定のテストだけ実行して問題を切り分け:
```bash
npm run test -- -t "問題のテスト名"
```

</details>

---

## Q5: Claudeが同じ修正を何度も提案する

**症状:**
Claudeが以前却下した修正を再び提案してくる。

**原因:**
- 会話の中で却下理由が薄れている
- CLAUDE.mdに禁止事項として記録されていない

**対処法:**

1. 「この修正は以前却下しました。理由は〇〇です」と再度伝える
2. CLAUDE.mdの禁止事項に追記
3. 新しい会話を開始し、禁止事項を明示

### 検証:解決できたか確認

CLAUDE.mdに禁止事項が追記されているか確認します。

```bash
grep -A 5 "禁止事項" CLAUDE.md
```

**期待される出力:**

```
## 禁止事項
- console.log をコミットしない
- any 型の使用禁止
- [新規追加] ○○パターンの使用禁止(理由: △△)
```

新しい禁止事項が追加されていれば、次回以降Claudeは同じ提案をしなくなります。

<details>
<summary>それでも提案してくる場合</summary>

- 新しい会話を開始し、「CLAUDE.mdを確認してから作業を始めてください」と指示
- 禁止事項の記述をより具体的に(例:「enumは禁止」→「TypeScriptのenumは禁止。代わりにUnion型を使用」)

</details>

**予防策:**
- 却下した修正は即座にCLAUDE.mdの禁止事項に追記
- 「なぜダメか」の理由も記録

---

## Q6: 生成されたコードが既存コードと整合しない

**症状:**
新しく生成されたコードが、既存のコードベースのスタイルや構造と合っていない。

**原因:**
- 既存コードを参照していない
- CLAUDE.mdにスタイルガイドがない

**対処法:**

Claudeへの指示:
```
「既存の〇〇.tsを参照して、同じスタイルで書いてください。」
```

### 検証:解決できたか確認

lintを実行して、スタイルが統一されているか確認します。

```bash
npm run lint
```

**期待される出力:**

```
✔ No ESLint warnings or errors
```

さらに、既存ファイルと新規ファイルの構造を比較:

```bash
head -20 src/modules/existing/Service.ts
head -20 src/modules/new/Service.ts
```

import文の順序、クラス構造、命名規則が一致していれば成功です。

<details>
<summary>スタイルが揃わない場合</summary>

- CLAUDE.mdにコーディングスタイルの具体例を追記
- 「この既存ファイルを参考に、同じ構造で実装してください」と明示的に指示
- ESLintルールを厳格化して強制

</details>

**予防策:**
- CLAUDE.mdにコーディングスタイルの例を記載
- 新機能実装時は「既存の〇〇を参照」と指示

---

## Q7: 環境依存の問題が発生

**症状:**
Claudeが生成したコードがローカルでは動くが、CI/本番では動かない。

**原因:**
- 環境変数の違い
- OSの違い(パス区切り等)
- Node.jsバージョンの違い

**対処法:**

1. CLAUDE.mdに環境情報を明記
2. 「この環境で動作することを確認して」と指示

### 検証:解決できたか確認

環境情報がCLAUDE.mdに記載されているか確認します。

```bash
grep -A 5 "環境情報" CLAUDE.md
```

**期待される出力:**

```
## 環境情報
- Node.js: v20.x
- OS: Linux (Ubuntu 22.04)
- パッケージマネージャ: npm
- CI: GitHub Actions
```

CIでも動作するか確認:

```bash
# ローカルでCI環境を再現(Docker使用)
docker run -v $(pwd):/app -w /app node:20 npm test
```

**期待される出力:**

```
 Test Files  X passed (X)
```

<details>
<summary>CIで失敗する場合</summary>

- パス区切りの問題: `path.join()` を使用しているか確認
- 環境変数: CIの設定で必要な環境変数が設定されているか確認
- Node.jsバージョン: `.nvmrc` または `package.json` の `engines` を確認

</details>

**CLAUDE.mdへの記載例:**
```markdown
## 環境情報
- Node.js: v20.x
- OS: Linux (Ubuntu 22.04)
- パッケージマネージャ: npm
- CI: GitHub Actions
```

---

## Q8: パフォーマンスが悪いコードが生成される

**症状:**
正しく動作するが、パフォーマンスが悪いコードが生成される。

**原因:**
- Claudeはパフォーマンス特性を実測できない
- 理論上正しくても実用上遅い実装

**対処法:**

Claudeへの指示:
```
「このコードのパフォーマンス特性を分析してください。
 要件: 1000件のデータを100ms以内に処理」
```

### 検証:解決できたか確認

パフォーマンステストを実行して、要件を満たしているか確認します。

```bash
npm run test -- --run -t "performance"
```

または、手動で計測:

```bash
time node -e "require('./dist/heavyFunction').process(testData)"
```

**期待される出力:**

```
real    0m0.095s
user    0m0.080s
sys     0m0.015s
```

100ms(0.1s)以内であれば要件を満たしています。

<details>
<summary>パフォーマンスが改善しない場合</summary>

- N+1問題がないか確認(DBアクセスをログ出力して件数を確認)
- アルゴリズムの計算量を確認(O(n²)がO(n)にできないか)
- キャッシュの導入を検討
- Claudeに「このコードのボトルネックを特定してください」と指示

</details>

**予防策:**
- パフォーマンス要件をCLAUDE.mdに明記
- 「N+1問題に注意」等の注意点を記載

---

## Q9: セキュリティ上の問題があるコードが生成される

**症状:**
SQLインジェクション、XSS等の脆弱性を含むコードが生成される。

**原因:**
- Claudeがセキュリティ要件を意識していない
- 明示的な指示がない

**対処法:**

Claudeへの指示:
```
「このコードにセキュリティ上の問題がないか確認してください。
 観点: SQLインジェクション、XSS」
```

### 検証:解決できたか確認

セキュリティチェックツールを実行します。

```bash
# npm auditで依存関係の脆弱性を確認
npm audit

# ESLintのセキュリティルールで確認
npx eslint --plugin security src/
```

**期待される出力(npm audit):**

```
found 0 vulnerabilities
```

**期待される出力(ESLint security):**

```
✔ No problems found
```

<details>
<summary>脆弱性が見つかった場合</summary>

- SQLインジェクション: パラメータ化クエリを使用
- XSS: ユーザー入力をエスケープ/サニタイズ
- 依存関係の脆弱性: `npm audit fix` を実行

Claudeへの指示:
```
「このコードにSQLインジェクションの脆弱性があります。
 パラメータ化クエリを使用して修正してください。」
```

</details>

**予防策:**

CLAUDE.mdへの記載例:
```markdown
## セキュリティルール
- ユーザー入力は必ずサニタイズ
- SQLはパラメータ化クエリを使用
- パスワードはbcryptでハッシュ化(ラウンド10以上)
```

---

## Q10: Claudeが自信満々に間違える

**症状:**
Claudeが「これで完璧です」「問題ありません」と言うが、実際にはバグがある。

**原因:**
過剰自信(Overconfidence)- Claudeは自己の出力を客観的に評価することが苦手。

**対処法:**

1. 必ず検証を実行(Claudeの言葉を信じない)
2. 「本当に正しいか、もう一度確認して」と促す
3. 複数の観点でレビューを依頼

### 検証:解決できたか確認

Claudeが「完璧」と言っても、必ず自分で検証を実行します。

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
 Test Files  X passed (X)
      Tests  Y passed (Y)
```

すべて成功して初めて「完璧」と判断してください。

<details>
<summary>検証で問題が見つかった場合</summary>

Claudeへの指示:
```
「『問題ありません』と言いましたが、typecheckでエラーが出ています。
 なぜこのエラーを見逃したか分析し、修正してください。」
```

この指摘を通じて、Claudeに自己検証の習慣を促します。

</details>

**予防策:**
- 「信頼するな、検証せよ」を常に意識
- 全ての実装に対してtypecheck + lint + testを実行
- Claudeの「完璧」発言を信じない習慣をつける

---

## Q11: ID型を取り違えたコードが生成される

**症状:**
UserIdとOrderIdなど、異なるID型を取り違えたコードが生成され、実行時にバグが発生する。

**原因:**
- すべてのIDが `string` 型として扱われている
- 型レベルでの区別がないためコンパイル時に検出できない

**対処法:**

1. Branded Typeを導入する
2. CLAUDE.mdに型ルールを追記

```typescript
// Branded Type の定義
type UserId = string & { readonly __brand: 'UserId' }
type OrderId = string & { readonly __brand: 'OrderId' }

// ファクトリ関数
const createUserId = (id: string): UserId => id as UserId
const createOrderId = (id: string): OrderId => id as OrderId
```

### 検証:解決できたか確認

意図的に間違った型を渡してtypecheckでエラーになることを確認します。

```typescript
// テストコード
function getUser(id: UserId): User { ... }

const orderId = createOrderId('order-123')
getUser(orderId)  // ← ここでエラーになるべき
```

```bash
npm run typecheck
```

**期待される出力(Branded Type導入後):**

```
error TS2345: Argument of type 'OrderId' is not assignable to parameter of type 'UserId'.
```

このエラーが出れば、型レベルでID取り違えを防止できています。

<details>
<summary>エラーが出ない場合</summary>

- Branded Typeが正しく定義されているか確認
- ファクトリ関数を使用しているか確認
- tsconfig.jsonで`strict: true`が有効か確認

</details>

**予防策:**

CLAUDE.mdへの記載例:
```markdown
## Type Rules
// ID型はBranded Typeで区別
type UserId = string & { readonly __brand: 'UserId' }
type OrderId = string & { readonly __brand: 'OrderId' }
```

---

## Q12: 型アサーション(as)が多用されたコードが生成される

**症状:**
Claudeが生成したコードに `as Type` や `as any` が多用されており、型安全性が損なわれている。

**原因:**
- 型エラーを「回避」するために型アサーションを使用
- 設計上の問題を型アサーションで隠蔽
- 適切な型ガードの代わりに型アサーションを使用

**対処法:**

1. 型アサーションの使用箇所を特定
2. 各箇所を適切な方法に置き換える

```typescript
// ❌ 危険:型アサーションで型チェックを回避
const user = data as User

// ❌ 特に危険:any経由のアサーション
const user = data as any as User

// ✅ 安全:型ガードで検証
function isUser(data: unknown): data is User {
  return (
    typeof data === 'object' &&
    data !== null &&
    'id' in data &&
    'email' in data
  )
}

if (isUser(data)) {
  const user = data  // 型ガードを通過したので安全
}

// ✅ 安全:zodで検証
const user = UserSchema.parse(data)
```

### 検証:解決できたか確認

コードベース内の型アサーションを検索します。

```bash
grep -rn "as any" src/
grep -rn "as [A-Z]" src/ | grep -v "as const"
```

**期待される出力(改善後):**

```
(出力なし、または許容されるケースのみ)
```

許容されるケース:
- `as const`(これは型アサーションではなく、リテラル型への変換)
- ファクトリ関数内での Branded Type 作成
- DOM操作での要素型指定

<details>
<summary>型アサーションが必要に見える場合</summary>

**設計を見直すサイン:**
- 型定義が実際のデータ構造と合っていない
- 外部APIの型定義が不完全
- 条件分岐で型が狭まることをTypeScriptが認識できていない

**対処:**
1. zodスキーマで型とバリデーションを統一
2. Discriminated Unionで状態を型安全に表現
3. 型ガード関数を作成

</details>

**予防策:**

CLAUDE.mdへの記載:
```markdown
## 禁止事項
- `as any` の使用は絶対禁止
- `as Type` は型ガードまたはzod検証の後のみ許可
- 型エラーを `as` で回避するのではなく、設計を見直す
```

Claudeへの指示:
```
「型アサーション(as)を使わずに実装してください。
 型エラーが出る場合は、型ガードまたはzodを使用してください。」
```

---

## クイックリファレンス

| 問題 | 最初に試すこと | 検証コマンド |
|------|---------------|-------------|
| 存在しないAPI | typecheck実行 | `npm run typecheck` |
| 指示を忘れる | CLAUDE.mdに記録 | `grep "キーワード" CLAUDE.md` |
| 大量のエラー | 根本原因を分類 | `npm run typecheck 2>&1 | grep -c "error"` |
| テスト不安定 | 5回連続実行 | `for i in {1..5}; do npm test; done` |
| 同じ提案 | 禁止事項に追記 | `grep "禁止" CLAUDE.md` |
| スタイル不一致 | 既存コードを参照させる | `npm run lint` |
| 環境依存 | 環境情報を明記 | `docker run ... npm test` |
| パフォーマンス | 要件を明示 | `time node script.js` |
| セキュリティ | レビューを依頼 | `npm audit` |
| 過剰自信 | 必ず検証 | `npm run typecheck && npm run lint && npm run test` |
| ID型取り違え | Branded Type導入 | `npm run typecheck` |
| 型アサーション多用 | 型ガード/zodに置換 | `grep -rn "as any" src/` |

---

前のドキュメント: [付録A: ウォークスルー例](./appendix-a-walkthrough.md)
次のドキュメント: [付録C: 指示設計ガイド](./appendix-c-prompt-design.md)
