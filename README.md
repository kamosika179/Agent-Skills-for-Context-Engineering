# Agent Skills for Context Engineering

あらゆるエージェントプラットフォームで本番レベルのAIエージェントシステムを構築するための、コンテキストエンジニアリングの原則に焦点を当てた包括的なオープンコレクションです。これらのスキルは、エージェントの有効性を最大化するためのコンテキストキュレーションの技術と科学を教えます。

## コンテキストエンジニアリングとは？

コンテキストエンジニアリングとは、言語モデルのコンテキストウィンドウを管理する技術分野です。効果的な指示の作成に焦点を当てるプロンプトエンジニアリングとは異なり、コンテキストエンジニアリングは、モデルの限られたアテンション予算に入るすべての情報（システムプロンプト、ツール定義、検索されたドキュメント、メッセージ履歴、ツール出力）の包括的なキュレーションに取り組みます。

根本的な課題は、コンテキストウィンドウが生のトークン容量ではなく、アテンションメカニクスによって制約されることです。コンテキスト長が増加すると、モデルは予測可能な劣化パターンを示します：「lost-in-the-middle」現象、U字型アテンションカーブ、アテンション不足です。効果的なコンテキストエンジニアリングとは、望ましい結果の可能性を最大化する、最小限の高シグナルトークンセットを見つけることを意味します。

## 評価・引用

このリポジトリは、静的スキルアーキテクチャに関する基礎的研究として学術論文で引用されています：

> "While static skills are well-recognized [Anthropic, 2025b; Muratcan Koylan, 2025], MCE is among the first to dynamically evolve them, bridging manual skill engineering and autonomous self-improvement."

