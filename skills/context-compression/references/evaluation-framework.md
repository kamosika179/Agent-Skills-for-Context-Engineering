# コンテキスト圧縮評価フレームワーク

このドキュメントは、コンテキスト圧縮の品質を測定するための完全な評価フレームワークを提供します。プローブタイプ、スコアリングルーブリック、LLM ジャッジの設定を含みます。

## プローブタイプ

### リコールプローブ

会話履歴から特定の詳細の事実保持をテストします。

**構造：**
```
Question: [Ask for specific fact from truncated history]
Expected: [Exact detail that should be preserved]
Scoring: Match accuracy of technical details
```

**例：**
- 「このデバッグセッションのきっかけとなった元のエラーメッセージは何でしたか？」
- 「使用することに決めた依存関係のバージョンは何でしたか？」
- 「失敗した正確なコマンドは何でしたか？」

### アーティファクトプローブ

ファイルの追跡と変更の認識をテストします。

**構造：**
```
Question: [Ask about files created, modified, or examined]
Expected: [Complete list with change descriptions]
Scoring: Completeness of file list and accuracy of change descriptions
```

**例：**
- 「どのファイルを変更しましたか？各ファイルで何が変わったか説明してください。」
- 「このセッションでどのような新しいファイルを作成しましたか？」
- 「どの設定ファイルを確認しましたが変更しませんでしたか？」

### 継続プローブ

コンテキストを再取得せずに作業を続行する能力をテストします。

**構造：**
```
Question: [Ask about next steps or current state]
Expected: [Actionable next steps based on session history]
Scoring: Ability to continue without requesting re-read of files
```

**例：**
- 「次に何をすべきですか？」
- 「まだ失敗しているテストはどれで、その理由は何ですか？」
- 「前のステップで未完了だったものは何ですか？」

### 意思決定プローブ

推論チェーンと意思決定の根拠の保持をテストします。

**構造：**
```
Question: [Ask about why a decision was made]
Expected: [Reasoning that led to the decision]
Scoring: Preservation of decision context and alternatives considered
```

**例：**
- 「Redis の問題についてオプションを議論しました。何を決定し、その理由は何ですか？」
- 「リクエストごとの接続ではなくコネクションプーリングを選んだ理由は何ですか？」
- 「認証の修正にどのような代替案を検討しましたか？」

## スコアリングルーブリック

### 正確性ディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| accuracy_factual | 事実、ファイルパス、技術的な詳細は正しいか？ | 完全に不正確または捏造 | おおむね正確で軽微なエラーあり | 完全に正確 |
| accuracy_technical | コード参照と技術的概念は正しいか？ | 重大な技術的エラー | おおむね正確で軽微な問題あり | 技術的に正確 |

### コンテキスト認識ディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| context_conversation_state | レスポンスは現在の会話状態を反映しているか？ | 以前のコンテキストの認識なし | 一般的な認識はあるがギャップあり | 会話履歴の完全な認識 |
| context_artifact_state | レスポンスはどのファイル/アーティファクトにアクセスしたかを反映しているか？ | アーティファクトの認識なし | 部分的なアーティファクト認識 | 完全なアーティファクト状態の認識 |

### アーティファクトトレイルディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| artifact_files_created | エージェントはどのファイルが作成されたか知っているか？ | 知識なし | ほとんどのファイルを把握 | 完全な知識 |
| artifact_files_modified | エージェントはどのファイルが変更され、何が変わったか知っているか？ | 知識なし | ほとんどの変更の良好な知識 | すべての変更の完全な知識 |
| artifact_key_details | エージェントは関数名、変数名、エラーメッセージを覚えているか？ | 想起なし | ほとんどの重要な詳細を想起 | 完全な想起 |

### 完全性ディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| completeness_coverage | レスポンスは質問のすべての部分に対応しているか？ | ほとんどの部分を無視 | ほとんどの部分に対応 | すべての部分に徹底的に対応 |
| completeness_depth | 十分な詳細が提供されているか？ | 表面的または詳細の欠如 | 適切な詳細 | 包括的な詳細 |

### 継続性ディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| continuity_work_state | エージェントは以前アクセスした情報を再取得せずに続行できるか？ | すべてのコンテキストの再取得なしには続行不可 | 最小限の再取得で続行可能 | シームレスに続行可能 |
| continuity_todo_state | エージェントは保留中のタスクの認識を維持しているか？ | すべての TODO を見失った | おおむね認識しているがギャップあり | 完全なタスク認識 |
| continuity_reasoning | エージェントは以前の決定の根拠を保持しているか？ | 推論の記憶なし | おおむね推論を覚えている | 優れた保持 |

