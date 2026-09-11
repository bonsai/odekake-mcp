# date-mcp

おでかけを「イベント中心」ではなく、**カフェ・ランチ・バーとイベントを近づけて組む**ための軽量MCP。

## Concept

```text
📅 日付 + 📍 エリア
        ↓
🎭 イベントを探す
        ↓
☕ カフェ / 🍴 ランチ / 🍸 バーを近くで探す
        ↓
🗺 距離・移動時間で並べる
        ↓
💑 GO OUT PLAN
```

## MVP

- `search_events` — イベント候補
- `search_places` — カフェ / ランチ / バー候補
- `plan_goout` — イベントを軸に前後の店を近距離で組み合わせる

### 距離ルール

イベント会場を中心に、まず徒歩10分圏を優先。候補が少なければ20分圏へ拡張する。

```json
{
  "event": "ライブ",
  "before": {"category": "lunch", "max_walk_minutes": 10},
  "after": {"category": "bar", "max_walk_minutes": 10},
  "optional": {"category": "cafe", "max_walk_minutes": 15}
}
```

まずは「全部入りの旅行プラン」ではなく、**イベントの近くに食事と飲みを寄せる**ところから始める。