— [Meta Context Engineering via Agentic Skill Evolution](https://arxiv.org/pdf/2601.21557), Peking University State Key Laboratory of General Artificial Intelligence (2026)

## スキル概要

### 基礎スキル

これらのスキルは、後続のすべてのコンテキストエンジニアリング作業に必要な基礎的理解を確立します。

| スキル | 説明 |
|-------|-------------|
| [context-fundamentals](skills/context-fundamentals/) | コンテキストとは何か、なぜ重要なのか、エージェントシステムにおけるコンテキストの構造を理解する |
| [context-degradation](skills/context-degradation/) | コンテキスト障害のパターンを認識する：lost-in-middle、ポイズニング、ディストラクション、クラッシュ |
| [context-compression](skills/context-compression/) | 長時間セッション向けの圧縮戦略を設計・評価する |

### アーキテクチャスキル

これらのスキルは、効果的なエージェントシステムを構築するためのパターンと構造をカバーします。

| スキル | 説明 |
|-------|-------------|
| [multi-agent-patterns](skills/multi-agent-patterns/) | オーケストレータ、ピアツーピア、階層型マルチエージェントアーキテクチャを習得する |
| [memory-systems](skills/memory-systems/) | 短期、長期、グラフベースのメモリアーキテクチャを設計する |
| [tool-design](skills/tool-design/) | エージェントが効果的に使用できるツールを構築する |
| [filesystem-context](skills/filesystem-context/) | 動的コンテキスト検出、ツール出力のオフロード、プラン永続化にファイルシステムを活用する |
| [hosted-agents](skills/hosted-agents/) | **NEW** サンドボックスVM、ビルド済みイメージ、マルチプレイヤーサポート、マルチクライアントインターフェースを備えたバックグラウンドコーディングエージェントを構築する |

### 運用スキル

これらのスキルは、エージェントシステムの継続的な運用と最適化に対応します。

| スキル | 説明 |
|-------|-------------|
| [context-optimization](skills/context-optimization/) | コンパクション、マスキング、キャッシング戦略を適用する |
| [evaluation](skills/evaluation/) | エージェントシステムの評価フレームワークを構築する |
| [advanced-evaluation](skills/advanced-evaluation/) | LLM-as-a-Judge技法を習得する：直接スコアリング、ペアワイズ比較、ルーブリック生成、バイアス緩和 |

### 開発方法論

これらのスキルは、LLMを活用したプロジェクト構築のメタレベルのプラクティスをカバーします。

| スキル | 説明 |
|-------|-------------|
| [project-development](skills/project-development/) | アイデア創出からデプロイまでのLLMプロジェクトの設計と構築。タスクモデル適合性分析、パイプラインアーキテクチャ、構造化出力設計を含む |

### 認知アーキテクチャスキル

これらのスキルは、合理的なエージェントシステムのための形式的認知モデリングをカバーします。

| スキル | 説明 |
|-------|-------------|
| [bdi-mental-states](skills/bdi-mental-states/) | **NEW** 外部RDFコンテキストを、形式的BDIオントロジーパターンを使用してエージェントの精神状態（信念、欲求、意図）に変換し、熟慮型推論と説明可能性を実現する |

## 設計哲学

### 段階的開示

各スキルは効率的なコンテキスト使用のために構造化されています。起動時、エージェントはスキル名と説明のみを読み込みます。フルコンテンツは、関連タスクでスキルがアクティベートされたときにのみ読み込まれます。

### プラットフォーム非依存

これらのスキルは、ベンダー固有の実装ではなく、移植可能な原則に焦点を当てています。パターンはClaude Code、Cursor、およびスキルをサポートするかカスタム指示を許可するあらゆるエージェントプラットフォームで機能します。

### 概念的基盤と実践的サンプル

スクリプトとサンプルは、特定の依存関係のインストールを必要とせず、環境を問わず動作するPython疑似コードを使用してコンセプトを実演します。

## 使い方

### Claude Codeでの使い方

このリポジトリは、Claudeがタスクコンテキストに基づいて自動的に検出・アクティベートするコンテキストエンジニアリングスキルを含む**Claude Codeプラグインマーケットプレイス**です。

### インストール

**ステップ1：マーケットプレイスの追加**

Claude Codeで以下のコマンドを実行し、このリポジトリをプラグインソースとして登録します：

```
/plugin marketplace add muratcankoylan/Agent-Skills-for-Context-Engineering
```

**ステップ2：ブラウズとインストール**

オプションA - 利用可能なプラグインをブラウズ：
1. `Browse and install plugins` を選択
2. `context-engineering-marketplace` を選択
3. プラグインを選択（例：`context-engineering-fundamentals`、`agent-architecture`）
4. `Install now` を選択

オプションB - コマンドで直接インストール：

```
/plugin install context-engineering-fundamentals@context-engineering-marketplace
/plugin install agent-architecture@context-engineering-marketplace
/plugin install agent-evaluation@context-engineering-marketplace
/plugin install agent-development@context-engineering-marketplace
/plugin install cognitive-architecture@context-engineering-marketplace
```

### 利用可能なプラグイン

| プラグイン | 含まれるスキル |
|--------|-----------------|
| `context-engineering-fundamentals` | context-fundamentals, context-degradation, context-compression, context-optimization |
| `agent-architecture` | multi-agent-patterns, memory-systems, tool-design, filesystem-context, hosted-agents |
| `agent-evaluation` | evaluation, advanced-evaluation |
| `agent-development` | project-development |
| `cognitive-architecture` | bdi-mental-states |

### スキルトリガー

| スキル | トリガー条件 |
|-------|-------------|
| `context-fundamentals` | "understand context", "explain context windows", "design agent architecture" |
| `context-degradation` | "diagnose context problems", "fix lost-in-middle", "debug agent failures" |
| `context-compression` | "compress context", "summarize conversation", "reduce token usage" |
| `context-optimization` | "optimize context", "reduce token costs", "implement KV-cache" |
| `multi-agent-patterns` | "design multi-agent system", "implement supervisor pattern" |
| `memory-systems` | "implement agent memory", "build knowledge graph", "track entities" |
| `tool-design` | "design agent tools", "reduce tool complexity", "implement MCP tools" |
| `filesystem-context` | "offload context to files", "dynamic context discovery", "agent scratch pad", "file-based context" |
| `hosted-agents` | "build background agent", "create hosted coding agent", "sandboxed execution", "multiplayer agent", "Modal sandboxes" |
| `evaluation` | "evaluate agent performance", "build test framework", "measure quality" |
| `advanced-evaluation` | "implement LLM-as-judge", "compare model outputs", "mitigate bias" |
| `project-development` | "start LLM project", "design batch pipeline", "evaluate task-model fit" |
| `bdi-mental-states` | "model agent mental states", "implement BDI architecture", "transform RDF to beliefs", "build cognitive agent" |

<img width="1014" height="894" alt="スクリーンショット 2025-12-26 at 12 34 47 PM" src="https://github.com/user-attachments/assets/f79aaf03-fd2d-4c71-a630-7027adeb9bfe" />

### Cursor & Codex & IDE向け

スキルの内容を `.rules` にコピーするか、プロジェクト固有のSkillsフォルダを作成してください。スキルは、効果的なコンテキストエンジニアリングとエージェント設計のためにエージェントが必要とするコンテキストとガイドラインを提供します。

### カスタム実装向け

任意のスキルから原則とパターンを抽出し、お使いのエージェントフレームワークに実装してください。スキルは意図的にプラットフォーム非依存で設計されています。

## サンプル

[examples](examples/) フォルダには、複数のスキルが実際にどのように連携するかを示す完全なシステム設計が含まれています。

| サンプル | 説明 | 適用スキル |
|---------|-------------|----------------|
| [digital-brain-skill](examples/digital-brain-skill/) | **NEW** 創業者やクリエイター向けのパーソナルオペレーティングシステム。6モジュール、4つの自動化スクリプトを備えた完全なClaude Codeスキル | context-fundamentals, context-optimization, memory-systems, tool-design, multi-agent-patterns, evaluation, project-development |
| [x-to-book-system](examples/x-to-book-system/) | Xアカウントを監視し、日次で合成書籍を生成するマルチエージェントシステム | multi-agent-patterns, memory-systems, context-optimization, tool-design, evaluation |
| [llm-as-judge-skills](examples/llm-as-judge-skills/) | TypeScript実装による本番対応のLLM評価ツール、19件のテスト合格 | advanced-evaluation, tool-design, context-fundamentals, evaluation |
| [book-sft-pipeline](examples/book-sft-pipeline/) | 任意の著者のスタイルで執筆するモデルを訓練。Gertrude Steinのケーススタディを含み、Pangramで70%の人間スコア、総コスト$2 | project-development, context-compression, multi-agent-patterns, evaluation |

各サンプルには以下が含まれます：
- アーキテクチャ決定を含む完全なPRD
- 各決定にどのコンセプトが反映されたかを示すスキルマッピング
- 実装ガイダンス

### Digital Brain Skillサンプル

[digital-brain-skill](examples/digital-brain-skill/) サンプルは、包括的なスキル適用を実演する完全なパーソナルオペレーティングシステムです：

- **段階的開示**：3レベルローディング（SKILL.md → MODULE.md → データファイル）
- **モジュール分離**：6つの独立モジュール（identity, content, knowledge, network, operations, agents）
- **追記専用メモリ**：エージェントフレンドリーなパース用のスキーマファーストラインを持つJSONLファイル
- **自動化スクリプト**：4つの統合ツール（weekly_review, content_ideas, stale_contacts, idea_to_draft）

[HOW-SKILLS-BUILT-THIS.md](examples/digital-brain-skill/HOW-SKILLS-BUILT-THIS.md) に、すべてのアーキテクチャ決定を特定のスキル原則にマッピングした詳細なトレーサビリティが含まれています。

### LLM-as-Judge Skillsサンプル

[llm-as-judge-skills](examples/llm-as-judge-skills/) サンプルは、以下を実演する完全なTypeScript実装です：

- **直接スコアリング**：ルーブリックサポート付きの重み付け基準に対してレスポンスを評価
- **ペアワイズ比較**：位置バイアス緩和を伴うレスポンス比較
- **ルーブリック生成**：ドメイン固有の評価基準を作成
- **EvaluatorAgent**：すべての評価機能を統合するハイレベルエージェント

### Book SFTパイプラインサンプル

[book-sft-pipeline](examples/book-sft-pipeline/) サンプルは、任意の著者のスタイルで執筆するための小型モデル（8B）の訓練を実演します：

- **インテリジェントセグメンテーション**：訓練サンプル最大化のためのオーバーラップ付き2段階チャンキング
- **プロンプト多様性**：暗記を防ぎスタイル学習を促す15以上のテンプレート
- **Tinker統合**：総コスト$2の完全なLoRA訓練ワークフロー
- **検証方法論**：スタイル転移とコンテンツ暗記の違いを証明するモダンなシナリオテスト

コンテキストエンジニアリングスキルとの統合：project-development, context-compression, multi-agent-patterns, evaluation。

## Star履歴
<img width="3664" height="2648" alt="star-history-2026224" src="https://github.com/user-attachments/assets/b3bdbf23-4b6a-4774-ae85-42ef4d9b2d79" />

## 構造

各スキルはAgent Skills仕様に従います：

```
skill-name/
├── SKILL.md              # 必須：指示 + メタデータ
├── scripts/              # 任意：コンセプトを実演する実行可能コード
└── references/           # 任意：追加ドキュメントとリソース
```

正規のスキル構造については [template](template/) フォルダを参照してください。

## コントリビューション

このリポジトリはAgent Skillsオープン開発モデルに従っています。幅広いエコシステムからのコントリビューションを歓迎します。コントリビューション時の注意事項：

1. スキルテンプレート構造に従う
2. 明確で実行可能な指示を提供する
3. 適切な場合は動作するサンプルを含める
4. トレードオフと潜在的な問題点を文書化する
5. 最適なパフォーマンスのためSKILL.mdを500行以下に保つ

コラボレーションの機会やお問い合わせについては、[Muratcan Koylan](https://x.com/koylanai) までお気軽にご連絡ください。

## ライセンス

MIT License - 詳細はLICENSEファイルを参照してください。

## 参考文献

これらのスキルの原則は、主要なAI研究機関やフレームワーク開発者の研究と本番経験から導き出されています。各スキルには、その推奨事項の根拠となる研究やケーススタディへの参照が含まれています。
