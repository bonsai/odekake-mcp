# odekake-mcp stack

> ひとりでも、デートでも。おでかけを組み立てて Google Calendar へ。

## Core idea

各ドメインのResearcherが、それぞれの世界を継続的に調査して知識を蓄積する。

`odekake-mcp` はそのデータを横断して、時間・距離・料金・営業時間・天候・雰囲気・同行者・過去の選択などを組み合わせ、**「なぜこのプランを選ぶのか」まで説明できる最適なおでかけプラン**を提案する。

```text
Domain Researchers
  ├─ owarai-live
  ├─ idol-live
  ├─ dj-event
  ├─ art-event
  ├─ tokyo-parks
  └─ cafe / shop / theatre ...
          │
          ▼
      DATA POOL
          │
          ├─ facts
          ├─ sources
          ├─ relationships
          └─ research history
          │
          ▼
   ODEKAKE PLANNER
          │
          ├─ time fit
          ├─ distance fit
          ├─ budget fit
          ├─ opening-hours fit
          ├─ weather fit
          ├─ atmosphere fit
          ├─ solo / date fit
          └─ preference fit
          │
          ▼
     Recommendation
          │
     ┌────┴────┐
     │  WHY?   │
     └────┬────┘
          ↓
   evidence + reasons
          ↓
    User confirmation
          ↓
   Google Calendar
```

## Architecture

```text
GitHub
  ├─ code / schema / seeds
  ├─ Issues = research instructions
  ├─ PRs = research results
  └─ Actions = scheduled researchers
          │
          ▼
      GCP / Agent
          │
    ┌─────┼──────────────┐
    ▼     ▼              ▼
 Drive  BigQuery       Cloud Run
    │     │              │
    │    BQML             │ API / MCP
    │     │              │
    └─────┼──────────────┘
          ▼
     Planner Agent
          │
          ▼
   Google Workspace
     ├─ gws
     ├─ GAS
     ├─ Drive
     ├─ Sheets
     └─ Calendar
          │
          ▼
   Google Calendar
      = GOAL
```

## Stack

| Layer | Technology | Role |
|---|---|---|
| Source | GitHub | code, schema, seeds, research history |
| Research workflow | GitHub Actions | scheduled discovery / validation |
| Agent | Gemini / MCP | research, planning, orchestration |
| Runtime | Cloud Run | API / agent execution |
| Data lake | Google Drive / Cloud Storage | source data / intermediate artifacts |
| Analytics DB | BigQuery | events, venues, people, spots |
| ML | BigQuery ML | preference / recommendation learning |
| Workspace bridge | gws / GAS | Google Workspace operations |
| Calendar | Google Calendar API | final itinerary registration |

## Domain data

- `owarai-live` — comedy / rakugo
- `idol-live` — idols
- `dj-event` — DJs / music
- `art-event` — art
- `tokyo-parks` — parks
- future: theatre / cafe / lunch / shops / spots

## Researcher → Planner

Researcherは「おすすめ」を決めない。まず事実と根拠を蓄積する。

```text
Researcher
   ↓
Candidate
   ↓
Evidence
   ├─ source
   ├─ time
   ├─ price
   ├─ location
   ├─ accessibility
   └─ context
   ↓
Verified data
   ↓
Planner
```

Plannerは候補を比較し、**選択理由を構造化して保持する**。

```json
{
  "candidate": "event-123",
  "score": 0.91,
  "reasons": [
    {
      "factor": "solo_friendliness",
      "evidence": "一人でも参加しやすい"
    },
    {
      "factor": "time_fit",
      "evidence": "前の予定から45分で移動可能"
    },
    {
      "factor": "preference_match",
      "evidence": "過去に音楽＋散歩を選択"
    }
  ]
}
```

## Data flow

```text
Seed
 ↓
Research Agent
 ↓
Candidate
 ↓
Verify
 ↓
GitHub PR
 ↓
Merge
 ↓
BigQuery / Drive
 ↓
Planner Agent
 ↓
Compare candidates
 ↓
Optimize itinerary
 ↓
Explain WHY
 ↓
User confirmation
 ↓
Google Calendar
```

## API direction

The API should eventually expose the planning operation rather than only search endpoints.

```text
GET  /events
GET  /venues
GET  /spots
GET  /people

POST /plan
```

Example:

```json
{
  "date": "2026-09-13",
  "mode": "solo",
  "area": "東京",
  "theme": ["art", "music"],
  "budget": 3000
}
```

Output:

```text
best plan
  + score
  + evidence
  + reasons
  + alternatives
  + calendar event data
```

## Design principle

> DBを一つにするのではなく、体験を一つにする。
>
> **最適なプランを出す。そして、その理由を説明できる。**

GitHub is the source of truth for research and code. Google services are the operational layer. BigQuery/BQML becomes useful after enough outing history accumulates.
