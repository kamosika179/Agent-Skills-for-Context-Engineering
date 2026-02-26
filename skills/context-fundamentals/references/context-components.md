# コンテキストコンポーネント：テクニカルリファレンス

このドキュメントは、エージェントシステムにおける各コンテキストコンポーネントの詳細なテクニカルリファレンスを提供します。

## システムプロンプトエンジニアリング

### セクション構造

システムプロンプトを明確な境界を持つ個別のセクションに整理します。推奨される構造：

```
<BACKGROUND_INFORMATION>
Context about the domain, user preferences, or project-specific details
</BACKGROUND_INFORMATION>

<INSTRUCTIONS>
Core behavioral guidelines and task instructions
</INSTRUCTIONS>

<TOOL_GUIDANCE>
When and how to use available tools
</TOOL_GUIDANCE>

<OUTPUT_DESCRIPTION>
Expected output format and quality standards
</OUTPUT_DESCRIPTION>
```

この構造により、エージェントは関連情報を素早く見つけることができ、高度な実装では選択的なコンテキストローディングが可能になります。

### 抽象度の調整

指示の「抽象度」とは、抽象化のレベルを指します。以下の例を検討してください：

**低すぎる（脆弱）：**
```
If the user asks about pricing, check the pricing table in docs/pricing.md.
If the table shows USD, convert to EUR using the exchange rate in
config/exchange_rates.json. If the user is in the EU, add VAT at the
applicable rate from config/vat_rates.json. Format the response with
the currency symbol, two decimal places, and a note about VAT.
```

**高すぎる（曖昧）：**
```
Help users with pricing questions. Be helpful and accurate.
```

**最適（ヒューリスティック駆動）：**
```
For pricing inquiries:
1. Retrieve current rates from docs/pricing.md
2. Apply user location adjustments (see config/location_defaults.json)
3. Format with appropriate currency and tax considerations

Prefer exact figures over estimates. When rates are unavailable,
say so explicitly rather than projecting.
```

最適な抽象度は、明確な手順を提供しつつ、実行の柔軟性を確保します。

## ツール定義仕様

### スキーマ構造

各ツールは以下を定義する必要があります：

```python
{
    "name": "tool_function_name",
    "description": "Clear description of what the tool does and when to use it",
    "parameters": {
        "type": "object",
        "properties": {
            "param_name": {
                "type": "string",
                "description": "What this parameter controls",
                "default": "reasonable_default_value"
            }
        },
        "required": ["param_name"]
    },
    "returns": {
        "type": "object",
        "description": "What the tool returns and its structure"
    }
}
```

### 説明文の設計

ツールの説明文は、ツールが何をするか、いつ使うか、何を生成するかに答えるべきです。使用コンテキスト、例、エッジケースを含めてください。

**弱い説明文：**
```
Search the database for customer information.
```

**強い説明文：**
```
Retrieve customer information by ID or email.

Use when:
- User asks about a specific customer's details, history, or status
- User provides a customer identifier and needs related information

Returns customer object with:
- Basic info (name, email, account status)
- Order history summary
- Support ticket count

Returns null if customer not found. Returns error if database unreachable.
```

## 取得ドキュメントの管理

### 識別子の設計

意味を伝え、効率的な取得を可能にする識別子を設計します：

**悪い識別子：**
- `data/file1.json`
- `ref/ref.md`
- `2024/q3/report`

**良い識別子：**
- `customer_pricing_rates.json`
- `engineering_onboarding_checklist.md`
- `2024_q3_revenue_report.pdf`

良い識別子により、エージェントは検索ツールがなくても関連ファイルを見つけることができます。

### ドキュメントチャンキング戦略

大きなドキュメントの場合、意味的な一貫性を保つために戦略的にチャンク分割します：

