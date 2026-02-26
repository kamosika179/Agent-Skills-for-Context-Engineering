---
name: advanced-evaluation
description: このスキルは、ユーザーが「LLM-as-judgeを実装する」「モデル出力を比較する」「評価ルーブリックを作成する」「評価バイアスを軽減する」と依頼した場合、または直接スコアリング、ペアワイズ比較、ポジションバイアス、評価パイプライン、自動品質評価に言及した場合に使用してください。
---

# 高度な評価

このスキルは、LLMを審査員として使用してLLM出力を評価するための本番グレードの手法をカバーします。学術論文、業界の実践、実践的な実装経験からの研究を統合し、信頼性の高い評価システムを構築するための実用的なパターンにまとめています。

**重要な洞察**：LLM-as-a-Judgeは単一の手法ではなく、それぞれ異なる評価コンテキストに適したアプローチのファミリーです。適切なアプローチを選択し、既知のバイアスを軽減することが、このスキルが育成するコアコンピテンシーです。

## アクティベーション条件

以下の場合にこのスキルをアクティベートしてください：

- LLM出力の自動評価パイプラインを構築する場合
- 最適なものを選択するために複数のモデル応答を比較する場合
- 評価チーム全体で一貫した品質基準を確立する場合
- 不整合な結果を示す評価システムをデバッグする場合
- プロンプトまたはモデル変更のA/Bテストを設計する場合
- 人間による評価または自動評価のルーブリックを作成する場合
- 自動評価と人間の判断の相関を分析する場合

## コアコンセプト

### 評価の分類体系

評価アプローチは、異なる信頼性プロファイルを持つ2つの主要カテゴリに分類されます：

**直接スコアリング**：単一のLLMが定義されたスケールで1つの応答を評価します。
- 最適な用途：客観的基準（事実の正確性、指示の遵守、毒性）
- 信頼性：明確に定義された基準では中程度から高い
- 失敗モード：スコアキャリブレーションのドリフト、スケール解釈の不整合

**ペアワイズ比較**：LLMが2つの応答を比較し、より良いものを選択します。
- 最適な用途：主観的な好み（トーン、スタイル、説得力）
- 信頼性：好みについては直接スコアリングより高い
- 失敗モード：ポジションバイアス、長さバイアス

MT-Bench論文（Zheng et al., 2023）の研究により、ペアワイズ比較は好みベースの評価において直接スコアリングよりも人間の審査員との一致度が高いことが確立されていますが、明確なグラウンドトゥルースを持つ客観的基準には直接スコアリングが適切です。

### バイアスの全体像

LLM審査員は、積極的に軽減する必要がある体系的なバイアスを示します：

**ポジションバイアス**：ペアワイズ比較で最初の位置の応答が優遇されます。軽減策：位置を入れ替えて2回評価し、多数決または整合性チェックを使用します。

**長さバイアス**：品質に関係なく、長い応答がより高く評価されます。軽減策：長さを無視するよう明示的にプロンプティングし、長さ正規化スコアリングを行います。

**自己強化バイアス**：モデルが自身の出力をより高く評価します。軽減策：生成と評価に異なるモデルを使用するか、制限を認識します。

**冗長性バイアス**：不必要な場合でも詳細な説明がより高いスコアを受けます。軽減策：無関係な詳細にペナルティを課す基準固有のルーブリック。

**権威バイアス**：正確性に関係なく、自信に満ちた権威的なトーンがより高く評価されます。軽減策：証拠の引用を要求し、ファクトチェックレイヤーを設けます。

### メトリクス選択フレームワーク

評価タスクの構造に基づいてメトリクスを選択します：

| タスクタイプ | 主要メトリクス | 副次メトリクス |
|------------|--------------|--------------|
| 二値分類（合格/不合格） | Recall, Precision, F1 | Cohen's κ |
| 順序尺度（1-5評価） | Spearman's ρ, Kendall's τ | Cohen's κ（加重） |
| ペアワイズ選好 | 一致率、ポジション整合性 | 信頼度キャリブレーション |
| マルチラベル | Macro-F1, Micro-F1 | ラベルごとのprecision/recall |

重要な洞察：高い絶対的一致度は、体系的な不一致パターンほど重要ではありません。特定の基準で人間と一貫して不一致する審査員は、ランダムなノイズを持つ審査員よりも問題があります。

