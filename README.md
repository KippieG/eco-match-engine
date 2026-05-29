# 🚛 Eco-Match Engine

**An empty mileage optimizer built for intermodal logistics — powered by Python, FastAPI, and a business-first matching algorithm.**

> Built as a proof-of-concept targeting the ECS European Containers tech stack (TAS · Business Central · T-SQL · Power Platform). Demonstrates how a Digital Solutions Expert would bridge the gap between business need and digital solution.

---

## The problem

In container logistics, trailers frequently return **empty** (deadhead) after dropping a load — pure cost with zero revenue. For a fleet operating across the Zeebrugge hinterland, even a 15% reduction in empty mileage translates to tens of thousands of euros per year and measurable CO₂ savings.

Planners currently do this matching manually: scrolling through TAS, cross-referencing open export orders in Business Central, calling drivers. It takes time they don't have.

---

## The solution

Eco-Match Engine scans **open export orders** and **outbound trips** simultaneously, then surfaces matches where:

- The trip's **drop-off location** is within X km of the order's **pickup**
- The order is **ready** within a configurable window after the trailer arrives
- A **confidence score** ranks matches by proximity + timing tightness

The planner confirms with one click. The system then updates TAS, Business Central, and notifies the driver — no emails, no spreadsheets.

---

## Architecture

```
eco-match-engine/
├── api/
│   ├── main.py          # FastAPI app (REST endpoints + Swagger UI)
│   ├── matcher.py       # Core matching algorithm
│   └── geo_utils.py     # Haversine distance, CO₂ & cost calculations
├── demo/
│   └── index.html       # ECS-branded planner dashboard (zero dependencies)
├── data/
│   └── mock_orders.json # Realistic mock TAS + Business Central data
├── tests/
│   └── test_matcher.py  # 15 unit tests covering geo, matching, scoring
├── requirements.txt
└── README.md
```

---

## Tech stack

| Layer | Technology |
|---|---|
| API | Python · FastAPI · Pydantic |
| Matching | Custom haversine geo-match · confidence scoring |
| Frontend | Vanilla HTML/CSS/JS (zero dependencies, no build step) |
| Data | JSON (mock TAS + Business Central export) |
| Tests | pytest (15 tests, 100% pass rate) |
| Docs | Auto-generated OpenAPI / Swagger at `/docs` |

> **Integration-ready**: The `/confirm/{match_id}` endpoint is designed as a drop-in for a real Power Automate flow. Connect it to your TAS and Business Central APIs and the UI works unchanged.

---

## Quick start

```bash
git clone https://github.com/your-username/eco-match-engine
cd eco-match-engine

pip install -r requirements.txt
uvicorn api.main:app --reload --port 8000
```

Then open:

- **Dashboard UI** → http://localhost:8000
- **Swagger docs** → http://localhost:8000/docs

---

## API reference

### `GET /matches`

Find all viable trip-order pairs.

| Parameter | Default | Description |
|---|---|---|
| `radius_km` | 50 | Max pickup distance from trip drop-off (km) |
| `time_window_h` | 6 | Max hours between arrival and order ready-time |
| `min_confidence` | 0 | Minimum confidence score (0–100) |

**Example:**
```bash
curl "http://localhost:8000/matches?radius_km=80&time_window_h=8"
```

### `GET /summary`

Aggregate KPIs across all matches (total revenue, CO₂ saved, cost avoided).

### `POST /confirm/{match_id}`

Simulate confirming a match — triggers mock updates to TAS, Business Central, and driver notification. In production, replace the response body with real API calls.

---

## Running tests

```bash
pip install pytest
pytest tests/ -v
```

```
tests/test_matcher.py::TestHaversine::test_zeebrugge_to_duisburg PASSED
tests/test_matcher.py::TestHaversine::test_symmetry PASSED
tests/test_matcher.py::TestFindMatches::test_exact_match PASSED
tests/test_matcher.py::TestFindMatches::test_no_match_when_too_far PASSED
tests/test_matcher.py::TestFindMatches::test_sorted_by_confidence PASSED
... 15 passed in 0.42s
```

---

## Business impact (mock data baseline)

Running the engine against the included dataset (4 outbound trips, 5 open orders, 50 km radius, 6h window):

| KPI | Value |
|---|---|
| Matches found | 4 |
| Revenue unlocked | €7,170 |
| CO₂ saved | ~689 kg |
| Diesel cost avoided | ~€521 |

Scale this to a real fleet of 200+ trailers and the numbers become significant — especially for CSR reporting.

---

## Roadmap (production considerations)

- [ ] **TAS connector** — replace mock JSON with live T-SQL queries via `pyodbc`
- [ ] **Business Central adapter** — read/write orders via BC REST API
- [ ] **Power Automate trigger** — call `/confirm` from a PA flow on planner approval
- [ ] **Power BI integration** — export match history to a PBIX dashboard
- [ ] **Multi-day planning** — extend time window beyond same-day matching

---

## About ECS

[ECS European Containers](https://www.ecs.be) is the operational heart of the port of Zeebrugge, offering full-load transport, supply chain logistics, conditioned transport, and Brexit & customs services across the UK, Ireland, and European mainland.

---

*Built with ❤ for the Digital Solutions Expert role · Zeebrugge, Belgium*
