---
name: filesystem-context
description: このスキルは、ユーザーが「コンテキストをファイルにオフロードする」「動的コンテキスト発見を実装する」「エージェントメモリにファイルシステムを使用する」「コンテキストウィンドウの肥大化を削減する」と依頼した場合、またはファイルベースのコンテキスト管理、ツール出力の永続化、エージェントスクラッチパッド、ジャストインタイムのコンテキストローディングに言及した場合に使用します。
---

# ファイルシステムベースのコンテキストエンジニアリング

ファイルシステムは、エージェントが事実上無制限の量のコンテキストを柔軟に保存、取得、更新できる単一のインターフェースを提供します。このパターンは、コンテキストウィンドウが限られている一方で、タスクが単一のウィンドウに収まる以上の情報を必要とすることが多いという根本的な制約に対処します。

核心的な洞察は、ファイルが動的コンテキスト発見を可能にするということです：エージェントはコンテキストウィンドウにすべてを保持するのではなく、必要に応じて関連するコンテキストをオンデマンドで取得します。これは、関連性に関係なく常に含まれる静的コンテキストとは対照的です。

## アクティベーション条件

以下の場合にこのスキルをアクティベートしてください：
- ツール出力がコンテキストウィンドウを肥大化させている場合
- エージェントが長いトラジェクトリにわたって状態を永続化する必要がある場合
- サブエージェントが直接メッセージパッシングなしで情報を共有する必要がある場合
- タスクがウィンドウに収まる以上のコンテキストを必要とする場合
- エージェントが自身の指示を学習・更新するシステムを構築する場合
- 中間結果のためのスクラッチパッドを実装する場合
- ターミナル出力やログをエージェントがアクセスできるようにする必要がある場合

## コアコンセプト

コンテキストエンジニアリングは4つの予測可能な方法で失敗する可能性があります。第一に、エージェントが必要とするコンテキストが利用可能な総コンテキストに含まれていない場合。第二に、取得されたコンテキストが必要なコンテキストを十分にカプセル化できない場合。第三に、取得されたコンテキストが必要なコンテキストを大幅に超え、トークンを浪費しパフォーマンスを低下させる場合。第四に、エージェントが多数のファイルに埋もれたニッチな情報を発見できない場合。

ファイルシステムはこれらの障害に対処するために、エージェントが一度書き込み選択的に読み取る永続レイヤーを提供し、検索ツールを通じて特定の情報を取得する能力を維持しながら大量のコンテンツをオフロードします。

## 詳細トピック

### 静的コンテキスト vs 動的コンテキストのトレードオフ

**静的コンテキスト**
静的コンテキストは常にプロンプトに含まれます：システム指示、ツール定義、重要なルール。静的コンテキストはタスクの関連性に関係なくトークンを消費します。エージェントがより多くの機能（ツール、スキル、指示）を蓄積するにつれ、静的コンテキストが増大し、動的情報のためのスペースを圧迫します。

**動的コンテキスト発見**
動的コンテキストは現在のタスクに関連する場合にオンデマンドでロードされます。エージェントは最小限の静的ポインター（名前、記述、ファイルパス）を受け取り、必要に応じて検索ツールを使用して完全なコンテンツをロードします。

動的発見は必要なデータのみがコンテキストウィンドウに入るため、トークン効率が高くなります。また、混乱や矛盾する可能性のある情報を減らすことで、レスポンスの品質を向上させることもできます。

トレードオフ：動的発見はモデルが追加コンテキストをロードすべきタイミングを正しく識別する必要があります。これは現在のフロンティアモデルではうまく機能しますが、追加情報が必要な場合にそれを認識できない能力の低いモデルでは失敗する可能性があります。

### パターン1：スクラッチパッドとしてのファイルシステム

**問題**
ツール呼び出しは大量の出力を返すことがあります。ウェブ検索は10kトークンの生のコンテンツを返す場合があります。データベースクエリは数百行を返す場合があります。このコンテンツがメッセージ履歴に入ると、会話全体にわたって残り、トークンコストを増大させ、より関連性の高い情報への注意を低下させる可能性があります。

**解決策**
大きなツール出力をコンテキストに直接返すのではなく、ファイルに書き込みます。その後、エージェントはターゲットを絞った取得（grep、行指定の読み取り）を使用して関連部分のみを抽出します。

**実装**
```python
def handle_tool_output(output: str, threshold: int = 2000) -> str:
    if len(output) < threshold:
        return output
    
    # Write to scratch pad
    file_path = f"scratch/{tool_name}_{timestamp}.txt"
    write_file(file_path, output)
    
    # Return reference instead of content
    key_summary = extract_summary(output, max_tokens=200)
    return f"[Output written to {file_path}. Summary: {key_summary}]"
```

エージェントは`grep`を使用して特定のパターンを検索したり、行範囲を指定した`read_file`を使用してターゲットセクションを取得できます。

**メリット**
- 長い会話でのトークン蓄積を削減
- 後で参照するための完全な出力を保存
- すべてを保持するのではなく、ターゲットを絞った取得を可能にする