## 評価アプローチ

### 直接スコアリングの実装

直接スコアリングには3つのコンポーネントが必要です：明確な基準、キャリブレーションされたスケール、構造化された出力フォーマット。

**基準定義パターン**：
```
Criterion: [Name]
Description: [What this criterion measures]
Weight: [Relative importance, 0-1]
```

**スケールキャリブレーション**：
- 1-3スケール：中立オプション付きの二値、認知負荷が最も低い
- 1-5スケール：標準的なLikertスケール、粒度と信頼性のバランスが良い
- 1-10スケール：高い粒度だがキャリブレーションが困難、詳細なルーブリックがある場合のみ使用

**直接スコアリングのプロンプト構造**：
```
You are an expert evaluator assessing response quality.

## Task
Evaluate the following response against each criterion.

## Original Prompt
{prompt}

## Response to Evaluate
{response}

## Criteria
{for each criterion: name, description, weight}

## Instructions
For each criterion:
1. Find specific evidence in the response
2. Score according to the rubric (1-{max} scale)
3. Justify your score with evidence
4. Suggest one specific improvement

## Output Format
Respond with structured JSON containing scores, justifications, and summary.
```

**Chain-of-Thought要件**：すべてのスコアリングプロンプトは、スコアの前に根拠を要求する必要があります。研究により、これはスコア優先アプローチと比較して信頼性を15〜25%向上させることが示されています。

### ペアワイズ比較の実装

ペアワイズ比較は、好みベースの評価において本質的により信頼性が高いですが、バイアス軽減が必要です。

**ポジションバイアス軽減プロトコル**：
1. 第1パス：応答Aを最初の位置に、応答Bを2番目に配置
2. 第2パス：応答Bを最初の位置に、応答Aを2番目に配置
3. 整合性チェック：パスが不一致の場合、信頼度を下げてTIEを返す
4. 最終判定：整合性のある勝者と平均化された信頼度

**ペアワイズ比較のプロンプト構造**：
```
You are an expert evaluator comparing two AI responses.

## Critical Instructions
- Do NOT prefer responses because they are longer
- Do NOT prefer responses based on position (first vs second)
- Focus ONLY on quality according to the specified criteria
- Ties are acceptable when responses are genuinely equivalent

## Original Prompt
{prompt}

## Response A
{response_a}

## Response B
{response_b}

## Comparison Criteria
{criteria list}

## Instructions
1. Analyze each response independently first
2. Compare them on each criterion
3. Determine overall winner with confidence level

## Output Format
JSON with per-criterion comparison, overall winner, confidence (0-1), and reasoning.
```

**信頼度キャリブレーション**：信頼度スコアはポジションの整合性を反映すべきです：
- 両方のパスが一致：信頼度 = 個々の信頼度の平均
- パスが不一致：信頼度 = 0.5、判定 = TIE

### ルーブリック生成

明確に定義されたルーブリックは、オープンエンドのスコアリングと比較して評価の分散を40〜60%削減します。

**ルーブリックの構成要素**：
1. **レベルの説明**：各スコアレベルの明確な境界
2. **特性**：各レベルを定義する観察可能な特徴
3. **例**：各レベルの代表的なテキスト（オプションだが有益）
4. **エッジケース**：曖昧な状況のためのガイダンス
5. **スコアリングガイドライン**：一貫した適用のための一般原則

**厳格さのキャリブレーション**：
- **寛容**：合格スコアの基準が低い、反復を奨励するのに適している
- **バランス**：公正、本番使用の典型的な期待値
- **厳格**：高い基準、安全性が重要またはハイステークスの評価に適している

**ドメイン適応**：ルーブリックはドメイン固有の用語を使用すべきです。「コードの可読性」ルーブリックは変数、関数、コメントに言及します。「医学的正確性」ルーブリックは臨床用語とエビデンス基準を参照します。

## 実践ガイダンス

### 評価パイプラインの設計

本番評価システムには複数のレイヤーが必要です：

