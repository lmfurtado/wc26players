# ⚽ World Cup 2026 — Data Pipeline

Data pipeline for the FIFA World Cup 2026 built on **Microsoft Fabric**, following a **Medallion architecture** (Bronze → Silver → Gold). Data is collected from a public football API, progressively transformed, and surfaced as interactive HTML visualizations.

---

## Architecture

```
football-data.org API
        │
        ▼
  Bronze  →  Silver  →  Gold  →  DataViz
```

The **masterPipeline** (Fabric Data Factory) orchestrates the three notebooks in sequence, sending Outlook email alerts on failure or success.

---

## Notebooks

| Notebook | Layer | What it does |
|---|---|---|
| `nBronze` | Raw | Fetches teams, squads and matches from the API; saves as Delta tables `bronze_Teams` and `bronze_Matches` |
| `nSilver` | Cleaned | Explodes the squad array into individual player rows; filters matches to `GROUP_STAGE`; saves `silver_Teams`, `silver_Players`, `silver_Matches` |
| `nGold` | Analytical | Enriches players with calculated age; joins team names/crests onto matches; aggregates home/away match counts and squad sizes per team; saves `gold_Teams`, `gold_Players`, `gold_Matches` |
| `nDataViz` | Visualization | Interactive HTML widgets (team grid, match cards, player roster, average age by team) rendered via `displayHTML` |

---

## Pipeline (`masterPipeline`)

```
Bronze ──► Silver ──► Gold ──► ✅ Notify Success
  │            │          │
  └──►         └──►       └──► ❌ Notify Error
```

- 12-hour timeout per activity
- Email notifications (Office365) on each failure and on final success

---

## Data Source

- **API:** [football-data.org](https://www.football-data.org/)
- **Competition:** FIFA World Cup 2026 (`id: 2000`)
- **Endpoints:** `/v4/competitions/2000/teams` and `/v4/competitions/2000/matches?season=2026`

---

## Requirements

- Microsoft Fabric workspace with a Lakehouse (`WCLakehouse`)
- `football-data.org` API token
- Outlook connection in Fabric (optional, for email alerts)