### パターン2：プラン永続化

**問題**
長期的なタスクではエージェントがプランを立て、それに従う必要があります。しかし、会話が延長されると、プランが注意から外れたり、要約によって失われたりする可能性があります。エージェントは何をすべきかを見失います。

**解決策**
プランをファイルシステムに書き込みます。エージェントはいつでもプランを再読み込みでき、現在の目標と進捗を思い出させることができます。これは「暗唱による注意の操作」と呼ばれることもあります。

**実装**
構造化されたフォーマットでプランを保存：
```yaml
# scratch/current_plan.yaml
objective: "Refactor authentication module"
status: in_progress
steps:
  - id: 1
    description: "Audit current auth endpoints"
    status: completed
  - id: 2
    description: "Design new token validation flow"
    status: in_progress
  - id: 3
    description: "Implement and test changes"
    status: pending
```

エージェントは各ターンの開始時または方向を再確認する必要がある場合にこのファイルを読み取ります。

### パターン3：ファイルシステムを介したサブエージェント間通信

**問題**
マルチエージェントシステムでは、サブエージェントは通常、メッセージパッシングを通じてコーディネーターエージェントに調査結果を報告します。これにより、各ホップでの要約によって情報が劣化する「伝言ゲーム」が発生します。

**解決策**
サブエージェントが調査結果をファイルシステムに直接書き込みます。コーディネーターはこれらのファイルを直接読み取り、中間のメッセージパッシングをバイパスします。これにより忠実性が保たれ、コーディネーターでのコンテキスト蓄積が削減されます。

**実装**
```
workspace/
  agents/
    research_agent/
      findings.md        # Research agent writes here
      sources.jsonl      # Source tracking
    code_agent/
      changes.md         # Code agent writes here
      test_results.txt   # Test output
  coordinator/
    synthesis.md         # Coordinator reads agent outputs, writes synthesis
```

各エージェントは相対的に独立して動作しますが、ファイルシステムを通じて状態を共有します。

### パターン4：動的スキルローディング

**問題**
エージェントは多数のスキルや指示セットを持つ場合がありますが、特定のタスクに対してほとんどは無関係です。すべての指示をシステムプロンプトに詰め込むとトークンを浪費し、矛盾する無関係なガイダンスでモデルを混乱させる可能性があります。

**解決策**
スキルをファイルとして保存します。静的コンテキストにはスキル名と簡単な記述のみを含めます。エージェントはタスクが必要とする場合に検索ツールを使用して関連するスキルコンテンツをロードします。

**実装**
静的コンテキストに含めるもの：
```
Available skills (load with read_file when relevant):
- database-optimization: Query tuning and indexing strategies
- api-design: REST/GraphQL best practices
- testing-strategies: Unit, integration, and e2e testing patterns
```

エージェントはデータベースタスクに取り組む場合にのみ`skills/database-optimization/SKILL.md`をロードします。

### パターン5：ターミナルとログの永続化

**問題**
長時間実行プロセスからのターミナル出力は急速に蓄積されます。出力をコピー＆ペーストしてエージェント入力に渡すのは手動で非効率的です。

**解決策**
ターミナル出力を自動的にファイルに同期します。エージェントはターミナル履歴全体をロードすることなく、関連するセクション（エラーメッセージ、特定のコマンド）をgrepで検索できます。

**実装**
ターミナルセッションはファイルとして永続化されます：
```
terminals/
  1.txt    # Terminal session 1 output
  2.txt    # Terminal session 2 output
```

エージェントはターゲットを絞ったgrepでクエリします：
```bash
grep -A 5 "error" terminals/1.txt
```

### パターン6：自己変更による学習

**問題**
エージェントはインタラクション中にユーザーが暗黙的または明示的に提供するコンテキストを欠いていることがよくあります。従来、これにはセッション間での手動のシステムプロンプト更新が必要でした。

**解決策**
エージェントが学習した情報を自身の指示ファイルに書き込みます。以降のセッションでこれらのファイルをロードし、学習したコンテキストを自動的に組み込みます。

**実装**
ユーザーの好みが提供された後：
```python
def remember_preference(key: str, value: str):
    preferences_file = "agent/user_preferences.yaml"
    prefs = load_yaml(preferences_file)
    prefs[key] = value
    write_yaml(preferences_file, prefs)
```

以降のセッションでは、ファイルが存在する場合にユーザーの好みをロードするステップが含まれます。

**注意**
このパターンはまだ発展途上です。自己変更には、エージェントが時間の経過とともに不正確または矛盾する指示を蓄積するのを防ぐための慎重なガードレールが必要です。

### ファイルシステム検索テクニック

モデルはファイルシステムのトラバーサルを理解するよう特別に訓練されています。`ls`、`glob`、`grep`、行範囲を指定した`read_file`の組み合わせにより、強力なコンテキスト発見が可能になります：