### 指示遵守ディメンション

| 基準 | 質問 | スコア 0 | スコア 3 | スコア 5 |
|-----------|----------|---------|---------|---------|
| instruction_format | レスポンスは要求されたフォーマットに従っているか？ | フォーマットを無視 | おおむねフォーマットに従う | 完全にフォーマットに従う |
| instruction_constraints | レスポンスは記載された制約を尊重しているか？ | 制約を無視 | おおむね制約を尊重 | すべての制約を完全に尊重 |

## LLM ジャッジの設定

### システムプロンプト

```
You are an expert evaluator assessing AI assistant responses in software development conversations.

Your task is to grade responses against specific rubric criteria. For each criterion:
1. Read the criterion question carefully
2. Examine the response for evidence
3. Assign a score from 0-5 based on the scoring guide
4. Provide brief reasoning for your score

Be objective and consistent. Focus on what is present in the response, not what could have been included.
```

### ジャッジ入力フォーマット

```json
{
  "probe_question": "What was the original error message?",
  "model_response": "[Response to evaluate]",
  "compacted_context": "[The compressed context that was provided]",
  "ground_truth": "[Optional: known correct answer]",
  "rubric_criteria": ["accuracy_factual", "accuracy_technical", "context_conversation_state"]
}
```

### ジャッジ出力フォーマット

```json
{
  "criterionResults": [
    {
      "criterionId": "accuracy_factual",
      "score": 5,
      "reasoning": "Response correctly identifies the 401 error, specific endpoint, and root cause."
    }
  ],
  "aggregateScore": 4.8,
  "dimensionScores": {
    "accuracy": 4.9,
    "context_awareness": 4.5,
    "artifact_trail": 3.2,
    "completeness": 5.0,
    "continuity": 4.8,
    "instruction_following": 5.0
  }
}
```

## ベンチマーク結果リファレンス

圧縮方式間のパフォーマンス（36,000件以上のメッセージに基づく）：

| 方式 | 総合 | 正確性 | コンテキスト | アーティファクト | 完全性 | 継続性 | 指示遵守 |
|--------|---------|----------|---------|----------|----------|------------|-------------|
| Anchored Iterative | 3.70 | 4.04 | 4.01 | 2.45 | 4.44 | 3.80 | 4.99 |
| Regenerative | 3.44 | 3.74 | 3.56 | 2.33 | 4.37 | 3.67 | 4.95 |
| Opaque | 3.35 | 3.43 | 3.64 | 2.19 | 4.37 | 3.77 | 4.92 |

**主な知見：**

1. **正確性のギャップ**：最良と最悪の方式間で0.61ポイント
2. **コンテキスト認識のギャップ**：0.45ポイント、Anchored Iterative が優位
3. **アーティファクトトレイル**：全体的に弱い（2.19-2.45）、専門的な対応が必要
4. **完全性と指示遵守**：差異は最小限

## 統計的考慮事項

- 0.26-0.35ポイントの差異は、タスクタイプとセッション長にわたって一貫している
- パターンは短いセッションと長いセッションの両方で維持される
- パターンはデバッグ、機能実装、コードレビューのタスク全体で維持される
- サンプルサイズ：数百の圧縮ポイントにわたる36,611メッセージ

## 実装に関する注意事項

### プローブ生成

各圧縮ポイントで切り詰められた履歴に基づいてプローブを生成します：
1. リコールプローブ用に事実の主張を抽出する
2. アーティファクトプローブ用にファイル操作を抽出する
3. 継続プローブ用に未完了タスクを抽出する
4. 意思決定プローブ用に意思決定ポイントを抽出する

### 採点プロセス

1. プローブの質問＋モデルのレスポンス＋圧縮されたコンテキストをジャッジに入力する
2. ルーブリックの各基準に対して評価する
3. スコアと理由を含む構造化されたJSONを出力する
4. ディメンションスコアを加重平均として計算する
5. 総合スコアをディメンションの非加重平均として計算する

### ブラインド化

ジャッジは、評価対象のレスポンスがどの圧縮方式で生成されたかを知るべきではありません。これにより、既知の方式に対するバイアスを防ぎます。

