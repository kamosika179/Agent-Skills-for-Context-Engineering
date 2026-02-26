# コンテキスト最適化リファレンス

このドキュメントは、コンテキスト最適化のテクニックと戦略に関する詳細なテクニカルリファレンスを提供します。

## コンパクション戦略

### 要約ベースのコンパクション

要約ベースのコンパクションは、重要な情報を保持しながら冗長なコンテンツを簡潔な要約に置き換えます。このアプローチは、圧縮可能なセクションを特定し、本質的なポイントを捉えた要約を生成し、完全なコンテンツを要約に置き換えることで機能します。

コンパクションの効果は、どの情報が保持されるかに依存します。重要な意思決定、ユーザーの好み、および現在のタスク状態は決してコンパクションすべきではありません。中間結果や補足的な証拠はより積極的に要約できます。ボイラープレート、繰り返し情報、探索的な推論は完全に削除できることが多いです。

### トークンバジェットの配分

効果的なコンテキストバジェット管理には、異なるコンテキストコンポーネントがどのようにトークンを消費するかを理解し、戦略的にバジェットを配分することが必要です：

| コンポーネント | 一般的な範囲 | 備考 |
|-----------|---------------|-------|
| システムプロンプト | 500-2000 トークン | セッション全体で安定 |
| ツール定義 | ツールあたり 100-500 | ツール数に応じて増加 |
| 取得ドキュメント | 可変 | 最大の消費者となることが多い |
| メッセージ履歴 | 可変 | 会話に応じて増加 |
| ツール出力 | 可変 | コンテキストを支配する可能性あり |

### コンパクション閾値

パフォーマンスを維持するために適切な閾値でコンパクションをトリガーします：

- 有効コンテキスト上限の70%で警告閾値
- 有効コンテキスト上限の80%でコンパクショントリガー
- 有効コンテキスト上限の90%で積極的コンパクション

正確な閾値はモデルの動作とタスクの特性に依存します。グレースフルデグラデーションを示すモデルもあれば、急激なパフォーマンス低下を示すモデルもあります。

## オブザベーションマスキングパターン

### 選択的マスキング

すべてのオブザベーションを同等にマスキングすべきではありません。目的を果たし、アクティブな推論に不要になったオブザベーションのマスキングを検討してください。現在のタスクの中心となるオブザベーションは保持してください。最新のターンからのオブザベーションは保持してください。再度参照される可能性のあるオブザベーションは保持してください。

### マスキングの実装

```python
def selective_mask(observations: List[Dict], current_task: Dict) -> List[Dict]:
    """
    Selectively mask observations based on relevance.
    
    Returns observations with mask field indicating masked content.
    """
    masked = []
    
    for obs in observations:
        relevance = calculate_relevance(obs, current_task)
        
        if relevance < 0.3 and obs["age"] > 3:
            # Low relevance and old - mask
            masked.append({
                **obs,
                "masked": True,
                "reference": store_for_reference(obs["content"]),
                "summary": summarize_content(obs["content"])
            })
        else:
            masked.append({
                **obs,
                "masked": False
            })
    
    return masked
```

## KVキャッシュ最適化

### プレフィックスの安定性

KVキャッシュのヒット率はプレフィックスの安定性に依存します。安定したプレフィックスはリクエスト間でのキャッシュの再利用を可能にします。動的なプレフィックスはキャッシュを無効にし、再計算を強制します。

安定すべき要素にはシステムプロンプト、ツール定義、頻繁に使用されるテンプレートが含まれます。変動し得る要素にはタイムスタンプ、セッション識別子、クエリ固有のコンテンツが含まれます。

### キャッシュフレンドリーな設計

キャッシュヒット率を最大化するようにプロンプトを設計します：

1. 安定したコンテンツを先頭に配置する
2. リクエスト間で一貫したフォーマットを使用する
3. 可能な限りプロンプト内の動的コンテンツを避ける
4. 動的コンテンツにはプレースホルダーを使用する

```python
# Cache-unfriendly: Dynamic timestamp in prompt
system_prompt = f"""
Current time: {datetime.now().isoformat()}
You are a helpful assistant.
"""

# Cache-friendly: Stable prompt with dynamic time as variable
system_prompt = """
You are a helpful assistant.
Current time is provided separately when relevant.
"""
```

## コンテキストパーティショニング戦略

### サブエージェントの分離

単一のコンテキストが大きくなりすぎることを防ぐために、サブエージェント間で作業を分割します。各サブエージェントは、そのサブタスクに焦点を当てたクリーンなコンテキストで動作します。

### パーティション計画

```python
def plan_partitioning(task: Dict, context_limit: int) -> Dict:
    """
    Plan how to partition a task based on context limits.
    
    Returns partitioning strategy and subtask definitions.
    """
    estimated_context = estimate_task_context(task)
    
    if estimated_context <= context_limit:
        return {
            "strategy": "single_agent",
            "subtasks": [task]
        }
    
    # Plan multi-agent approach
    subtasks = decompose_task(task)
    
    return {
        "strategy": "multi_agent",
        "subtasks": subtasks,
        "coordination": "hierarchical"
    }
```