```
┌─────────────────────────────────────────────────┐
│                 Evaluation Pipeline              │
├─────────────────────────────────────────────────┤
│                                                   │
│  Input: Response + Prompt + Context               │
│           │                                       │
│           ▼                                       │
│  ┌─────────────────────┐                         │
│  │   Criteria Loader   │ ◄── Rubrics, weights    │
│  └──────────┬──────────┘                         │
│             │                                     │
│             ▼                                     │
│  ┌─────────────────────┐                         │
│  │   Primary Scorer    │ ◄── Direct or Pairwise  │
│  └──────────┬──────────┘                         │
│             │                                     │
│             ▼                                     │
│  ┌─────────────────────┐                         │
│  │   Bias Mitigation   │ ◄── Position swap, etc. │
│  └──────────┬──────────┘                         │
│             │                                     │
│             ▼                                     │
│  ┌─────────────────────┐                         │
│  │ Confidence Scoring  │ ◄── Calibration         │
│  └──────────┬──────────┘                         │
│             │                                     │
│             ▼                                     │
│  Output: Scores + Justifications + Confidence     │
│                                                   │
└─────────────────────────────────────────────────┘
```

### よくあるアンチパターン

**アンチパターン：根拠なしのスコアリング**
- 問題：スコアに根拠がなく、デバッグや改善が困難
- 解決策：常にスコアの前にエビデンスに基づく根拠を要求する

**アンチパターン：シングルパスのペアワイズ比較**
- 問題：ポジションバイアスが結果を汚染する
- 解決策：常に位置を入れ替えて整合性をチェックする

**アンチパターン：過負荷の基準**
- 問題：複数のことを測定する基準は信頼性が低い
- 解決策：1つの基準 = 1つの測定可能な側面

**アンチパターン：エッジケースガイダンスの欠如**
- 問題：評価者が曖昧なケースを不整合に処理する
- 解決策：明示的なガイダンス付きでルーブリックにエッジケースを含める

**アンチパターン：信頼度キャリブレーションの無視**
- 問題：高信頼度の誤った判断は低信頼度よりも悪い
- 解決策：信頼度をポジションの整合性とエビデンスの強度にキャリブレーションする

### 意思決定フレームワーク：直接スコアリング vs. ペアワイズ

この決定木を使用してください：

```
Is there an objective ground truth?
├── Yes → Direct Scoring
│   └── Examples: factual accuracy, instruction following, format compliance
│
└── No → Is it a preference or quality judgment?
    ├── Yes → Pairwise Comparison
    │   └── Examples: tone, style, persuasiveness, creativity
    │
    └── No → Consider reference-based evaluation
        └── Examples: summarization (compare to source), translation (compare to reference)
```

### 評価のスケーリング

大量評価の場合：

1. **LLMパネル（PoLL）**：複数のモデルを審査員として使用し、投票を集約
   - 個々のモデルのバイアスを削減
   - より高価だが、ハイステークスの判断にはより信頼性が高い

2. **階層的評価**：スクリーニングに高速で安価なモデル、エッジケースに高価なモデル
   - 大量処理にコスト効率が良い
   - スクリーニング閾値のキャリブレーションが必要

3. **Human-in-the-loop**：明確なケースには自動評価、低信頼度には人間のレビュー
   - 重要なアプリケーションに最も信頼性が高い
   - 自動評価を改善するフィードバックループを設計

## 例

### 例1：正確性の直接スコアリング

**入力**：
```
Prompt: "What causes seasons on Earth?"
Response: "Seasons are caused by Earth's tilted axis. As Earth orbits the Sun, 
different hemispheres receive more direct sunlight at different times of year."
Criterion: Factual Accuracy (weight: 1.0)
Scale: 1-5
```

**出力**：
```json
{
  "criterion": "Factual Accuracy",
  "score": 5,
  "evidence": [
    "Correctly identifies axial tilt as primary cause",
    "Correctly explains differential sunlight by hemisphere",
    "No factual errors present"
  ],
  "justification": "Response accurately explains the cause of seasons with correct 
scientific reasoning. Both the axial tilt and its effect on sunlight distribution 
are correctly described.",
  "improvement": "Could add the specific tilt angle (23.5°) for completeness."
}
```

### 例2：ポジションスワップ付きペアワイズ比較

**入力**：
```
Prompt: "Explain machine learning to a beginner"
Response A: [Technical explanation with jargon]
Response B: [Simple analogy-based explanation]
Criteria: ["clarity", "accessibility"]
```

**第1パス（Aが最初）**：
```json
{ "winner": "B", "confidence": 0.8 }
```

**第2パス（Bが最初）**：
```json
{ "winner": "A", "confidence": 0.6 }
```
（注：Bが最初の位置にあったため、勝者はA）

