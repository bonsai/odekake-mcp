# date-mcp

**FE オーケストレーターとして、おでかけ全体を組み立てる MCP。**

date-mcp 自身がイベントや店の巨大DBを持つのではなく、各ドメインを **BE / DB / Researcher** として分離し、FE オーケストレーターが横断して「今日どこへ行く？」を組み立てる。

## Architecture

```text
                         date-mcp
                    🎛 FE Orchestrator
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       🎭 BE/DB          🌳 BE/DB         ☕ BE/DB
      owarai-live      tokyo-parks       places
          │                │                │
      🔎 Researcher     🔎 Researcher     🔎 Researcher
          │                │                │
          └────────────────┼────────────────┘
                           │
                 🧩 Normalize / Rank
                           │
                    📅 Date Plan
                           │
              🚶 Event + Park + Food + Bar
```

## Responsibility

### date-mcp — FE / Orchestrator

ユーザーから見える「おでかけ体験」を担当する。

- 日付・エリア・ジャンルを受け取る
- 各ドメインBEへ問い合わせる
- 候補を共通形式へ正規化する
- 距離・時間・順番でランキングする
- ライブを中心に前後の行動を組み立てる
- 最終的な `GO OUT PLAN` を返す

**date-mcp はデータの所有者ではなく、データを組み合わせる司令塔。**

## Domain BE / DB / Researcher

各ドメインは独立して育てる。

| Domain | BE / DB | Researcher | 主なデータ |
|---|---|---|---|
| 🎭 お笑い・落語 | `owarai-live` | performance researcher | 芸人・落語家・ライブ・会場 |
| 🎤 アイドル | `idol-live` | idol researcher | アイドル・ライブ・会場 |
| 🎧 DJ | `dj-event` | DJ researcher | DJ・イベント・会場 |
| 🎨 アート | `art-event` | art researcher | 展示・アートイベント・会場 |
| 🌳 公園 | `tokyo-parks` | park researcher | 公園・散歩・場所 |
| ☕ 店 | places domain | place researcher | カフェ・ランチ・バー |
| 🎭 演劇 | theatre domain | theatre researcher | 劇場・演劇・出演者 |

各 Researcher は「発見」を担当し、BE/DB は「正規化されたデータ」を担当する。

```text
Researcher
    ↓ discover
candidate
    ↓ verify
BE
    ↓ normalize
DB
    ↓ query
FE Orchestrator
```

## Core flow

```text
📅 2026-09-13
📍 三軒茶屋
        ↓
🎧 DJ event
        ↓
☕ cafe / 🍴 lunch
        ↓
🌳 park / 🚶 walk
        ↓
🍸 bar
        ↓
💑 GO OUT PLAN
```

イベントを起点に、前後の場所を近づける。

### Distance policy

1. まず会場から徒歩10分圏
2. 候補が少なければ20分圏
3. 必要なら駅・公園などの中間地点を探索
4. 移動時間と営業時間を考慮して順序を決定

## Domain contract

FE はドメインごとの内部DB構造を知らない。

```json
{
  "id": "event-001",
  "kind": "event",
  "domain": "dj",
  "title": "DJ Event",
  "start_at": "2026-09-13T16:00:00+09:00",
  "end_at": "2026-09-13T22:00:00+09:00",
  "venue": {
    "name": "Example Venue",
    "area": "三軒茶屋",
    "lat": 35.64,
    "lng": 139.67
  },
  "source": "domain-be"
}
```

場所も同じ考え方で共通化する。

```json
{
  "id": "place-001",
  "kind": "place",
  "domain": "cafe",
  "name": "Example Cafe",
  "area": "三軒茶屋",
  "lat": 35.64,
  "lng": 139.67,
  "opening_hours": "...",
  "source": "places-be"
}
```

## Researcher philosophy

Researcher は単なるスクレイパーではない。

```text
Seed
 ↓
Search
 ↓
Discover people / venues / events
 ↓
Expand graph
 ↓
Verify source
 ↓
Upsert DB
```

たとえば `owarai-live` なら、下北GRIP / DASH を Seed にして、出演芸人 → 別ライブ → 会場 → 新しい出演者、と探索を広げる。

この探索ロジックは date-mcp に持たせず、各ドメイン Researcher が所有する。

## FE Orchestrator API concept

```text
POST /plan
```

```json
{
  "date": "2026-09-13",
  "area": "三軒茶屋",
  "domains": ["dj", "park", "cafe", "bar"],
  "constraints": {
    "max_walk_minutes": 15,
    "include_food": true,
    "include_walk": true
  }
}
```

Response:

```json
{
  "plan": [
    {"type": "cafe", "time": "14:00"},
    {"type": "event", "time": "16:00"},
    {"type": "park", "time": "18:30"},
    {"type": "bar", "time": "20:00"}
  ],
  "score": 0.91
}
```

## Design principle

### 1. FE は組み合わせる

ユーザー体験、検索条件、ランキング、プランニングを担当。

### 2. BE はドメインを守る

各DBのAPI、正規化、ドメイン固有ルールを担当。

### 3. DB は事実を持つ

イベント、人物、会場、場所、出典を蓄積する。

### 4. Researcher は世界を探索する

Webや公式情報から新しいノードを発見し、検証してDBへ渡す。

## Monorepo にしない理由

ドメインごとのDBとResearcherを独立させることで、

- お笑いだけ先に強くできる
- アイドルの探索方法を変えても date-mcp は変えない
- 公園DBを後から追加できる
- 外部APIや別Researcherへ差し替えられる
- 各ドメインが自分のデータ品質に責任を持てる

という構造にする。

## Roadmap

### Phase 1 — Contract

- [x] FE / BE / DB / Researcher の責務を分離
- [ ] 共通 Event / Place schema
- [ ] domain adapter interface

### Phase 2 — Orchestrator

- [ ] `search_events`
- [ ] `search_places`
- [ ] `plan_goout`
- [ ] distance / time ranking

### Phase 3 — Domain connection

- [ ] `owarai-live`
- [ ] `idol-live`
- [ ] `dj-event`
- [ ] `art-event`
- [ ] `tokyo-parks`
- [ ] cafe / lunch / bar domain
- [ ] theatre domain

### Phase 4 — Research loop

```text
Researcher → Candidate → Verify → DB
                         ↑        ↓
                         └── feedback
```

最終的には、**各ドメインが世界を調べ、date-mcp がその世界を一日の体験として編集する。**

## Philosophy

> DBを一つにするのではなく、体験を一つにする。

`date-mcp` は「イベント検索サイト」ではない。

**お笑い、落語、アイドル、DJ、演劇、アート、公園、カフェ、ランチ、バーという別々の世界を、時間と距離でつないで「今日はこれ」と編集するFEオーケストレーターである。**
