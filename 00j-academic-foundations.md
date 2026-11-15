# 学術基盤 — フレームワークはどんな知の上に立っているか

> **このドキュメントの位置づけ**: [Part 0: 哲学と原則](./00-philosophy.md)の基盤章
>
> **対象レベル**: L2（実践）〜L3（最適化）。最初の「この章の目的」と「根拠の種類の表記」はL1向け
>
> **内容**: フレームワークの各層と原則が、どんな学術的知見に支えられているか。そして、どんな知見がフレームワークに疑問を投げているか

---

## この章の目的 [L1]

このフレームワークは「期待値0から始める」「AIの出力を盲目的に信頼しない（BL4）」ことを求めています。
その姿勢は、フレームワーク自身にも向けられるべきです。

**「このフレームワークの主張は、何に基づいているのか？」**

この章は、その問いに答えるためにあります。目的は3つです。

1. **根拠を一か所にまとめる** — 各章で「なぜそう言えるのか」を問われたとき、ここを見れば辿れるようにする
2. **根拠の強さを区別する** — 実験で確かめられたことと、たとえ話で示唆されることを、同じ重みで語らない
3. **反証側の知見も載せる** — フレームワークの書き方に疑問を投げる研究を、隠さずに示す

> ⚠️ この章は「フレームワークが学術的に証明されている」と主張するものではありません。
> 多くの知見は**示唆**であり、**証明**ではありません。その区別を明示することが、この章の役割です。

---

## 根拠の種類の表記 [L1]

V5.0では、学術的な根拠に次の4つの表記を付けます。本文中で `[E]` などの記号を見かけたら、この表を参照してください。

| 表記 | 種類 | 意味 | 本文での言い方 |
|------|------|------|-------------|
| **[E]** | 実証（Empirical） | 実験、縦断研究、メタ分析で確かめられた知見 | 「研究によると」「示されている」 |
| **[F]** | 形式理論（Formal） | ゲーム理論などの数理モデルを当てはめた推論 | 「モデルで分析すると」「示唆される」 |
| **[A]** | アナロジー（Analogy） | 構造が似ていることによる示唆 | 「〜にたとえると」「〜と読むこともできる」 |
| **[H]** | 哲学・歴史（Historical） | 思想や歴史の流れによる解釈 | 「〜の系譜に位置づけられる」 |

**書き方のルール**