**マッピングされた第2パス**：
```json
{ "winner": "B", "confidence": 0.6 }
```

**最終結果**：
```json
{
  "winner": "B",
  "confidence": 0.7,
  "positionConsistency": {
    "consistent": true,
    "firstPassWinner": "B",
    "secondPassWinner": "B"
  }
}
```

### 例3：ルーブリック生成

**入力**：
```
criterionName: "Code Readability"
criterionDescription: "How easy the code is to understand and maintain"
domain: "software engineering"
scale: "1-5"
strictness: "balanced"
```

**出力**（簡略化）：
```json
{
  "levels": [
    {
      "score": 1,
      "label": "Poor",
      "description": "Code is difficult to understand without significant effort",
      "characteristics": [
        "No meaningful variable or function names",
        "No comments or documentation",
        "Deeply nested or convoluted logic"
      ]
    },
    {
      "score": 3,
      "label": "Adequate",
      "description": "Code is understandable with some effort",
      "characteristics": [
        "Most variables have meaningful names",
        "Basic comments present for complex sections",
        "Logic is followable but could be cleaner"
      ]
    },
    {
      "score": 5,
      "label": "Excellent",
      "description": "Code is immediately clear and maintainable",
      "characteristics": [
        "All names are descriptive and consistent",
        "Comprehensive documentation",
        "Clean, modular structure"
      ]
    }
  ],
  "edgeCases": [
    {
      "situation": "Code is well-structured but uses domain-specific abbreviations",
      "guidance": "Score based on readability for domain experts, not general audience"
    }
  ]
}
```

## ガイドライン

1. **常にスコアの前に根拠を要求する** - Chain-of-thoughtプロンプティングは信頼性を15〜25%向上させる

2. **ペアワイズ比較では常に位置を入れ替える** - シングルパスの比較はポジションバイアスによって汚染される

3. **スケールの粒度をルーブリックの具体性に合わせる** - 詳細なレベルの説明なしに1-10を使用しない

4. **客観的基準と主観的基準を分離する** - 客観的には直接スコアリング、主観的にはペアワイズを使用

5. **信頼度スコアを含める** - ポジションの整合性とエビデンスの強度にキャリブレーションする

6. **エッジケースを明示的に定義する** - 曖昧な状況が最も評価の分散を引き起こす

7. **ドメイン固有のルーブリックを使用する** - 汎用的なルーブリックは汎用的な（有用性の低い）評価を生む

8. **人間の判断に対して検証する** - 自動評価は人間の評価と相関する場合にのみ価値がある

9. **体系的なバイアスを監視する** - 基準、応答タイプ、モデルごとの不一致パターンを追跡する

10. **反復のために設計する** - 評価システムはフィードバックループによって改善される

## 統合

このスキルは以下と統合します：

- **context-fundamentals** - 評価プロンプトには効果的なコンテキスト構造が必要
- **tool-design** - 評価ツールには適切なスキーマとエラーハンドリングが必要
- **context-optimization** - 評価プロンプトはトークン効率のために最適化可能
- **evaluation**（基礎） - このスキルは基礎的な評価コンセプトを拡張する

## 参考資料

内部参考資料：
- [LLM-as-Judge Implementation Patterns](./references/implementation-patterns.md)
- [Bias Mitigation Techniques](./references/bias-mitigation.md)
- [Metric Selection Guide](./references/metrics-guide.md)

外部研究：
- [Eugene Yan: Evaluating the Effectiveness of LLM-Evaluators](https://eugeneyan.com/writing/llm-evaluators/)
- [Judging LLM-as-a-Judge (Zheng et al., 2023)](https://arxiv.org/abs/2306.05685)
- [G-Eval: NLG Evaluation using GPT-4 (Liu et al., 2023)](https://arxiv.org/abs/2303.16634)
- [Large Language Models are not Fair Evaluators (Wang et al., 2023)](https://arxiv.org/abs/2305.17926)

このコレクションの関連スキル：
- evaluation - 基礎的な評価コンセプト
- context-fundamentals - 評価プロンプトのコンテキスト構造
- tool-design - 評価ツールの構築

---

## スキルメタデータ

**作成日**: 2024-12-24
**最終更新日**: 2024-12-24
**著者**: Muratcan Koylan
**バージョン**: 1.0.0

