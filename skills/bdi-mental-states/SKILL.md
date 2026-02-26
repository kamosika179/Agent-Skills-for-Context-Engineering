---
name: bdi-mental-states
description: このスキルは、ユーザーが「エージェントのメンタルステートをモデル化する」「BDIアーキテクチャを実装する」「Belief-Desire-Intentionモデルを作成する」「RDFをビリーフに変換する」「認知エージェントを構築する」と依頼した場合、またはBDIオントロジー、メンタルステートモデリング、合理的エージェンシー、ニューロシンボリックAI統合に言及した場合に使用してください。
---

# BDIメンタルステートモデリング

外部RDFコンテキストをエージェントのメンタルステート（ビリーフ、デザイア、インテンション）に変換するために、形式的なBDIオントロジーパターンを使用します。このスキルは、エージェントが認知アーキテクチャを通じてコンテキストについて推論することを可能にし、熟慮的推論、説明可能性、マルチエージェントシステムにおけるセマンティックな相互運用性を支援します。

## アクティベーション条件

以下の場合にこのスキルをアクティベートしてください：
- 外部RDFコンテキストを世界の状態に関するエージェントのビリーフに処理する場合
- 知覚、熟慮、行動のサイクルで合理的エージェンシーをモデル化する場合
- 追跡可能な推論チェーンによる説明可能性を実現する場合
- BDIフレームワーク（SEMAS、JADE、JADEX）を実装する場合
- 形式的な認知構造でLLMを拡張する場合（Logic Augmented Generation）
- マルチエージェントプラットフォーム間でメンタルステートを調整する場合
- ビリーフ、デザイア、インテンションの時間的進化を追跡する場合
- 動機的状態をアクションプランにリンクする場合

## コアコンセプト

### メンタルリアリティアーキテクチャ

**メンタルステート（Endurants）**：永続的な認知属性
- `Belief`：エージェントが世界について真であると信じていること
- `Desire`：エージェントが実現したいと望んでいること
- `Intention`：エージェントが達成にコミットしていること

**メンタルプロセス（Perdurants）**：メンタルステートを変更するイベント
- `BeliefProcess`：知覚からビリーフを形成/更新する
- `DesireProcess`：ビリーフからデザイアを生成する
- `IntentionProcess`：デザイアを実行可能なインテンションとしてコミットする

### 認知チェーンパターン

```turtle
:Belief_store_open a bdi:Belief ;
    rdfs:comment "Store is open" ;
    bdi:motivates :Desire_buy_groceries .

:Desire_buy_groceries a bdi:Desire ;
    rdfs:comment "I desire to buy groceries" ;
    bdi:isMotivatedBy :Belief_store_open .

:Intention_go_shopping a bdi:Intention ;
    rdfs:comment "I will buy groceries" ;
    bdi:fulfils :Desire_buy_groceries ;
    bdi:isSupportedBy :Belief_store_open ;
    bdi:specifies :Plan_shopping .
```

### ワールドステートのグラウンディング

メンタルステートは環境の構造化された設定を参照します：

```turtle
:Agent_A a bdi:Agent ;
    bdi:perceives :WorldState_WS1 ;
    bdi:hasMentalState :Belief_B1 .

:WorldState_WS1 a bdi:WorldState ;
    rdfs:comment "Meeting scheduled at 10am in Room 5" ;
    bdi:atTime :TimeInstant_10am .

:Belief_B1 a bdi:Belief ;
    bdi:refersTo :WorldState_WS1 .
```

### 目標指向プランニング

インテンションは、タスクシーケンスを通じてゴールに対処するプランを指定します：

```turtle
:Intention_I1 bdi:specifies :Plan_P1 .

:Plan_P1 a bdi:Plan ;
    bdi:addresses :Goal_G1 ;
    bdi:beginsWith :Task_T1 ;
    bdi:endsWith :Task_T3 .

:Task_T1 bdi:precedes :Task_T2 .
:Task_T2 bdi:precedes :Task_T3 .
```

## T2B2Tパラダイム

Triples-to-Beliefs-to-Triplesは、RDFナレッジグラフと内部メンタルステート間の双方向フローを実装します：