- 「証明する」「正当化する」という言い方は、**[E]** で直接示されたこと以外には使いません
- **[F]** の分析では、人間とAIを「利得を持つプレイヤー」とみなしています。AIの「利得」や「裏切り」はメタファーです
- 数値は一次資料で確認したものだけを載せます
- README の「[数値の根拠について](./README.md#数値の根拠について)」の分類（研究／業界報告／実践的な目安／目標値）と併用します

---

## フレームワークの知的地盤 — 3段構造 [L1]

フレームワークを支える知は、役割の違いで3段に分けられます。

```
┌─────────────────────────────────────────────────────────────────┐
│                  フレームワークの知的地盤                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  【メタ哲学】 増強主義（Licklider, Engelbart）                  │
│   └ 全体を貫く立場: AIは人間を置き換えるのではなく増強する      │
│                                                                 │
│  【知的源流】 7つの源流                                         │
│   └ 原則が「なぜ正しいか」を支える                              │
│     反証主義 / プラグマティズム / 分散認知 / 認知負荷理論 /     │
│     社会技術システム論 / 足場かけ理論 / 信頼較正理論            │
│                                                                 │
│  【分析の道具】                                                 │
│   └ 特定の原則を、別の角度から分析するときに使う                │
│     ゲーム理論・ESS / 行動経済学 / 文化心理学 /                 │
│     パラダイム論 / タスク分割の学際理論                         │
│                                                                 │
│  ─────────────────────────────────────────                      │
│  【参考に留める】 進化生物学 / 進化心理学                       │
│   └ 発想の手がかりとしては有用だが、根拠としては使わない        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**なぜ3段に分けるのか**

知的源流は「この原則は正しいか」を支えます。分析の道具は「この原則はどう働くか」「なぜ広まりにくいか」を説明します。
役割の違うものを同じ「柱」として並べると、どの主張がどれだけ強く支えられているのかが見えなくなります。

---

## メタ哲学: 増強主義 [L2]

| 項目 | 内容 |
|------|------|
| 源流 | Licklider「Man-Computer Symbiosis」（1960）、Engelbart「Augmenting Human Intellect」（1962） |
| 核心 | 人間とコンピュータは役割を分担し、互いの強みで知的能力を高め合う |
| フレームワークでの位置 | 全5層を貫く立場。「判断は常に人間」（BL1）の思想的な出発点 |
| 根拠の種類 | **[H]** |

詳しくは [00b-augmentation.md](./00b-augmentation.md) の「人間-コンピュータ協働の知的系譜」を参照してください。

---

## 7つの知的源流 [L2]

| # | 源流 | 代表的な論者 | 核心の考え | 主に支える要素 | 種類 |
|---|------|-----------|-----------|-------------|------|
| 1 | **反証主義** | Popper（1934/1959） | 知識は、確かめることより、反駁を試みることで確実に進む | 1文1検証、反証の原則、BL2、BL4 | [H] |
| 2 | **プラグマティズム** | Peirce, James, Dewey | 考えの価値は、実践で問題を解決できるかで決まる | L0、ボーイスカウトルール、期待値0 | [H] |
| 3 | **分散認知** | Hutchins（1995） | 認知は個人の頭の中だけでなく、人・道具・環境にまたがって起きる | L4、認知労働分業、CLAUDE.md | [H]＋[E] |
| 4 | **認知負荷理論** | Sweller（1988） | ワーキングメモリには限りがあり、学習と作業の質は負荷の設計で決まる | L3、1文1検証の「1文」 | [E] |
| 5 | **社会技術システム論** | Trist & Bamforth（1951） | 技術と社会の仕組みは一体で設計しなければ機能しない | L0、L1、チーム導入 | [H]＋[E] |
| 6 | **足場かけ理論** | Wood, Bruner & Ross（1976）、Vygotsky | 学ぶ人の少し先を支え、できるようになったら支えを外す | L2、フェーディング、成長モデル | [E] |
| 7 | **信頼較正理論** | Lee & See（2004） | 自動化への信頼は、システムの実際の能力に合わせて調整されるべきである | BL4、レビューの深さ | [E] |

---

## 層ごとの学術的根拠 [L2]

### L0 持続可能性

| 主張 | 根拠 | 種類 |
|------|------|------|
| AIへの頼りすぎは、批判的思考を弱める方向に働く | AIツールの頻繁な使用と批判的思考の負の相関（Gerlich 2025）。相関研究であり、因果は示していない | [E] |
| 支援を受けたままでは学習が起きにくい | AIの推薦を見せると判断はよくなるが、学習は起きなかった（Gajos & Mamykina 2022） | [E] |
| 長く続く関係ほど、協力は保たれやすい | 繰り返しゲームのフォーク定理 | [F] |
| 実践で問題を解決できるかで価値を判断する | プラグマティズム | [H] |

### L1 心理的安全性

| 主張 | 根拠 | 種類 |
|------|------|------|
| 心理的安全性はチームの学習行動を支える | Edmondson（1999） | [E] |
| AIを使うこと自体が、周囲からの評価を下げることがある | HBR（2025年8月）が報じた「コンピテンス・ペナルティ」研究 | [E] |
| 新しい方法への抵抗は、非合理ではなく、機能している方法への合理的なこだわりである | Kuhn（1962）、Laudan（1977） | [H] |

### L2 急がば回れ

| 主張 | 根拠 | 種類 |
|------|------|------|
| 速くなったという実感と、実際の速さは食い違うことがある | 経験豊富な開発者がAIを使うと、実際には19%遅くなったが、本人は速くなったと感じていた（METR 2025、RCT） | [E] |
| 学ぶ人の少し先を支え、できたら外す | 足場かけ理論 | [E] |
| 初めは厳密に確かめ、信頼が積み上がったら確認を減らす戦略が、常に全量を確かめる戦略より長期的に優れる。ただし確認をゼロにすると崩れる | 進化ゲーム理論による信頼の分析（Han, Perret & Powers 2021） | [F] |

### L3 シンプルさ

| 主張 | 根拠 | 種類 |
|------|------|------|
| 一度に扱える要素には限りがある | 認知負荷理論（Sweller 1988） | [E] |
| 最適なタスクの粒度は一つではなく、タスクと実行者によって変わる | 人の課題分解は、効用と計算コストのトレードオフで説明できる（Correa et al. 2023） | [E] |
| 迷ったら小さい側に寄せる方が合理的 | バッチサイズのU字型コスト曲線（Reinertsen 2009） | [F] |

### L4 AIファースト

| 主張 | 根拠 | 種類 |
|------|------|------|
| 人とAIで認知の仕事を分け合える | 分散認知（Hutchins 1995） | [H]＋[E] |
| 人間とAIを組み合わせても、自動的にはよくならない。平均すると、優れた方が単独で行うより劣る。特に判断のタスクで劣る | 106実験のメタ分析（Vaccaro, Almaatouq & Malone 2024） | [E] |
| 相補性は、情報の非対称性（人間だけが知る文脈）と、能力の非対称性（間違え方の違い）から生まれる | Hemmer et al.（arXiv:2404.00029） | [E]＋[F] |
| AIの出力を先に見ると、人の判断はそれに引きずられる | アンカリングと、判断時間の効果（Rastogi et al. 2022） | [E] |

### 1文1検証

| 主張 | 根拠 | 種類 |
|------|------|------|
| 「1文」：入力を小さく保つと、理解と指示の質が保たれる | 認知負荷理論 | [E] |
| 「1検証」：判断の前に考えることを強制する仕組みは、AIへの過信を減らす | 認知的強制機能（Buçinca et al. 2021） | [E] |
| 確かめることより、反駁を試みることで知識は進む | 反証主義（Popper）、TDD、ミューテーションテスト | [H] |
| 事前にすべてを仕様に書くことはできないので、小さく合意と確認を重ねるのが合理的 | 不完備契約の理論、AIへの委任の意思決定理論（Herrmann 2025） | [F] |
| LLMは反証の実行を大規模に担える | LLMによるミューテーションテストの産業展開（Meta、FSE 2025） | [E] |

### BL4（盲目的信頼の禁止）と無慈悲な親切心

| 主張 | 根拠 | 種類 |
|------|------|------|
| 自動化への信頼は、実際の能力に合わせて較正されるべきである | 信頼較正理論（Lee & See 2004） | [E] |
| AIを使うと、自分の成績を過大に評価しやすい。AIリテラシーが高い人ほど、その傾向が強いこともある | Fernandes et al.（2026） | [E] |
| 生成AIは、使う人に高いメタ認知（自分の考えを監視する力）を求める | Tankelevitch et al.（2024） | [E]＋[H] |
| アルゴリズムを避ける段階から、AIに依存する段階へ移る構造がある | Kim et al.（2026） | [H] |

---

## 反証側の知見 — フレームワークに疑問を投げる研究 [L2]

フレームワークを支える知見だけを集めると、確証バイアスに陥ります。
ここでは、**v4.8までの書き方に修正を迫った知見**を載せます。V5.0の改訂は、これらへの応答でもあります。

| 知見 | v4.8の書き方への疑問 | V5.0での応答 |
|------|-------------------|-------------|
| 人間とAIの組み合わせは、判断のタスクでは平均して性能が下がる（Vaccaro et al. 2024）**[E]** | F1「初期案生成はAIから」は、判断のタスクにも当てはまるのか | F1を生成のタスクに限定し、判断のタスクでは人間が先に見立てを持つ原則（F5）を加える（[00g](./00g-ai-first.md)） |
| AIの案を先に見ると、判断が引きずられる（Rastogi et al. 2022）**[E]** | 同上 | 同上 |
| 確認を常に全量で行う戦略は、信頼に基づく戦略に長期的には劣る（Han et al. 2021）**[F]** | 人間のレビューは、常に同じ深さであるべきか | 1文1検証を二層に分け、人間のレビューの深さを信頼とリスクで変える（[00e](./00e-haste-simplicity.md)） |
| AIリテラシーを高めても、自己評価の精度はよくならない（Fernandes et al. 2026）**[E]** | BL4を「知っている」だけで、盲目的な信頼は防げるのか | 知識ではなく仕組み（予測→実測の振り返り、リライアンスドリル）で認知的関与を保つ（[00i](./00i-merciless-kindness.md)） |
| 自律的に長く動くエージェントでは、人間が各ステップを確認できない | 人間の役割は「各ステップの確認者」だけでよいのか | 人間が反証の基準を設計し、反証の実行はAIやツールに任せる（Human-on-the-Loop、[00e](./00e-haste-simplicity.md)） |

> 💡 この表は、V5.0以降も更新し続けます。新しい反証側の知見は、フレームワークを改善する材料です。

---

## 分析の道具 [L3]

以下の理論は、原則の根拠そのものではなく、**原則がどう働くかを別の角度から分析する道具**として使います。

| 道具 | 何を分析するのに使うか | 注意点 |
|------|-------------------|-------|
| **ゲーム理論・ESS（進化的に安定な戦略）** | 検証の頻度、委任の範囲、信頼の構築を、戦略として分析する | 人間とAIを、利得を持つプレイヤーとみなしたモデルである。AIに「利得」や「意図」があるわけではない |
| **行動経済学** | オートメーションバイアス、アンカリング、現在バイアスなど、判断のゆがみと、それへの介入（ナッジ、コミットメントデバイス）を設計する | 介入の効果は文脈に依存する。他の文脈での効果を、そのまま当てはめない |
| **文化心理学** | 日本の職場（報連相、稟議、面子）で、心理的安全性やAI導入をどう設計するかを考える | 国の文化次元（Hofstedeなど）は集団の傾向であり、個人やチームを決めつけるものではない |
| **パラダイム論**（Kuhn, Lakatos, Laudan） | AI協働への抵抗や、普及の進み方を説明する。[04-growth-adoption.md](./04-growth-adoption.md) で使う | ソフトウェアの方法論は、Kuhnの厳密な意味でのパラダイムではない。概念を借りる範囲を限定する |
| **タスク分割の学際理論**（認知科学、SE、OR） | タスクの粒度、階層、バッチサイズを検討する | 「最適な粒度」は一つに決まらない |

---

## 参考に留める知見 [L3]

**進化生物学**（共生のスペクトラム、ニッチ構築、ボールドウィン効果など）と**進化心理学**（進化的ミスマッチ、エージェンシー検知など）は、
AI協働を考える発想の手がかりとしては有用です。

しかし、これらの知見をAI協働に当てはめるのは、ほとんどが**アナロジー [A]** です。研究者自身も、直接の実証はこれからの課題だと述べています。
このフレームワークでは、**原則の根拠としては使いません**。

---

## 根拠の読み方 — 読者へのお願い [L2]

この章の根拠も、AIの出力と同じように、**盲目的に信頼しないでください**。

- 気になる主張は、参考文献の一次資料に当たってください
- 「[F]」や「[H]」の主張は、別の解釈がありえます
- 自分のチームで試して、合わなければ、その経験こそが大切な反証です

フレームワークの主張に対する反証や、新しい知見があれば、[CONTRIBUTING.md](./CONTRIBUTING.md) に沿って提案してください。

---

## 参考文献

### 知的源流・メタ哲学

- Dijkstra, E. W. (1970). Notes on structured programming (EWD249).
- Engelbart, D. C. (1962). *Augmenting Human Intellect: A Conceptual Framework*. SRI.
- Hutchins, E. (1995). *Cognition in the Wild*. MIT Press.
- Lee, J. D., & See, K. A. (2004). Trust in automation: Designing for appropriate reliance. *Human Factors*, 46(1), 50–80.
- Licklider, J. C. R. (1960). Man-computer symbiosis. *IRE Transactions on Human Factors in Electronics*, HFE-1, 4–11.
- Popper, K. (1959). *The Logic of Scientific Discovery*. Hutchinson.（原著 1934）
- Sweller, J. (1988). Cognitive load during problem solving: Effects on learning. *Cognitive Science*, 12(2), 257–285.
- Wood, D., Bruner, J. S., & Ross, G. (1976). The role of tutoring in problem solving. *Journal of Child Psychology and Psychiatry*, 17(2), 89–100.
- Trist, E. L., & Bamforth, K. W. (1951). Some social and psychological consequences of the longwall method of coal-getting. *Human Relations*, 4(1), 3–38.

### 実証研究・形式理論

- Buçinca, Z., Malaya, M. B., & Gajos, K. Z. (2021). To trust or to think: Cognitive forcing functions can reduce overreliance on AI in AI-assisted decision-making. *Proc. ACM HCI*, 5(CSCW1).
- Correa, C. G., Ho, M. K., Callaway, F., Daw, N. D., & Griffiths, T. L. (2023). Humans decompose tasks by trading off utility and computational cost. *PLOS Computational Biology*.
- Farley, D. (2021). *Modern Software Engineering*. Addison-Wesley.
- Edmondson, A. (1999). Psychological safety and learning behavior in work teams. *Administrative Science Quarterly*, 44(2), 350–383.
- Fernandes, D., et al. (2026). AI makes you smarter but none the wiser: The disconnect between performance and metacognition. *Computers in Human Behavior*, 175, 108779.
- Gajos, K. Z., & Mamykina, L. (2022). Do people engage cognitively with AI? Impact of AI assistance on incidental learning. *Proc. IUI 2022*.
- Gerlich, M. (2025). AI tools in society: Impacts on cognitive offloading and the future of critical thinking. *Societies*, 15(1), 6.
- Groce, A., et al. (2018). How verified (or tested) is my code? Falsification-driven verification and testing. *Automated Software Engineering*, 25.
- Han, T. A., Perret, C., & Powers, S. T. (2021). When to (or not to) trust intelligent machines: Insights from an evolutionary game theory analysis of trust in repeated games. *Cognitive Systems Research*.
- Hemmer, P., et al. Complementarity in human-AI collaboration. arXiv:2404.00029.
- Herrmann (2025). A decision-theoretic approach for managing misalignment. arXiv:2512.15584.
- Kim, et al. (2026). From algorithm aversion to AI dependence: Deskilling, upskilling, and emerging addictions in the GenAI age. *Consumer Psychology Review*. doi:10.1002/arcp.70008
- METR (2025). Measuring the impact of early-2025 AI on experienced open-source developer productivity. arXiv:2507.09089.
- Rastogi, C., et al. (2022). Deciding fast and slow: The role of cognitive biases in AI-assisted decision-making. *Proc. ACM HCI*, 6(CSCW1).
- Reinertsen, D. G. (2009). *The Principles of Product Development Flow*. Celeritas.
- Tankelevitch, L., et al. (2024). The metacognitive demands and opportunities of generative AI. *Proc. CHI 2024*.
- Vaccaro, M., Almaatouq, A., & Malone, T. (2024). When combinations of humans and AI are useful: A systematic review and meta-analysis. *Nature Human Behaviour*.

### 分析の道具

- Kuhn, T. S. (1962). *The Structure of Scientific Revolutions*. University of Chicago Press.
- Lakatos, I. (1970). Falsification and the methodology of scientific research programmes.
- Laudan, L. (1977). *Progress and Its Problems*. University of California Press.

### 産業報告・記事

- Harvard Business Review (2025年8月). Research: The hidden penalty of using AI at work.
- Meta. Mutation-guided LLM-based test generation at Meta（FSE 2025 Industry Track）.

> 📝 V5.0の学術基盤の整理に使った調査資料と、引用の確認結果は [docs/v5-academic-map.md](./docs/v5-academic-map.md) にあります。

---

> 💡 **関連ドキュメント**
> - [増強の哲学・知的系譜](./00b-augmentation.md) — メタ哲学の詳細
> - [急がば回れ・シンプルさ・1文1検証](./00e-haste-simplicity.md) — 認知負荷理論と反証の実践
> - [AIファースト](./00g-ai-first.md) — 分散認知と相補性の設計
> - [無慈悲な親切心](./00i-merciless-kindness.md) — 信頼較正とメタ認知
> - [成長と導入](./04-growth-adoption.md) — パラダイム論による普及の分析