```python
# Pseudocode for semantic chunking
def chunk_document(content):
    """Split document at natural semantic boundaries."""
    boundaries = find_section_headers(content)
    boundaries += find_paragraph_breaks(content)
    boundaries += find_logical_breaks(content)
    
    chunks = []
    for i in range(len(boundaries) - 1):
        chunk = content[boundaries[i]:boundaries[i+1]]
        if len(chunk) > MIN_CHUNK_SIZE and len(chunk) < MAX_CHUNK_SIZE:
            chunks.append(chunk)
    
    return chunks
```

文の途中や概念の途中で分割するような任意の文字数制限は避けてください。

## メッセージ履歴管理

### ターン表現

重要な情報を保持するようにメッセージ履歴を構造化します：

```python
{
    "role": "user" | "assistant" | "tool",
    "content": "message text",
    "reasoning": "optional chain-of-thought",
    "tool_calls": [list if role="assistant"],
    "tool_output": "output if role="tool"",
    "summary": "compact summary if conversation is long"
}
```

### 要約注入パターン

長い会話の場合、一定間隔で要約を注入します：

```python
def inject_summaries(messages, summary_interval=20):
    """Inject summaries at regular intervals to preserve context."""
    summarized = []
    for i, msg in enumerate(messages):
        summarized.append(msg)
        if i > 0 and i % summary_interval == 0:
            summary = generate_summary(summarized[-summary_interval:])
            summarized.append({
                "role": "system",
                "content": f"Conversation summary: {summary}",
                "is_summary": True
            })
    return summarized
```

## ツール出力の最適化

### レスポンスフォーマット

トークン使用量を制御するためのレスポンスフォーマットオプションを提供します：

```python
def get_customer_response_format():
    return {
        "format": "concise | detailed",
        "fields": ["id", "name", "email", "status", "history_summary"]
    }
```

簡潔フォーマットは必須フィールドのみを返し、詳細フォーマットは完全なオブジェクトを返します。

### オブザベーションマスキング

冗長なツール出力に対して、マスキングパターンの検討を推奨します：

```python
def mask_observation(output, max_length=500):
    """Replace long observations with compact references."""
    if len(output) <= max_length:
        return output
    
    reference_id = store_observation(output)
    return f"[Previous observation elided. Full content stored at reference {reference_id}]"
```

これにより、トークン使用量を削減しつつ、情報へのアクセスを保持します。

## コンテキストバジェットの見積もり

### トークン数の概算

計画目的では、英語テキストの場合、1トークンあたり約4文字として見積もります：

```
1000 words ≈ 7500 characters ≈ 1800-2000 tokens
```

これは大まかな概算であり、実際のトークン化はモデルやコンテンツタイプによって異なります。

### コンテキストバジェットの配分

コンポーネント間でコンテキストバジェットを配分します：

| コンポーネント | 一般的な範囲 | 備考 |
|-----------|---------------|-------|
| システムプロンプト | 500-2000 トークン | セッション全体で安定 |
| ツール定義 | ツールあたり 100-500 | ツール数に応じて増加 |
| 取得ドキュメント | 可変 | 最大の消費者となることが多い |
| メッセージ履歴 | 可変 | 会話に応じて増加 |
| ツール出力 | 可変 | コンテキストを支配する可能性あり |

開発中に実際の使用量を監視し、ベースラインの配分を確立してください。

## プログレッシブディスクロージャーの実装

### スキルアクティベーションパターン

```python
def activate_skill_context(skill_name, task_description):
    """Load skill context when task matches skill description."""
    skill_metadata = load_all_skill_metadata()
    
    relevant_skills = []
    for skill in skill_metadata:
        if skill_matches_task(skill, task_description):
            relevant_skills.append(skill)
    
    # Load full content only for most relevant skills
    for skill in relevant_skills[:MAX_CONCURRENT_SKILLS]:
        skill_context = load_skill_content(skill)
        inject_into_context(skill_context)
```

### リファレンスローディングパターン

```python
def get_reference(file_reference):
    """Load reference file only when explicitly needed."""
    if not file_reference.is_loaded:
        file_reference.content = read_file(file_reference.path)
        file_reference.is_loaded = True
    return file_reference.content
```

このパターンにより、ファイルは一度だけ読み込まれ、セッション中キャッシュされます。