**フェーズ1：Triples-to-Beliefs**
```turtle
# External RDF context triggers belief formation
:WorldState_notification a bdi:WorldState ;
    rdfs:comment "Push notification: Payment request $250" ;
    bdi:triggers :BeliefProcess_BP1 .

:BeliefProcess_BP1 a bdi:BeliefProcess ;
    bdi:generates :Belief_payment_request .
```

**フェーズ2：Beliefs-to-Triples**
```turtle
# Mental deliberation produces new RDF output
:Intention_pay a bdi:Intention ;
    bdi:specifies :Plan_payment .

:PlanExecution_PE1 a bdi:PlanExecution ;
    bdi:satisfies :Plan_payment ;
    bdi:bringsAbout :WorldState_payment_complete .
```

## レベル別の表記法選択

| C4レベル | 表記法 | メンタルステートの表現 |
|----------|--------|---------------------|
| L1 Context | ArchiMate | エージェント境界、外部知覚ソース |
| L2 Container | ArchiMate | BDI推論エンジン、ビリーフストア、プランエグゼキュータ |
| L3 Component | UML | メンタルステートマネージャー、プロセスハンドラー |
| L4 Code | UML/RDF | Belief/Desire/Intentionクラス、オントロジーインスタンス |

## 正当化と説明可能性

メンタルエンティティは追跡可能な推論のために裏付けるエビデンスにリンクします：

```turtle
:Belief_B1 a bdi:Belief ;
    bdi:isJustifiedBy :Justification_J1 .

:Justification_J1 a bdi:Justification ;
    rdfs:comment "Official announcement received via email" .

:Intention_I1 a bdi:Intention ;
    bdi:isJustifiedBy :Justification_J2 .

:Justification_J2 a bdi:Justification ;
    rdfs:comment "Location precondition satisfied" .
```

## 時間的次元

メンタルステートは限定された時間期間にわたって存続します：

```turtle
:Belief_B1 a bdi:Belief ;
    bdi:hasValidity :TimeInterval_TI1 .

:TimeInterval_TI1 a bdi:TimeInterval ;
    bdi:hasStartTime :TimeInstant_9am ;
    bdi:hasEndTime :TimeInstant_11am .
```

特定の時点でアクティブなメンタルステートをクエリ：

```sparql
SELECT ?mentalState WHERE {
    ?mentalState bdi:hasValidity ?interval .
    ?interval bdi:hasStartTime ?start ;
              bdi:hasEndTime ?end .
    FILTER(?start <= "2025-01-04T10:00:00"^^xsd:dateTime && 
           ?end >= "2025-01-04T10:00:00"^^xsd:dateTime)
}
```

## 構成的メンタルエンティティ

複雑なメンタルエンティティは、選択的な更新のために構成部分に分解されます：

```turtle
:Belief_meeting a bdi:Belief ;
    rdfs:comment "Meeting at 10am in Room 5" ;
    bdi:hasPart :Belief_meeting_time , :Belief_meeting_location .

# Update only location component
:BeliefProcess_update a bdi:BeliefProcess ;
    bdi:modifies :Belief_meeting_location .
```

## 統合パターン

### Logic Augmented Generation（LAG）

オントロジー制約でLLM出力を拡張：

```python
def augment_llm_with_bdi_ontology(prompt, ontology_graph):
    ontology_context = serialize_ontology(ontology_graph, format='turtle')
    augmented_prompt = f"{ontology_context}\n\n{prompt}"
    
    response = llm.generate(augmented_prompt)
    triples = extract_rdf_triples(response)
    
    is_consistent = validate_triples(triples, ontology_graph)
    return triples if is_consistent else retry_with_feedback()
```

### SEMASルール変換

BDIオントロジーを実行可能なプロダクションルールにマッピング：

```prolog
% Belief triggers desire formation
[HEAD: belief(agent_a, store_open)] / 
[CONDITIONALS: time(weekday_afternoon)] » 
[TAIL: generate_desire(agent_a, buy_groceries)].

% Desire triggers intention commitment
[HEAD: desire(agent_a, buy_groceries)] / 
[CONDITIONALS: belief(agent_a, has_shopping_list)] » 
[TAIL: commit_intention(agent_a, buy_groceries)].
```