- `ls` / `list_dir`：ディレクトリ構造の発見
- `glob`：パターンに一致するファイルの検索（例：`**/*.py`）
- `grep`：パターンに対するファイルコンテンツの検索、一致する行を返す
- 範囲指定の`read_file`：ファイル全体をロードせずに特定の行範囲を読み取る

この組み合わせは、意味的な意味が希薄で構造的なパターンが明確な技術コンテンツ（コード、APIドキュメント）に対して、セマンティック検索を上回ることがよくあります。

セマンティック検索とファイルシステム検索は相補的に機能します：概念的なクエリにはセマンティック検索、構造的および完全一致のクエリにはファイルシステム検索。

## 実践的ガイダンス

### ファイルシステムコンテキストの使用タイミング

**ファイルシステムパターンを使用する場合：**
- ツール出力が2000トークンを超える場合
- タスクが複数の会話ターンにまたがる場合
- 複数のエージェントが状態を共有する必要がある場合
- スキルや指示がシステムプロンプトに快適に収まらない場合
- ログやターミナル出力の選択的クエリが必要な場合

**ファイルシステムパターンを避ける場合：**
- タスクが単一ターンで完了する場合
- コンテキストがウィンドウに快適に収まる場合
- レイテンシーが重要な場合（ファイルI/Oがオーバーヘッドを追加）
- ファイルシステムツールの使用能力がない単純なモデルの場合

### ファイル整理

発見可能性のためにファイルを構造化：
```
project/
  scratch/           # Temporary working files
    tool_outputs/    # Large tool results
    plans/           # Active plans and checklists
  memory/            # Persistent learned information
    preferences.yaml # User preferences
    patterns.md      # Learned patterns
  skills/            # Loadable skill definitions
  agents/            # Sub-agent workspaces
```

一貫した命名規則を使用してください。曖昧さを排除するためにスクラッチファイルにタイムスタンプやIDを含めてください。

### トークン会計

トークンの発生源を追跡：
- 静的コンテキストと動的コンテキストの比率を測定
- オフロード前後のツール出力サイズを監視
- 動的コンテキストが実際にロードされる頻度を追跡

想定ではなく、測定に基づいて最適化してください。

## 例

**例1：ツール出力のオフロード**
```
Input: Web search returns 8000 tokens
Before: 8000 tokens added to message history
After: 
  - Write to scratch/search_results_001.txt
  - Return: "[Results in scratch/search_results_001.txt. Key finding: API rate limit is 1000 req/min]"
  - Agent greps file when needing specific details
Result: ~100 tokens in context, 8000 tokens accessible on demand
```

**例2：動的スキルローディング**
```
Input: User asks about database indexing
Static context: "database-optimization: Query tuning and indexing"
Agent action: read_file("skills/database-optimization/SKILL.md")
Result: Full skill loaded only when relevant
```

**例3：ファイル参照としてのチャット履歴**
```
Trigger: Context window limit reached, summarization required
Action: 
  1. Write full history to history/session_001.txt
  2. Generate summary for new context window
  3. Include reference: "Full history in history/session_001.txt"
Result: Agent can search history file to recover details lost in summarization
```

## ガイドライン

1. 大きな出力をファイルに書き込み、コンテキストにはサマリーと参照を返す
2. 再読み込み用に構造化されたファイルにプランと状態を保存する
3. メッセージチェーンの代わりにサブエージェントのファイルワークスペースを使用する
4. すべてをシステムプロンプトに詰め込むのではなく、スキルを動的にロードする
5. ターミナルとログ出力を検索可能なファイルとして永続化する
6. 包括的な発見のためにgrep/globとセマンティック検索を組み合わせる
7. 明確な命名でエージェントの発見可能性のためにファイルを整理する
8. ファイルシステムパターンが効果的であることを検証するためにトークン削減を測定する
9. 無制限な増大を防ぐためにスクラッチファイルのクリーンアップを実装する
10. 自己変更パターンにはバリデーションによるガードを設ける

## 統合

このスキルは以下と関連します：

- context-optimization - ファイルシステムオフロードはオブザベーションマスキングの一形態
- memory-systems - ファイルシステムをメモリとして使用するのはシンプルなメモリレイヤー
- multi-agent-patterns - サブエージェントのファイルワークスペースが分離を可能にする
- context-compression - ファイル参照がロスレスの「圧縮」を可能にする
- tool-design - ツールは大きな出力に対してファイル参照を返すべき

## 参考資料

内部参考資料：
- [実装パターン](./references/implementation-patterns.md) - 詳細なパターン実装

このコレクション内の関連スキル：
- context-optimization - トークン削減テクニック
- memory-systems - 永続ストレージパターン
- multi-agent-patterns - エージェント連携

外部リソース：
- LangChain Deep Agents：エージェントがコンテキストエンジニアリングにファイルシステムを使用する方法
- Cursor：動的コンテキスト発見パターン
- Anthropic：Agent Skills仕様

---

## スキルメタデータ

**作成日**: 2026-01-07
**最終更新日**: 2026-01-07
**著者**: Agent Skills for Context Engineering Contributors
**バージョン**: 1.0.0