## 最適化の意思決定フレームワーク

### いつ最適化するか

コンテキスト使用率が70%を超えた場合、会話が長くなるにつれてレスポンス品質が低下する場合、長いコンテキストによりコストが増加する場合、または会話の長さに応じてレイテンシが増加する場合に、コンテキスト最適化を検討してください。

### どの最適化を適用するか

コンテキストの構成に基づいて最適化戦略を選択します：

ツール出力がコンテキストを支配している場合は、オブザベーションマスキングを適用します。取得ドキュメントがコンテキストを支配している場合は、要約またはパーティショニングを適用します。メッセージ履歴がコンテキストを支配している場合は、要約付きコンパクションを適用します。複数のコンポーネントが寄与している場合は、戦略を組み合わせます。

### 最適化の評価

最適化を適用した後、効果を評価します：

- 達成されたトークン削減量を測定する
- 品質の保持を測定する（出力品質が低下すべきではない）
- レイテンシの改善を測定する
- コスト削減を測定する

評価結果に基づいて最適化戦略を反復します。

## よくある落とし穴

### 過度なコンパクション

過度に積極的なコンパクションは、重要な情報を削除する可能性があります。タスクの目標、ユーザーの好み、および最近の会話コンテキストを常に保持してください。最適なバランスを見つけるために、積極性のレベルを段階的に上げてコンパクションをテストしてください。

### 重要なオブザベーションのマスキング

まだ必要なオブザベーションをマスキングするとエラーが発生する可能性があります。オブザベーションの使用状況を追跡し、参照されなくなったコンテンツのみをマスキングしてください。必要に応じて取得できるよう、マスキングされたコンテンツへの参照を保持することを検討してください。

### アテンション分布の無視

Lost-in-Middle 現象は、情報の配置が重要であることを意味します。重要な情報をアテンションが優位な位置（コンテキストの先頭と末尾）に配置してください。明示的なマーカーを使用して重要なコンテンツを強調してください。

### 早すぎる最適化

すべてのコンテキストが最適化を必要とするわけではありません。最適化の仕組みを追加するとオーバーヘッドが発生します。コンテキストの制限が実際にエージェントのパフォーマンスを制約している場合にのみ最適化してください。

## モニタリングとアラート

### 主要メトリクス

最適化のニーズを理解するために以下のメトリクスを追跡します：

- 時間経過によるコンテキストトークン数
- 繰り返しパターンのキャッシュヒット率
- コンテキストサイズ別のレスポンス品質メトリクス
- コンテキスト長別の会話あたりコスト
- コンテキストサイズ別のレイテンシ

### アラート閾値

以下のアラートを設定します：

- コンテキスト使用率が80%超
- キャッシュヒット率が50%未満
- 品質スコアが10%以上低下
- コストがベースラインを超えて増加

## インテグレーションパターン

### エージェントフレームワークとの統合

最適化をエージェントのワークフローに統合します：

```python
class OptimizingAgent:
    def __init__(self, context_limit: int = 80000):
        self.context_limit = context_limit
        self.optimizer = ContextOptimizer()
    
    def process(self, user_input: str, context: Dict) -> Dict:
        # Check if optimization needed
        if self.optimizer.should_compact(context):
            context = self.optimizer.compact(context)
        
        # Process with optimized context
        response = self._call_model(user_input, context)
        
        # Track metrics
        self.optimizer.record_metrics(context, response)
        
        return response
```

### メモリシステムとの統合

最適化をメモリシステムと接続します：

```python
class MemoryAwareOptimizer:
    def __init__(self, memory_system, context_limit: int):
        self.memory = memory_system
        self.limit = context_limit
    
    def optimize_context(self, current_context: Dict, task: str) -> Dict:
        # Check if information is in memory
        relevant_memories = self.memory.retrieve(task)
        
        # Move information to memory if not needed in context
        for mem in relevant_memories:
            if mem["importance"] < threshold:
                current_context = remove_from_context(current_context, mem)
                # Keep reference that memory can be retrieved
        
        return current_context
```

## パフォーマンスベンチマーク

### コンパクションのパフォーマンス

コンパクションは品質を保持しながらトークン数を削減すべきです。目標：

- 積極的コンパクションで50-70%のトークン削減
- コンパクションによる品質低下は5%未満
- コンパクションのオーバーヘッドによるレイテンシ増加は10%未満

### マスキングのパフォーマンス

オブザベーションマスキングはトークン数を大幅に削減すべきです：

- マスキングされたオブザベーションで60-80%の削減
- マスキングによる品質への影響は2%未満
- レイテンシのオーバーヘッドはほぼゼロ

### キャッシュのパフォーマンス

KVキャッシュ最適化はコストとレイテンシを改善すべきです：

- 安定したワークロードで70%以上のキャッシュヒット率
- キャッシュヒットによる50%以上のコスト削減
- キャッシュヒットによる40%以上のレイテンシ削減

