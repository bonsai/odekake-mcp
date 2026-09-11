# odekake-mcp

> **ひとりでも、デートでも。**
>
> GitHubに住む、おでかけエージェント。

`odekake-mcp` は、場所・時間・人をつないで「今日のおでかけ」を組み立てる **Agent + Workflow** プロジェクトです。

アプリを中心にするのではなく、**GitHubを作業場・記憶・実行基盤にする**ことを基本方針にします。

## GitHubに住む

```text
                    GitHub
              ┌─────────────────┐
              │ Issues           │ ← 探索依頼
              │ Discussions      │ ← 仮説・相談
              │ Data             │ ← 事実・候補
              │ Actions          │ ← 定期実行
              │ PR               │ ← 検証・レビュー
              └────────┬────────┘
                       │
                 🤖 ODEKAKE AGENT
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       📍 LOCATION    🕐 TIME      👤 PEOPLE
          └────────────┼────────────┘
                       ↓
                 🎛 ORCHESTRATOR
                       ↓
                 🚶 ODEKAKE PLAN
```

GitHubを単なるソースコード置き場ではなく、**おでかけOSのバックエンド**として使います。

- **Issue** = 「こういうおでかけを探して」
- **Agent** = 探す・広げる・正規化する・評価する
- **Workflow** = 定期巡回・更新・検証
- **Data** = 発見した事実を蓄積
- **PR** = Agentと人間のレビュー境界
- **MCP** = 外部AIから呼び出す入口

## Agent model

Agentは一枚岩にしません。探索対象ごとに小さなResearcherを持ち、最後にOrchestratorが組み合わせます。

```text
seed
 ↓
researcher
 ↓
normalize
 ↓
validate
 ↓
commit / PR
 ↓
knowledge graph
 ↓
planner
 ↓
ODEKAKE PLAN
```

### Researcher

- イベントを探す
- 会場を探す
- 公園・店・飲食店を探す
- 出演者・アーティストを探す
- 新しい探索ノードを発見する

### Normalizer

異なるドメインの情報を共通モデルへ変換します。

### Validator

日時、会場、URL、重複、信頼度などを確認します。

### Planner

「どこで・いつ・誰に会えるか」を組み合わせ、移動可能な一日のプランへ変換します。

## Data layers

> **DBを一つにするのではなく、体験を一つにする。**

### 📍 LOCATION

`venues / parks / spots / shops / eatin`

### 🕐 TIME

`events / calendar / opening hours`

### 👤 PEOPLE

`artists / comedians / rakugo / idols / DJs / theatre`

### 🎛 ORCHESTRATOR

3つのレイヤーを横断して、おでかけを生成します。

## Modes

```text
👤 SOLO
  自分のためのおでかけ

👫 DATE
  ふたりのおでかけ

👥 GROUP
  友達・仲間のおでかけ
```

恋愛専用のDateアプリではありません。

**ひとりでも、デートでも、グループでも使えるおでかけOS**です。

## Workflow

GitHub Actionsを反復実行のエンジンにします。

```text
[Schedule / Issue]
       ↓
  discover seeds
       ↓
  expand graph
       ↓
  normalize data
       ↓
  validate facts
       ↓
  update JSONL
       ↓
  open PR
       ↓
  human review
       ↓
  merge
       ↓
  planner sees new knowledge
```

Agentが勝手に本番データを書き換えるのではなく、**PRを境界**にします。

## Connected domain repos

```text
bonsai/odekake-mcp   ← Agent / Workflow / Orchestrator
        │
        ├── bonsai/tokyo-parks
        ├── bonsai/owarai-live
        ├── bonsai/idol-live
        ├── bonsai/dj-event
        └── bonsai/art-event
```

各Repoは専門DB、`odekake-mcp` はそれらを横断する **体験生成層** です。

## Research graph

Seedは入口であって、探索範囲ではありません。

```text
seed
 ↓
venue / person / event
 ↓
related event
 ↓
new venue / person
 ↓
new event
 ↓
...
```

Agentは検索結果を並べるだけではなく、**人物・会場・主催者・場所を次の探索ノード**として扱います。

## First workflows

- 公園を定期探索
- お笑い・落語ライブを探索
- DJイベントを探索
- アイドルライブを探索
- アートイベントを探索
- 会場から出演者を発見
- 出演者から別イベントを発見
- イベントから周辺スポットを発見
- 公園 → カフェ → ライブ → バーの行程を生成

## Workflow contract

```yaml
workflow:
  trigger: schedule | issue | manual
  agent: researcher
  input:
    seed: venue | person | area | date
  steps:
    - discover
    - normalize
    - validate
    - propose
    - review
    - merge
```

Workflowは「コード」だけでなく、**Agentが何をしてよいかを定義する運用契約**です。

## Human in the loop

```text
Agent discovers
      ↓
Candidate data
      ↓
Validation
      ↓
PR
      ↓
Human review
      ↓
Merge
```

大量探索はAgent、人間は最終判断。

## Philosophy

### 1. GitHubを住処にする

Issue、PR、Actions、JSONL、履歴をAgentの作業環境と記憶として使う。

### 2. Seedは広げる

既知の場所だけを検索するのではなく、新しい人物・会場・イベントを探索ノードとして登録する。

### 3. Workflowは習慣をコード化する

「毎朝イベントを探す」「毎週公園を更新する」のような反復作業をActionsへ移す。

### 4. PRは境界

Agentは大量に動き、人間は重要な判断をする。

### 5. 最後に体験へ戻す

最終出力はデータ一覧ではなく、

> **「今日はここへ行こう」**

と言えるおでかけプランです。

## Status

🚧 **Architecture reset — GitHub-native Agent + Workflow foundation**

次の実装では、Agent定義・Workflow定義・共通ContractをRepo内に置き、GitHub Actionsから実行できる最小ループを作ります。
