# odekake-mcp stack

> ひとりでも、デートでも。おでかけを組み立てて Google Calendar へ。

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
Odekake Plan
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

Output is an itinerary that can be registered in Google Calendar.

## Design principle

> DBを一つにするのではなく、体験を一つにする。

GitHub is the source of truth for research and code. Google services are the operational layer. BigQuery/BQML becomes useful after enough outing history accumulates.