## ガイドライン

1. ワールドステートをエージェントの視点から独立した設定としてモデル化し、メンタルステートの参照基盤を提供する。

2. Endurants（永続的なメンタルステート）とPerdurants（時間的なメンタルプロセス）を区別し、DOLCEオントロジーと整合させる。

3. ゴールをメンタルステートではなく記述として扱い、認知層とプランニング層の分離を維持する。

4. 選択的なビリーフ更新を可能にするメロニミック構造のために`hasPart`関係を使用する。

5. すべてのメンタルエンティティを`atTime`または`hasValidity`を介して時間的構造に関連付ける。

6. 柔軟なクエリのために双方向プロパティペア（`motivates`/`isMotivatedBy`、`generates`/`isGeneratedBy`）を使用する。

7. 説明可能性と信頼のためにメンタルエンティティを`Justification`インスタンスにリンクする。

8. T2B2Tを以下の手順で実装する：(1) RDFをビリーフに変換、(2) BDI推論を実行、(3) メンタルステートをRDFに射影。

9. メンタルプロセスに存在制限を定義する（例：`BeliefProcess ⊑ ∃generates.Belief`）。

10. 相互運用性のために確立されたODP（EventCore、Situation、TimeIndexedSituation、BasicPlan、Provenance）を再利用する。

## コンピテンシークエスチョン

以下のSPARQLクエリで実装を検証してください：

```sparql
# CQ1: 特定のデザイアの形成を動機づけたビリーフは何か？
SELECT ?belief WHERE {
    :Desire_D1 bdi:isMotivatedBy ?belief .
}

# CQ2: 特定のインテンションはどのデザイアを満たすか？
SELECT ?desire WHERE {
    :Intention_I1 bdi:fulfils ?desire .
}

# CQ3: ビリーフを生成したメンタルプロセスは何か？
SELECT ?process WHERE {
    ?process bdi:generates :Belief_B1 .
}

# CQ4: プラン内のタスクの順序付きシーケンスは何か？
SELECT ?task ?nextTask WHERE {
    :Plan_P1 bdi:hasComponent ?task .
    OPTIONAL { ?task bdi:precedes ?nextTask }
} ORDER BY ?task
```

## アンチパターン

1. **メンタルステートとワールドステートの混同**：メンタルステートはワールドステートを参照するものであり、ワールドステートそのものではありません。

2. **時間的境界の欠如**：すべてのメンタルステートには通時的推論のための有効期間が必要です。

3. **フラットなビリーフ構造**：複雑なビリーフには`hasPart`を使用した構成的モデリングを使用してください。

4. **暗黙的な正当化**：常にメンタルエンティティを明示的な正当化インスタンスにリンクしてください。

5. **インテンションからアクションへの直接マッピング**：インテンションはタスクを含むプランを指定し、アクションはタスクを実行します。

## 統合

- **RDF処理**：外部RDFコンテキストを解析した後に適用し、認知的表現を構築する
- **セマンティック推論**：オントロジー推論と組み合わせて、暗黙的なメンタルステートの関係を推論する
- **マルチエージェント通信**：FIPA ACLと統合し、プラットフォーム間のビリーフ共有を実現する
- **時間的コンテキスト**：メンタルステートの進化のために時間的推論と調整する
- **説明可能なAI**：知覚から熟慮を経て行動までの推論を追跡する説明システムに供給する
- **ニューロシンボリックAI**：LAGパイプラインで適用し、認知構造でLLM出力を制約する

## 参考資料

詳細なドキュメントについては`references/`フォルダを参照してください：
- `bdi-ontology-core.md` - コアオントロジーパターンとクラス定義
- `rdf-examples.md` - 完全なRDF/Turtleの例
- `sparql-competency.md` - 完全なコンピテンシークエスチョンSPARQLクエリ
- `framework-integration.md` - SEMAS、JADE、LAG統合パターン

主要な出典：
- Zuppiroli et al. "The Belief-Desire-Intention Ontology" (2025)
- Rao & Georgeff "BDI agents: From theory to practice" (1995)
- Bratman "Intention, plans, and practical reason" (1987)

