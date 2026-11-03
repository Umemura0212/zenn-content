---
title: "LangGraphで外側を制御し、内側をループさせるAIエージェント設計"
emoji: "🔁"
type: "tech"
topics: ["langgraph", "ai", "agent", "architecture"]
published: false
---

Anthropicは[Building effective agents](https://www.anthropic.com/research/building-effective-agents)で、**事前に定めた経路を進むWorkflow**と、**LLMが手順やツールを動的に選ぶAgent**を区別しています。前者は予測しやすく、後者は柔軟です。

では、この2つをどう組み合わせればよいのでしょうか。

この記事では、**外側の工程をLangGraphで管理し、内側の作業はAgent Loopに任せる**設計を紹介します。どちらも既存の考え方です。ポイントは、**制御と自律性の境界をどこに置くか**です。

## 外側と内側で役割を分ける

```mermaid
flowchart TD
    A[LangGraph: タスクを渡す] --> B[Agent Loop: 調査・実行・確認・修正]
    B --> C[LangGraph: 成果物を検証]
    C -->|合格| D[次の工程]
    C -->|要修正| B
    C -->|判断が必要| E[人間の承認]
    C -->|上限到達| F[停止・報告]
```

**外側：LangGraph**

工程の進行と判断を管理します。例えば、タスクの割り当て、検証結果による分岐、再試行の上限、人間の承認です。

**内側：Agent Loop**

与えられた目的に向けて、AIが調査・実行・確認・修正を繰り返します。作業の手順を毎回固定する必要はありません。

LangGraph公式の[Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph)でも、処理をノードに分け、共有する状態を設計し、エラーの種類に応じて再試行や人間への確認を行う考え方が示されています。

## なぜ全部をAIに任せないのか

例えば、AIに「不具合を修正して」と依頼したとします。

AIはコードを調べ、修正し、テストを実行します。しかしテストに失敗し続けたら、いつ止めるべきでしょうか。権限変更が必要になったら、誰が承認するのでしょうか。

これらまでAI自身の判断に任せると、**作業の成功条件と、作業を続けてよい条件**が混ざります。

そこで外側では「最大3回まで再試行」「検証に合格したら次へ」「権限変更は承認待ち」といったルールを持たせます。内側では、各試行でどのファイルを調べ、どう修正するかをAIに任せます。

**何を達成するかは外側が決め、どう達成するかは内側に任せる。** この分離が設計の核です。

## 分けることで何が変わるか

外側のグラフには、タスクの状態、検証結果、再試行回数などを保持します。一方、内側のエージェントは作業中の調査結果やツール実行結果を扱います。

これにより、工程全体の進捗や停止理由を追いやすくなります。また、作業担当のモデルやツールを入れ替えても、外側の承認ルールを維持しやすくなります。

ただし、**LangGraphを使うだけで安全になるわけではありません**。検証条件、権限、停止条件は別途設計が必要です。内側の途中経過まで復旧したい場合は、内側にも状態保存の仕組みが要ります。

また、LangGraph自体もループを表現できます。すべての構成で二重のループが必要なわけではありません。小さな処理なら単純なワークフローで十分です。

## まとめ

外側のLangGraphで**工程と制約**を管理し、内側のAgent Loopで**手段の探索と実行**を行う。

これは新しいフレームワークの発明ではなく、既存のWorkflowとAgentを役割分担させる設計です。

**AIの自由度を残しながら、停止・検証・承認を人間が設計できること**に価値があります。

## 参考資料

- [Anthropic — Building effective agents](https://www.anthropic.com/research/building-effective-agents)
- [LangGraph — Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph)
- [LangGraph — Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
